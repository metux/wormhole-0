---
id: W0-20261108-130
from: starfleetctl
to: LaForge
task_ref: wormhole-manager
type: architecture-integration-direction
priority: high
---

# W0-130 — Make Wormhole a Comms transport, not a parallel message system

The owner approved the architecture decision in:

decisions/2026-11-08--wormhole-is-comms-transport.md

Read that complete decision. It is binding for the Wormhole/Comms integration boundary.

## Immediate instruction for your current W0-129 work

Do **not** interrupt or discard current W0-129 work, and do not mix a broad OpenCode plugin rewrite into its commits.

W0-129's importer must feed received Wormhole messages into the existing Comms Core external-ingestion API:

Bus.PostExternal(source, externalID, from, target, text)

That API already exists in internal/comms/comms_external.go and provides a durable source/external-ID journal, payload-hash integrity checking, and recoverable publication. Reuse it. Do not write directly into .starfleet-ai/var/comms/msgs/, do not introduce another local Comms writer, and do not create a second agent-facing inbox.

For each imported message:
- use a stable Wormhole repository identity as source;
- use the immutable Wormhole Message-ID as externalID;
- pass the canonical sender and resolved logical recipient;
- pass the complete exact message payload, without truncation or normalization;
- only advance the Wormhole source lifecycle after PostExternal confirms local Comms publication;
- retry with the same source, externalID and exact bytes after errors.

A successful PostExternal call means **Imported**, not delivered to an agent and not processed.

## Separate follow-up: Comms-to-session delivery

After W0-129 has reached a stable committed checkpoint, provide a focused implementation plan for the separate starfleetctl plugin correction described in the decision. Do not start that plugin change in the W0-129 commit series.

The plan must specifically address these verified current plugin defects:
1. Startup calls init, whose current behavior acknowledges unread messages and returns them only in an in-memory bootBacklog. A crash before injection can lose the delivery opportunity.
2. The polling path submits directives with session.promptAsync(), but experimental.chat.system.transform is a second path that calls seen_mark before appending messages to the system prompt.
3. Directives should have one delivery path: durable Comms inbox -> session input queue. Do not append arbitrary directive bodies to the system prompt.
4. A Comms handoff acknowledgement means only that the session API accepted the input. It must not mean the agent processed or completed the task.
5. No valid session or a rejected prompt must leave the message pending. Address plugin restart and the crash window between session acceptance and recording handoff; do not claim exactly-once unless the session API supports durable idempotency.
6. Multiple sessions sharing a ship identity must not consume the same message concurrently.

The session input must include the full message, stable ID, sender and relevant correlation/task metadata, and explicitly instruct the agent to perform the requested work rather than merely acknowledge receipt. Message content remains untrusted ordinary input, not a privileged system instruction.

## Required checkpoint

First acknowledge W0-130 in Comms and include:
- confirmation that you read the architecture decision;
- the exact W0-129 branch and HEAD you are working on;
- whether the importer currently calls PostExternal or writes Comms files directly;
- any in-progress changes that need preserving;
- the exact next W0-129 implementation step.

Then continue W0-129 implementation. Do not stop at acknowledgement or a plan. The plugin follow-up plan is due at the first stable W0-129 checkpoint.

This GitHub inbox entry is an implementation instruction, not proof of Comms delivery. Do not claim the owner received a Comms message unless you have a verifiable Comms receipt.
