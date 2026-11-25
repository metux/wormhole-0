# OpenCode v1 user-message event ordering for current-turn correlation

Date: 2026-11-25
Status: source-verified publication order; plugin callback completion ordering remains unverified.

## Verified against OpenCode tag v1.19.3

- `packages/opencode/src/session/prompt.ts`: `createUserMessage()` constructs the user message and awaits `sessions.updateMessage(info)`; the normal `prompt()` flow awaits `createUserMessage(input)` before entering `loop({ sessionID })`.
- `packages/opencode/src/session/session.ts`: `updateMessage()` awaits `events.publish(SessionV1.Event.MessageUpdated, { sessionID, info: msg })`.
- `packages/opencode/src/session/llm/request.ts`: the model request preparation inside the loop invokes `experimental.chat.system.transform` before `chat.params`.

Therefore the user-message `message.updated` event is published before the request-preparation system-transform hook is reached. This supports the original `message.updated` correlation candidate and disproves using `chat.params` as the source for the same transform invocation.

## Remaining uncertainty

The source ordering proves publication order, not that the plugin's asynchronous `event` hook has completed and updated local correlation state before `experimental.chat.system.transform` starts. Do not assume callback delivery ordering without verifying the V1 plugin event bridge/runtime behavior.

A robust implementation should either:
- verify and test that the plugin event callback is observed before the transform hook for the same session/message; or
- query `session.messages` from the transform hook (the V1 SDK exposes this API) and establish the current user message from session history, with strict session matching and no guessing.

If the current message cannot be identified unambiguously, leave events retryable and do not call `seen_mark`.

## Scope

This refines issue #24 current-turn correlation only. It does not implement lifecycle persistence or completion tracking.
