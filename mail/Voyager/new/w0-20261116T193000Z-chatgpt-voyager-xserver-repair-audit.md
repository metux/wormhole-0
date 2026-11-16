---
id: w0-20261116T193000Z-chatgpt-voyager-xserver-repair-audit
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T19:30:00Z
---

Continue the existing X11Libre backport repair work in your EXISTING session and worktree. Do not restart or duplicate the ship. Report task_progress immediately with session/worktree/branch/HEAD before work.

The latest PR state shows the repair is NOT complete:

1. #4147 release/25.1 still has a materially broken diff. It rewrites KdCreateScreenResources but also removes unrelated KdCloseScreen behavior: DPMS restore, video restore, finiCursor, OS Disable and other lifecycle code. Rebuild this narrowly from exact base 15bef074f91b5fae9d7a9436ffd91a59a9082666 and intended upstream commits 550b9ed0dd38546b973a0b567868119422d1f370 + 47d68e7c57ef7a341ff254d3f3fb4eb6f72c3ce3. Preserve all unrelated close/cleanup behavior. One squashed commit, force-with-lease only after checking exact current head 78fe2b0ce8413dbfad8f7294ad57081cde46e7f9.

2. #4157 release/25.2 has the same serious collateral deletions, plus removes free(pScreenPriv) from KdCloseScreen. Rebuild independently from exact base d92f94c64f47fb5014fdc702db56938c37e625c7. Preserve all unrelated behavior. One squashed commit, force-with-lease only after checking exact current head b673e1d17d6311162b77b5690c5eb3ca669f6cff.

3. #4116 release/25.0: review current exact diff/head 5154a7791dbb8323c21c4543090cfb32af8f7d13 against the target-branch implementation and original CVE-2026-109093 fix. Ensure validation uses the correct first_valuator/base semantics and does not introduce an off-by-one or weaken the existing axis bounds. Do not accept the stale automated review as current evidence; fetch exact-head CI and inspect the actual source. Fix only if necessary, narrowly.

4. Reconfirm #4153 CVE-2026-109094 applicability on exact current master and release/25.2 tips; inspect dispatch registration and swapped-client handling. Do not edit #4153 unless you establish vulnerability.

5. #4125, #4148, #4156 GetScreenPixmap hook changes remain on hold pending Stefan's answer. Scan other open PRs for the same API migration and report candidates; do not submit them.

For every repaired PR: inspect full diff against exact base, run focused tests/builds where possible, fetch CI for exact pushed SHA, report SHA and links. Do not merge. Send task_done only when every assigned item is completed or explicitly mark blocked with reason.