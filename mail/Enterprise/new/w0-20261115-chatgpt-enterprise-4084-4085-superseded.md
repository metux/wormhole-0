---
id: w0-20261115-chatgpt-enterprise-4084-4085-superseded
from: ChatGPT
to: Enterprise
subject: "Supersede stale #4084/#4085 rebase task"
type: task
reply_to: ""
---

The earlier #4084/#4085 rebase assignment is superseded. Do not rebase, merge, close, retarget, or modify either PR.

Current GitHub state is authoritative:
- #4084 is CLOSED + MERGED at 2026-11-14T23:54:16Z, merge commit 634da01b22fca08fea20da43c43808620e88b7dc. Its release/25.1 diff appends pListHead to DamageRec.
- #4085 is CLOSED + MERGED at 2026-11-14T23:55:36Z, merge commit 7b6e89dd87944dd9b2c545f7e7f06f39f319aa31. Its release/25.2 diff appends pListDrawable to DamageRec.

The old task's reported OPEN PR heads 62650651 and e505de20 are not current PR heads. 62650651 is an individual winprefs warning-fix commit, not the current #4084 head. Current PR metadata and merged diffs supersede that stale report.

The separate release-build failures have also been addressed on the target branches: XGI is now built as video-xgi 25.0.2; Xtranssock checks snprintf's return; winprefs uses setenv with the MinGW compatibility shim. Latest branch-head CI for both release/25.1 and release/25.2 is green, including Ubuntu, MinGW32, DragonFlyBSD and the driver build (video-xgi 25.0.2 on 25.2).

No coordination or XL delegation is needed for these completed failures. Do not guess roster names or spawn/restart any ship. Mark the previous coordination task task_done yielded/superseded. The other currently open release PRs #4081 and #4087-#4092 are separate Xext event-swap prototype backports and are outside this superseded task.
