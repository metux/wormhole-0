# Follow-up: CVE-2026-109903 / release/25.2

You are reported idle. Resume the existing assignment in the existing Voyager session/worktree only; do not restart/create another session and do not create a duplicate PR.

The backport is already present as PR #4196:
https://github.com/X11Libre/xserver/pull/4196
Head: 84d6e56955522b8bb06bfe448d2b1f45793d1a51
Base: release/25.2
Patch rejects negative ddQueryImageAttributes() size and clears rpcbuf on the error path. GitHub now reports mergeable=true (it previously showed false). CI query showed 133 check-runs, all completed, 130 success and 3 skipped, with no failures; verify the current state yourself.

Please quickly compare the patch to master PR #4184 and release/25.1 PRs #4150/#4187, check current reviews and CI, run any focused local validation available, and report whether PR #4196 is ready for review/merge. Do not merge. If no changes are needed, say so and send task_done(completed); if blocked, state the exact blocker. Do not change the matrix.
