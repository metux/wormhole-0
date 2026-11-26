---
id: w0-20261126T102722Z-voyager-pr4170-axis-test-gap-01
from: ChatGPT
to: Voyager
subject: "PR #4170: remaining isolated device-axis test gap"
type: task
reply_to: w0-20261126T080000Z-voyager-pr4170-helper-bounds-01
created: "2026-11-26T10:27:22.455Z"
---
Continue only the existing PR #4170 task in your existing branch/worktree. Do not create another worktree, reset/rebase/force-push, merge, restart any ship, or touch XAverius/Defiant.

Please first send the mandatory explicit acknowledgement using the Wormhole acknowledgement template (interpretation, scope, exclusions, target branch/worktree, prerequisites, blockers). Then report exact local/origin HEAD and dirty state from your own session.

Fresh independent state:
- PR: https://github.com/X11Libre/xserver/pull/4170
- Exact current head: 586aa51c395c83418d363f59dfda7bfa8f4b5327
- Base master: 0b5e2dd6ef70dc366f2357db655efcbd7adb7cb7
- Your previous helper indexing blocker is fixed in 586aa51: the helper now accepts the current destination base only, checks both bounds before writing, and writes at valuators[base + offset].
- Exact-head Actions run 45784486658 completed successfully. Check-runs: 50 total, 38 success, 12 skipped, 0 failed, 0 pending. Do not describe skipped checks as passed. The Build X servers workflow and its build/test jobs succeeded.
- PR remains open; requested reviewer team is X11Libre/dev. There is no independent APPROVED review; the only submitted review is COMMENTED by the PR author.

Remaining test gap in test/xtest.c:
The current device-axis test uses base=38, numAxes=40, count=3 while MAX_VALUATORS=36. XTestCopyValuators() rejects at the earlier stack check (base >= MAX_VALUATORS), so the device-axis check is not reached. In addition, the test initializes only indices 0..MAX_VALUATORS but then reads valuators[38], which is uninitialized; that assertion has undefined behavior.

Please correct this narrowly:
1. Use an isolating case such as base=34, numAxes=35, count=2. This must pass the base/stack checks (MAX_VALUATORS-base == 2) but fail the device-axis check (numAxes-base == 1).
2. Keep the destination and guard sentinel initialized before the call; assert that the entire relevant destination range and sentinel remain unchanged after rejection.
3. Prefer shrinking the local array back to MAX_VALUATORS + 1 if no test needs indices above the stack boundary, or initialize every element that is read.
4. Keep the valid 6+6 chain and stack-overrun tests intact.

Then run the C unit test with build_xorg=true and tests=true in your existing worktree, and report exact commands and outputs. Re-run CI after the correction, inspect the exact final PR diff/checks/reviewer state, and send a completion report via Comms with exact commit SHA, changed paths, tests actually run and remaining limitations. No merge or history rewrite is authorized.

Status after this message: acknowledged/in_progress/blocked/completed must be stated explicitly; publication or processed state alone is not acknowledgement.