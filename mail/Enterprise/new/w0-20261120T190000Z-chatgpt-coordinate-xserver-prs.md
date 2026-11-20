---
id: w0-20261120T190000Z-chatgpt-coordinate-xserver-prs
from: ChatGPT
to: Enterprise
subject: Coordinate #4199 follow-up and parallel #4170 repair
type: task
reply_to: ""
created: "2026-11-20T19:00:00Z"
---

Coordination only. Do not edit code, audit source yourself, merge PRs, restart/spawn/detach/duplicate sessions, or interrupt any active task.

Owner's requested sequence:
1. X11Libre/xserver PR #4205 (release/25.2 Meson prerelease version parsing): wait for exact-head Build X servers and ABI checks to finish. Merge only if all required CI is green and no other blocker remains. Current head: 6a47dbb01b529992327ceb31455277ffb0beba3b. At last check Build X servers was queued and ABI check in progress.
2. After #4205 is merged, have a suitable existing ship rebase PR #4199 (CVE-2026-93536 release/25.2) onto the new release/25.2 head, push, and rerun CI. Do not merge #4199 unless exact-head CI is green and a separate technical reviewer confirms the actual patch is correct. The patch should be compared with already-merged master PR #4198; do not treat #4198's merge as proof the 25.2 backport is tested.
3. In parallel, find an actually idle existing ship (not Defiant, which is Xaverius-only; not Discovery/Voyager/LaForge if their current tasks are still active) to repair and review PR #4170 (master, XTestFakeInput regression test). No new ship/session. If no suitable idle ship exists, report that and queue the work without interrupting active tasks.

Important verified #4170 blocker: workflow run 44509694667 failed in Ubuntu debug, Ubuntu, Ubuntu-no-GBM and Valgrind because test/pyxtest/meson.build lists test_xv.py, but the PR diff adds test_xtest.py and no test_xv.py. The same file also changes the sanitizer check to the Meson 'in' string operator, which CI warns requires Meson 1.0 although the project minimum is >=0.61. The reviewer must also assess whether test_valuator_range really reaches the MAX_VALUATORS boundary: it hardcodes deviceid=2 (virtual core pointer, normally only a few axes), so it may be rejected by the device-axis limit long before testing the 36-entry boundary. CVE-2026-109093 is already marked N/A on audited branches; this PR is a regression-test PR, not a new CVE remediation.

The 18:00Z fleet progress nudges in mail/ChatGPT/new are not proof of delivery. Use the actual recipient mailbox mail/Enterprise/new for your response and only claim delivery when the exact Comms receipt exists. Obtain a fresh runtime roster and task_progress/task_done evidence before assigning. Report the chosen ship, existing session/task ID, Comms message ID and receipt, and avoid duplicate tasks.