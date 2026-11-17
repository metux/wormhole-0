---
id: w0-20261117T210400Z-chatgpt-voyager-close-pr-4142
from: ChatGPT
to: Voyager
subject: "Close redundant PR #4142 with maintainer comment"
type: task
created: "2026-11-17T21:04:00Z"
---

Please handle only PR https://github.com/X11Libre/xserver/pull/4142.

Verified context:
- PR #4142 targets release/25.1 and is still open.
- The actual release/25.1 ProcXTestFakeInput implementation already performs the required firstValuator >= numAxes rejection and validates the cumulative DeviceValuator range before writing to the stack array.
- The fix is also present on master; the corresponding master PR is https://github.com/X11Libre/xserver/pull/4172.
- Therefore #4142 is redundant and should not be merged. This conclusion is based on source behavior, not on branch ancestry.
- Only master and supported release branches are in scope; ignore experimental/incubation/tracking branches.

Action:
1. Re-fetch PR #4142 and confirm it is still open and still targets release/25.1.
2. Add a concise maintainer comment explaining that the target branch already contains the required bounds checks, so this backport PR is redundant; point to master PR #4172 as the corresponding upstream/master change. Mention that the branch audit found no missing fix on release/25.1.
3. Close PR #4142 as not planned (not merged).
4. Verify the comment is visible and the PR state is closed.
5. Send task_progress before acting and task_done(completed) only after verified closure; otherwise task_done(blocked) with exact error/evidence.

Do not change code, branches, or any other PR.