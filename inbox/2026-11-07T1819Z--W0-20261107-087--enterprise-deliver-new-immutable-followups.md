---
id: W0-20261107-087
kind: task-followup
from: owner-coordination
to: enterprise
created: 2026-11-07T18:19:00Z
in_reply_to: W0-20261107-084
task_ref: W0-075,W0-076,W0-085,W0-086
workspace: metux/wormhole-0@master
---

# W0-087 — Deliver new immutable follow-ups and enforce receipt semantics

This is a NEW message supplementing W0-084 and W0-077. It does not edit or replace either prior task. Read it in full.

## Owner decision: immutable messages from now on

The owner has approved the following communication rule:

1. Never append new instructions, corrections, or status requests to an already-published inbox message.
2. Every new instruction or correction is a new uniquely identified immutable message file.
3. Each new message explicitly references the prior task/message it supplements or supersedes.
4. Existing messages remain unchanged as historical records.
5. Delivery requires a correlated durable receipt from the recipient. A Git commit/push, inbox poll, or transport send success is not by itself proof that the recipient received/read the message.
6. Acknowledgement of receipt is distinct from a processing/completion report.
7. On each normal coordination cycle, refresh the inbox and inspect newly published messages. Do not assume a running agent will notice a changed file without an explicit refresh/delivery.
8. If no delivery receipt or correlated acknowledgement is visible, state `unconfirmed`; never invent IDs or claim delivery.

## New messages requiring delivery now

- Defiant: `W0-20261107-085`, file `inbox/2026-11-07T1819Z--W0-20261107-085--defiant-followup-exact-tip-repair.md`
- LaForge: `W0-20261107-086`, file `inbox/2026-11-07T1819Z--W0-20261107-086--laforge-followup-immutable-message-policy.md`

Immediately:
1. Fetch and read both exact new messages from Wormhole master.
2. Deliver each through the actual fleet Comms transport to the existing ship instance, using the exact Message-ID as correlation ID and exact canonical recipient casing (`Defiant`, `LaForge`).
3. Obtain and record a durable correlated delivery receipt for each, or report the exact transport/tool error and mark delivery unconfirmed.
4. Obtain a separate correlated acknowledgement/reply from each ship. Preserve exact reply IDs and UTC timestamps.
5. Ask each ship for its current prompt/Ask state and current work checkpoint. If an Ask is pending, record exact prompt text and request ID; do not approve it or work around it.
6. Do not restart, relaunch, detach, duplicate, or replace existing console-attached ship processes/sessions.

## Required owner-facing report

Write a new immutable report in `outbox/` (do not edit an older report) with:
- UTC timestamp and Enterprise's own session/worktree/branch/HEAD/timer status, if observable;
- each target's exact message ID, recipient, send timestamp, transport result, receipt ID/status, correlated reply ID/timestamp and liveness probe evidence;
- separate status for message delivery, receipt, ship acknowledgement, liveness, and implementation progress;
- exact current task ID, branch/HEAD/worktree and last commit/test evidence where observable;
- exact blockers and next concrete actions;
- previous outbox report reference and next expected report time.

Continue W0-084's fresh-nonce liveness probes for Defiant and LaForge with the prescribed 60-second timeout plus one retry. Do not confuse slow response, timeout, unavailable Comms, pending Ask, or confirmed process hang. Do not spawn replacement ships.

## Continuous reporting

Publish a durable outbox report every normal ~5-minute coordination cycle, including blocked/no-change cycles. Silence is not a status. If the timer itself fails, record that fact in the next available report. Keep Enterprise in its coordination-only role; no code review or implementation work.

Acknowledge this exact message ID `W0-20261107-087` via the durable receipt path, then execute immediately. Do not treat the GitHub publication of this file as delivery to Enterprise.
