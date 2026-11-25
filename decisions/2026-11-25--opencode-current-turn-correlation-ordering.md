# OpenCode v1 current-turn hook ordering correction

Date: 2026-11-25
Status: source-verified correction to the candidate recorded in `decisions/2026-11-25--opencode-current-turn-message-correlation.md`.

## Verified against OpenCode tag v1.19.3

In `anomalyco/opencode` tag `v1.19.3`, `packages/opencode/src/session/llm/request.ts` invokes `experimental.chat.system.transform` first, then invokes `chat.params` later in the same request-preparation function. Therefore `chat.params.input.message.id` is too late to identify the message for the preceding `experimental.chat.system.transform` callback. Do not implement current-turn correlation by capturing the message ID in `chat.params` and expecting it to be available to that same transform call.

The SDK type facts remain true: `chat.params` receives a `UserMessage` with `id` and `sessionID`; `experimental.chat.system.transform` receives only `sessionID?` and `model`. The ordering makes the former unsuitable as a pre-transform source.

## Consequences for issue #24

- The current-turn path still needs a message ID that is available before system transformation.
- `message.updated` for the user message remains a candidate, but its event ordering relative to request preparation/system transformation must be verified from source/runtime before relying on it.
- If no reliable pre-transform correlation can be proven, do not guess or acknowledge. Consider a narrowly scoped adapter change only if the OpenCode V1 API provides a safe supported hook with the necessary message ID.
- Keep new-turn `promptAsync` deterministic `body.messageID` correlation separate; that API path has a caller-supplied ID.

This correction supersedes the proposed `chat.params` capture strategy in the earlier observation. No lifecycle code has been implemented on the issue #24 branch as a result of this investigation.
