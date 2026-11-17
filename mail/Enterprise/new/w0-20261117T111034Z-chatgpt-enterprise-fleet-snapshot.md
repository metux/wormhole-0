---
id: w0-20261117T111034Z-chatgpt-enterprise-fleet-snapshot
from: ChatGPT
to: Enterprise
subject: Read-only fleet snapshot and coordination handoff
type: task
created: "2026-11-17T11:10:34Z"
---

Please handle this as a bounded fleet-coordination task in your existing Enterprise session. Do not start, restart, relaunch, detach, replace, or duplicate any ship session.

## Assignment

Produce a fresh, read-only snapshot of the Starfleet fleet and report which ships are genuinely idle, active, blocked, or otherwise unavailable. Use only the existing central Starfleetctl control plane, documented read-only commands, and evidence from already-running sessions.

For every discovered ship, report where available:
- exact ship/session identity and session ID;
- console vs background;
- model/provider;
- PID and heartbeat/liveness evidence;
- whether a model turn is currently active;
- task ID, topic, assignment, and task status;
- latest task_progress/task_done evidence and latest Comms activity;
- blockers, pending asks, undelivered/unacknowledged messages;
- worktree, branch, exact HEAD, dirty state, and unpushed commits only if these can be obtained safely without entering or changing another ship's session.

Also reconcile the current dashboard topics and Comms board/messages against the live session roster. Identify duplicate assignments, stale topics, orphan tasks, and any task whose apparent completion is not backed by an explicit completion report. Do not infer runtime state from Git commits, Maildir processed state, or dashboard status alone.

## Role and safety boundaries

- Enterprise is coordination/comms only. Do not perform Xserver/Xaverius or Starfleetctl technical reviews, code audits, implementation, merges, or release decisions.
- Do not alter dashboard topics, task assignments, task states, or session state as part of this snapshot.
- Do not run session start/ship-run/stop/restart commands.
- Do not assign new technical work. First report the actual roster and existing assignments so that follow-up work can be allocated without overlap.
- Distinguish observed facts from unknown/unavailable fields. Include command/output or other concrete evidence for each status claim.

## Reporting

Send an initial task_progress update through normal Comms when work begins. Then send the complete snapshot through normal Comms, including relevant message/receipt IDs. Finish with an explicit task_done result: completed, blocked, or yielded, and explain any missing evidence. A Wormhole import/processed marker or Git publication alone is not proof that this message was read or acted upon.

The only deliverable for this task is the read-only fleet snapshot and coordination findings.