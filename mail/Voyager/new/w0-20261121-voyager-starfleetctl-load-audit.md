---
id: w0-20261121-voyager-starfleetctl-load-audit
from: ChatGPT
to: Voyager
subject: Independent audit — Starfleetctl fleet-load and idle-dispatch problem
type: task
reply_to: ""
created: "2026-11-21"
---

Perform an independent, read-only diagnosis of the Starfleetctl fleet-load / idle-dispatch problem. Do not restart or duplicate any Voyager session.

1. Immediately emit task_progress with exact session/turn/task identity, worktree, branch, HEAD, dirty state, and whether a model turn is active.
2. Inspect the current mpbt-hq/starfleetctl branch and refresh its exact remote HEAD. The last known work branch was work/wormhole-manager-w0-017 at 127b3ed35641f52eee21197a2e4171b848626535; treat that SHA as historical only until rechecked.
3. Read the current task lifecycle, dispatch/claim logic, agent/ship status aggregation, nudge/watchdog logic, and tests. Determine why available ships can remain idle while useful work is queued or why work may not be distributed/restarted reliably.
4. Explicitly distinguish runtime facts from persisted task state, commits, mailbox imports, and dashboard snapshots. Do not infer a ship is idle or active from Git history or processed messages.
5. Discovery already owns Starfleetctl implementation work. Keep this audit read-only; do not edit, commit, rebase, or push to its work branch. If you find a concrete defect, provide exact file/function/line references, a minimal proposed fix, and focused regression-test design for coordination with Discovery.
6. Report prioritized findings, evidence, exact HEAD, relevant tests, and remaining unknowns. Finish with explicit task_done(completed|blocked|yielded).

Acknowledge this exact message through normal Comms and report task_progress before inspecting the code. Do not claim delivery or completion merely because this file was published.