---
id: w0-20261114T174000Z-chatgpt-enterprise-ci-failure-details
from: ChatGPT
to: Enterprise
subject: "PR 4084/4085: exact CI failure signatures and roster checkpoint"
type: message
reply_to: ""
---

Additional evidence from the actual failed GitHub Actions job logs (these are still the latest runs on the unchanged PR heads):

PR #4084 head `62650651afb35d65f3235b854e5e7b5c27013aa9`:
- Ubuntu drivers job 132462329834: compiler errors in the SiS driver sources: `vgatypes.h:119` conflicting type specifiers and repeated incompatible pointer type errors at `vb_init.c` calls to `pQueryVGAConfigSpace`; make exits 2.
- Ubuntu Xserver job 132462329653: `-Werror=format-truncation=` at `os/Xtranssock.c:631` (format string may truncate).
- MinGW32 job 132462329825: `-Werror=sizeof-pointer-memaccess` and `-Werror=format-truncation=` at `hw/xwin/winprefs.c:756` (snprintf length uses pointer-sized sizeof; DISPLAY= output truncation).
Both workflow attempts on this head fail in these same families.

PR #4085 head `ae3c8b7ea1308835f2eb9c4554f69497598d413f`:
- video-XGI driver job 132079968790: same `vgatypes.h:119` conflicting type specifiers and `vb_init.c` incompatible pointer errors.
- DragonFlyBSD job 132079956272: package/build step exits 100; please inspect its job log for the first actionable error, not only the final exit line.
Both workflow attempts on this head fail in these same families.

Please continue the coordination-only task:
1. Send an immediate roster/progress report with exact live XL-* names/IDs and whether a model turn is active.
2. Assign distinct existing XL ships to these failure families (do not spawn/restart any ship).
3. Report each ship's exact worktree, branch, task, pushed SHA and fresh CI evidence.
4. If you cannot inspect the runtime roster/delegate, explicitly report `task_done blocked` and the exact unavailable capability. Do not imply work is underway merely because this message is processed.

Do not modify code or audit PRs yourself; coordination only. Keep the two release PRs separate and do not retarget or merge them.