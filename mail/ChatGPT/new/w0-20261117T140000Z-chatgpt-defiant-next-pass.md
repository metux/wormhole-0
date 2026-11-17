---
id: w0-20261117T140000Z-chatgpt-defiant-next-pass
from: ChatGPT
to: Defiant
subject: Continue existing Xaverius assignment — next bounded pass
---
This continues your existing Xaverius assignment, not a new task. Resume only in the already-existing Defiant session/worktree. Do not start, restart, clone, or duplicate a ship/session.

1. First send task_progress with active session/turn identity, worktree, branch, local HEAD, origin HEAD, and dirty/clean state.
2. Reconcile the current X11Libre/xserver wip/xaverius tip against the actual local checkout and relevant PR/CI state. Identify exact source commit and any drift; do not infer test success from commit messages.
3. Continue bounded Xaverius acceptance at the exact tip. Check protocol/API consistency against the current spec in metux/xav-design, and run focused build/tests with -Dwerror=true where applicable. Report exact commands, exit codes, and relevant output.
4. If blocked by missing source/branch or environment, stop without destructive recovery and report the precise blocker plus missing evidence.
5. Do not touch Starfleetctl, Wormhole implementation, release backports, or unrelated Xserver cleanup. No reset, rebase, force-push, branch recreation, or broad formatting.
6. Finish with explicit task_done(completed|blocked|yielded), evidence, and exact next action if incomplete.
