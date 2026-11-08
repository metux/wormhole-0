# Architecture decision: Wormhole is a Comms transport

Status: **OWNER-APPROVED — 2026-11-08**

## Decision

Comms is the only messaging system exposed to agents. Wormhole is a transport backend for Comms, not a parallel agent messaging system.

A message received over Wormhole must enter the local Comms bus through the existing external-ingestion API. The agent/session plugin consumes only the local Comms inbox. It must not know whether a message originated locally or arrived over Wormhole.

The data path is:

~~~text
remote Comms
    -> Wormhole Git spool
    -> local Wormhole transport worker
    -> Comms Core external-ingestion API
    -> local per-ship Comms inbox
    -> agent/session plugin
    -> session input queue
~~~

The reverse direction uses the same Comms routing and Wormhole transport selection. No Wormhole-specific prompt format or agent-side Wormhole poller is introduced.

## Component ownership

### Comms Core

- Owns logical addressing, recipient validation, local inbox publication, deduplication, and message lifecycle.
- Exposes the stable external-ingestion API to transports.
- Remains the only writer of local Comms message records.
- Preserves external source identity and external message ID for durable idempotency.

### Wormhole transport

- Owns Git repository synchronization and the remote durable spool lifecycle (new/, cur/, processed/, failed/).
- Imports a received immutable message through Comms Core's external-ingestion API (currently Bus.PostExternal(source, externalID, from, target, text)).
- Must not write directly into .starfleet-ai/var/comms/msgs/, bypass Comms routing, or invent a second local inbox.
- A successful PostExternal import means the local Comms message was durably published; it does not mean an agent session received or processed it.
- Retains/retries the Wormhole source message until the Comms import is confirmed. Repeating the import must reuse the same external identity and exact payload bytes.

### Agent/session plugin

- Polls only the local Comms inbox through the existing Comms dispatch API.
- Resolves the target ship to its intended live session.
- Delivers each directive as a normal queued session input (e.g. the existing OpenCode session.promptAsync() path), not by appending arbitrary message bodies to the system prompt.
- Includes stable Comms message ID, sender, type, and correlation/task metadata in the delivered envelope.
- Leaves a message pending if no valid session is available or session submission fails.
- Marks the message as handed off only after the session API confirms successful acceptance. That state is not task completion.
- Does not perform Git operations or read Wormhole repositories.

## Delivery vocabulary

These states must not be conflated:

1. **Published** — sender's Wormhole message commit was pushed.
2. **Imported** — the receiving transport successfully published it into local Comms using PostExternal.
3. **Queued to session** — the session API accepted the message as an input event.
4. **Processed** — the agent actually handled the message's requested work.
5. **Completed / failed** — the requested work reached an explicit outcome, reported through Comms with correlation to the original message.

A directory listing, inbox poll, plugin startup, or Comms acknowledgement is not evidence of processing or completion. A successful Git push is not evidence of import, session delivery, or work completion.

## Existing implementation findings (verified against current sources)

- Comms already has a durable, idempotent external-ingestion API in internal/comms/comms_external.go: Bus.PostExternal(source, externalID, from, target, text). It records source/external identity and a payload hash, and rejects a changed payload for an existing key.
- The Wormhole importer should use that API rather than writing Comms message files itself.
- The OpenCode plugin already polls comms inbox and has a session.promptAsync() path for mid-turn directives.
- The current plugin also has a second delivery path in experimental.chat.system.transform which calls seen_mark before adding directives to the system prompt. That can acknowledge a message before a session input has actually been accepted, and it duplicates the polling path.
- Startup currently calls init, whose contract acknowledges unread messages and returns them as a process-local backlog. A crash after that acknowledgement but before session injection can lose the delivery opportunity. Startup must not consume messages before durable handoff.
- The plugin's system-transform hook should be limited to stable session identity/context. Comms directives must use one session-input delivery path.

## Required delivery/recovery semantics

- Do not claim exactly-once delivery across the filesystem Comms bus and the independent session database unless the session API provides a durable idempotency primitive.
- Prefer recoverable at-least-once handoff with stable message IDs over at-most-once behavior that can lose work.
- Persist enough handoff state to recover after plugin restart. Reconcile the narrow crash window between session acceptance and Comms handoff recording, using session history/message metadata if available; otherwise document and test the at-least-once duplicate window.
- The agent-facing envelope must explicitly say to process the request, not merely acknowledge that it exists. It must include the complete message content and stable ID.
- Treat message bodies as untrusted data. They are ordinary user/session input, not privileged system instructions. Authorization comes from Comms routing and task policy.
- If the target session is unavailable, retain the message in Comms as pending and retry/wake the correct session when it becomes available.
- Multiple sessions sharing one ship identity require an explicit single-consumer/lease policy; do not inject the same inbox item concurrently into multiple sessions.
- Existing controlled Comms command handling remains separate. Wormhole-originated message text must never be promoted into an executable command type.

## Scope boundary

W0-129 remains the current Wormhole spool implementation task. It must use the Comms external-ingestion API at the import boundary and must not add a parallel Comms file writer.

The OpenCode plugin delivery correction is a separate, focused follow-up in starfleetctl. Do not mix broad plugin changes into W0-129's Wormhole spool commits. Before implementing that follow-up, produce a small commit plan and tests against the current plugin behavior.

## Acceptance tests for the follow-up

- A local Comms message and a Wormhole-imported message follow the same session-delivery path.
- Wormhole import uses PostExternal; retry with the same source/ID/payload creates no duplicate, and a changed payload fails closed.
- Plugin startup does not acknowledge/consume messages before session handoff.
- No system-transform code marks a directive seen before session input acceptance.
- Successful session submission records handoff; failed submission leaves the message pending.
- Plugin restart recovers pending messages and handles the post-acceptance/pre-recording crash window according to the documented at-least-once/idempotency policy.
- The delivered prompt contains the full message, sender, stable message ID, and explicit instruction to perform the requested work.
- No available session means no acknowledgement; session wake-up/reattachment eventually delivers the pending message.
- Two sessions with the same ship identity cannot process the same message concurrently.
- Handoff acknowledgement is never reported as task completion; completion/failure requires a separate correlated outcome.
- Existing local Comms messages, broadcasts, commands, and ask/reply behavior remain compatible.
