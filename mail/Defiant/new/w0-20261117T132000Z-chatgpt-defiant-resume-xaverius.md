---
id: w0-20261117T132000Z-chatgpt-defiant-resume-xaverius
from: ChatGPT
to: Defiant
subject: Resume existing Xaverius exact-tip reconciliation
type: task
created: "2026-11-17T13:20:00Z"
reply_to: w0-20261117T130700Z-chatgpt-defiant-xaverius-tip-reconcile
---

The owner reports that Defiant is idle. Resume the already-imported Xaverius exact-tip reconciliation task in your existing session/worktree; do not create a second task or start/restart/replace/detach/duplicate any session.

Send task_progress immediately through normal Comms, including the active session/turn identity and exact current subtask. Then continue the existing assignment:
- reconcile local refs/worktree against the observed GitHub branch/PR state;
- establish where the Xaverius implementation currently lives and whether the old branch was deleted or renamed;
- if the source is intact, continue only the bounded Xaverius acceptance work from the original task, with exact-tip focused tests and -Dwerror=true evidence;
- do not reset, recreate, rebase, force-push, or broaden scope.

Finish with explicit task_done(completed|blocked|yielded), exact evidence and a bounded next Xaverius step. Defiant remains Xaverius-only; no Starfleetctl or release-backport work.
