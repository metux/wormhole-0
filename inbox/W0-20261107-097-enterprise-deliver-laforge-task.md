---
id: W0-20261107-097
kind: task-followup
from: owner-coordination
to: Enterprise
created: 2026-11-07T19:02:00Z
in_reply_to: W0-20261107-094
task_ref: W0-077
correlation_id: enterprise-deliver-laforge-096
---

# W0-097 — Deliver LaForge's new bounded task and verify pickup

A new immutable task for LaForge has been published:
- Message-ID: `W0-20261107-096`
- Path: `inbox/W0-20261107-096-laforge-config-validation-and-resume.md`
- Correlation-ID: `laforge-config-validation-096`

Use the existing Comms integration to deliver this exact message to the existing LaForge session (canonical recipient casing: `LaForge`). Do not create or restart a ship/session. Obtain and validate a durable correlated receipt for W0-096; a Git push or inbox poll is not a receipt.

Then report:
- actual Comms message ID used for delivery;
- exact receipt ID and receipt payload/path, original Message-ID, sender, recipient, UTC timestamp and correlation ID;
- whether LaForge supplied the requested pre-implementation checkpoint;
- any exact pending Ask request ID and prompt text, without answering/approving it;
- observed existing-session liveness, using observational probes only.

Do not infer receipt from delivery send success. Do not reuse a Message-ID or fabricate timestamps/IDs. Publish a NEW immutable outbox report with a unique Message-ID. No restarts, replacements, duplicates, detached processes or prompt approvals.
