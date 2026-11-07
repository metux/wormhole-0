---
id: W0-20261107-074
kind: task
from: owner-coordination
to: enterprise
created: 2026-11-07T14:55Z
priority: urgent
workspace: mpbt-hq/starfleetctl
correlation_id: W0-20261107-074
---

# Immediate coordination escalation — deliver tasks and prove liveness

The owner reports that Defiant and Laforge have again stopped progressing. The two new urgent tasks are now committed to Wormhole inbox:

- Defiant: W0-20261107-072
- Laforge: W0-20261107-073

Your role remains coordination only. Do not perform their technical audits or edit their source branches. Do not restart, relaunch, detach, duplicate, or replace any console-attached ship.

## Required actions in the existing Enterprise session

1. Read both task files from the shared Wormhole inbox.
2. Deliver each task to its named ship through the real Comms transport, using unique correlation IDs W0-20261107-072 and W0-20261107-073.
3. Obtain and preserve durable delivery receipts. A GitHub inbox commit alone is not delivery.
4. Send a unique nonce liveness probe to Defiant and Laforge through Comms. Record actual Comms message IDs, timestamps, latency, and correlated replies. Use a 60-second timeout and one retry; do not infer liveness from GitHub activity.
5. For each ship, report its current task, branch/worktree/HEAD if observable, whether it is actively working, idle, blocked, or waiting on a prompt, and the exact pending prompt/request ID and text if blocked. Do not answer or approve prompts.
6. If a ship replies with a concrete next action and is not blocked, explicitly direct it to continue the existing assigned work immediately and keep producing durable checkpoints after each meaningful step.
7. Commit a concise factual report in Wormhole outbox, including delivery/probe receipts and any failures. If the Comms transport cannot be used, report the exact technical blocker; do not fabricate IDs or claim delivery.

## Anti-idling

Do not answer with only a board snapshot or “working”. Perform delivery and probes first. Continue through all steps without waiting for another owner nudge. Do not spawn replacement sessions.