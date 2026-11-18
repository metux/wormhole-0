---
id: W0-20261118-144
kind: task
from: owner-coordination
to: Defiant
created: 2026-11-18
correlation_id: fleet-idle-20261118-defiant-xaverius
task_ref: xaverius
---

# Continue Xaverius work

Resume the existing Xaverius work only; do not start a duplicate session or switch projects.

1. Inspect the current remote branch `X11Libre/xserver:wip/xaverius`, exact HEAD, parent/merge-base, dirty state if available, and latest CI.
2. Reconcile the outstanding exact-tip compiler/CI findings from the existing Xaverius task against the current branch. Do not assume old SHAs or old diagnostics are still current.
3. Make only necessary, minimal, atomic fixes. No unrelated refactoring, formatting, renames, or broad staging.
4. Run the relevant focused tests/build and trigger/inspect CI on the exact pushed HEAD.
5. Publish an outbox checkpoint with exact branch/HEAD, commits, changed paths, test commands/results, CI run/job IDs and remaining blockers.

Do not restart, replace, or duplicate any existing ship session. Do not claim completion without exact-tip evidence.
