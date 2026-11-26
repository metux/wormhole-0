# OpenCode current-turn correlation via chat.message hook

Date: 2026-11-26
Status: source-verified candidate for Starfleetctl issue #24; runtime integration tests still required.

## Verified against OpenCode tag v1.19.3

In `anomalyco/opencode` tag `v1.19.3`:

- `packages/plugin/src/index.ts` declares the `chat.message` hook with `output.message: UserMessage`; the message includes its `id` and `sessionID`.
- `packages/opencode/src/session/prompt.ts`, in `SessionPrompt.createUserMessage()`, awaits `plugin.trigger("chat.message", ...)` with the freshly constructed `info` message before calling `sessions.updateMessage(info)`.
- `SessionPrompt.prompt()` awaits `createUserMessage(input)` before entering `loop({ sessionID })`.
- `packages/opencode/src/session/llm/request.ts` invokes `experimental.chat.system.transform` inside request preparation in that later loop, before `chat.params`.

Therefore a plugin's awaited `chat.message` hook is a stronger current-turn correlation candidate than an asynchronous `message.updated` event callback: it receives the concrete user message ID synchronously in the hook sequence before the subsequent request loop can reach system transformation. This also avoids the known incorrect `chat.params`-before-transform assumption.

## Proposed use for issue #24

- Capture `output.message.id` and `output.message.sessionID` in `chat.message` into a session-keyed pending-current-turn slot.
- In `experimental.chat.system.transform`, consume only a pending correlation whose session ID exactly matches `input.sessionID`; never use a stale or cross-session value.
- Associate the event IDs injected by that transform with the captured user-message ID in the durable generic lifecycle API.
- Define and test behavior for multiple queued user messages, retries, regenerations, compaction/autocontinue, concurrent sessions, transform failures and plugin restart. Clear or expire stale pending correlations conservatively.
- Keep assistant completion based on a matching assistant `message.updated` with `parentID` equal to the correlated user message and `time.completed` present. This candidate hook only solves the pre-transform user-message identity; it does not prove turn start/completion.

## Limitations

This is source-order evidence from upstream v1.19.3, not a runtime test of the installed plugin/event bridge. Before implementation is accepted, add focused tests demonstrating that `chat.message` is awaited before the same request reaches system transformation and that session correlation remains correct under retries/concurrency. Do not mark events complete from hook invocation, message persistence, `seen_mark`, or session idle.

## Scope

Starfleetctl issue #24 only. This does not modify the generic Go event protocol or claim that lifecycle code is implemented.
