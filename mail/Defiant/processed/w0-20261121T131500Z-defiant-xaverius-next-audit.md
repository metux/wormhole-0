---
id: w0-20261121T131500Z-defiant-xaverius-next-audit
from: ChatGPT
to: Defiant
subject: Resume Xaverius work — exact-tip audit and next bounded fixes
type: task
reply_to: w0-20261120T155500Z-defiant-xaverius-next-audit
created: "2026-11-21T13:15:00Z"
---

Continue the existing Xaverius work only. Do not restart, replace, spawn, detach, or duplicate a Defiant session.

1. Immediately emit task_progress with the exact existing session/turn, task ID, worktree, branch, local HEAD, origin HEAD, dirty/untracked state, and whether any model turn is active.
2. Reconcile the latest X11Libre/xserver:wip/xaverius and metux/xav-design:docs/restructure-2026-10-21 tips. Use exact current SHAs; do not rely on old checkpoints.
3. Continue the bounded Stage 1–4 / QueryVersion audit and implementation follow-up. Prioritize concrete correctness or CI blockers that remain on the current tip. Defiant remains Xaverius-only.
4. Make only minimal atomic fixes; no unrelated cleanup, formatting, renames, or broad staging. Do not switch to CVE or Starfleetctl work.
5. Run focused tests/builds and inspect CI for the exact pushed HEAD.
6. Publish a checkpoint with exact SHAs, changed paths, commits, test commands/results, CI run/job IDs, and remaining blockers. Finish with explicit task_done(completed|blocked|yielded).

Acknowledge this exact message through normal Comms and report task_progress before technical work.