# Comms event lifecycle correlation design

Date: 2026-11-25
Status: implementation design for issue #24; not yet implemented

## Verified OpenCode 1.19.3 facts

The exact upstream tag `v1.19.3` was inspected:

- The V1 `session.promptAsync` request body accepts optional `messageID`.
- `promptAsync` responds with HTTP 204 and no message ID, so the plugin must supply a correlation ID if it needs to track the created user message.
- `message.updated` carries the message info. User messages expose their ID; assistant messages expose `parentID`, `time.completed`, and optional error information.
- `session.status` is session-scoped and has busy/idle/retry states. Deprecated `session.idle` is not event-correlated and cannot prove completion.
- The V1 SDK exposes `session.messages` for querying persisted message history during restart recovery.

## Correlation design

- New-turn adapter: derive a deterministic OpenCode message ID from the Comms event batch and persist the mapping before/at submission. Supply it as `body.messageID` to `promptAsync`. Retries of the same batch in the same session reuse the same ID.
- Current-turn adapter: associate injected Comms event IDs with the active user message ID observed through `message.updated` before the system-transform hook runs.
- One OpenCode user message may represent multiple Comms event IDs; persist the full mapping.
- `session.status: busy` can establish `turn_started` for the correlated active turn.
- Completion requires an assistant `message.updated` whose `parentID` matches the correlated user message and whose `time.completed` is present. A correlated assistant/session error is failure.
- Session idle alone never marks a Comms event complete.

## Durable state design

Go stores generic opaque correlation references, not OpenCode-specific prompt structures. Lifecycle records should include event ID, state, lease owner/expiry, attempt/backoff, opaque session/message correlation IDs, timestamp, and bounded error details.

Proposed generic transitions: offered (poll-derived), received/prepared (optionally plugin-local), submitted, turn_started, completed, failed. Repeated identical transitions must be idempotent. Leases must prevent concurrent consumers and expire/recover after restart. Only correlated completion moves the message to seen; legacy `seen_mark` remains for older plugins.

The new-turn and current-turn adapters feed the same lifecycle; typed actions (model switch, quit, reset) remain a later extension.

## Tracking

Implementation issue: https://github.com/mpbt-hq/starfleetctl/issues/24
Depends on event protocol v1 in PR #23: https://github.com/mpbt-hq/starfleetctl/pull/23

This is a proposed implementation design, not a claim that lifecycle tracking or end-to-end completion proof already exists.
