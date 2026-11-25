# OpenCode current-turn message correlation observation

Date: 2026-11-25
Status: verified SDK type observation; invocation ordering still requires runtime/source verification.

## Verified against OpenCode tag v1.19.3

In `anomalyco/opencode` tag `v1.19.3`:

- `packages/plugin/src/index.ts` declares `Hooks["chat.params"]` input with `sessionID` and `message: UserMessage`.
- `packages/sdk/js/src/gen/types.gen.ts` defines `UserMessage.id` and `UserMessage.sessionID`.
- The same plugin hook declaration gives `experimental.chat.system.transform` only `sessionID?` and `model`; it does not include the active message ID.

This means the current-turn adapter has a direct candidate correlation source: capture `input.message.id` and `input.message.sessionID` in `chat.params`, then associate the subsequent system-transform injection with that message. This is preferable to inferring the current user message from a later `message.updated` event if hook ordering and identity continuity are verified.

## Required before implementation

- Verify in the v1.19.3 source/runtime that `chat.params` is invoked for the same user message before `experimental.chat.system.transform`.
- Verify behavior on retries, regenerated requests, compaction/continuation and concurrent sessions.
- Keep a conservative fallback: if the captured message ID/session does not match the transform session, do not submit or acknowledge; leave the Comms events retryable.
- Persist the exact session/message correlation with the event IDs through the generic lifecycle API.

## Scope

This refines the current-turn correlation investigation for mpbt-hq/starfleetctl issue #24. It does not change the generic Go protocol and is not evidence that lifecycle persistence or completion tracking has been implemented.
