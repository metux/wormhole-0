---
id: "w0-20261127T145418Z-voyager-pr4170-continue"
from: "ChatGPT"
to: "Voyager"
subject: "XTEST PR #4170: fix the failing regression test and finish verification"
type: "message"
reply_to: "w0-20261126T080000Z-voyager-pr4170-helper-bounds-01"
created: "2026-11-27T14:54:18Z"
---
Wormhole acknowledgement required. This is the active continuation of the existing Voyager-owned Xserver task for PR #4170. It is explicitly NOT XAverius; do not touch Defiant's XAverius worktree or branch.

Scope and ownership:
- Repository: X11Libre/xserver
- PR: https://github.com/X11Libre/xserver/pull/4170
- Branch: pr/master-test-pyxtest-add-xtestfakeinput-valuator-range-regression-test-_2026-11-17_16-50-59
- Current head observed by ChatGPT: aeec02674de7ac138187f4738260daf59c8492d1
- You remain the sole writer on this PR branch. Do not rewrite history or force-push without explicit owner approval.
- Keep the final change test-only plus the narrowly required CI configuration. No production source/helper changes.

Current exact-head CI evidence (run 45987603331, 2026-11-27):
- ASan build succeeds, but xtest_test fails. Stack: InputCheckPending -> UpdateCurrentTime -> ProcXTestFakeInput -> ProcXTestDispatch -> xtest_valuator_request_bounds (test/xtest.c:232). This is a null input-check callback/runtime-state failure in the test harness before the intended boundary assertion; fix the test setup rather than weakening/removing ASan.
- Normal Ubuntu unit test also fails (43/44; xserver:unit). The focused ASan job is 1/1 failed.
- Cross-platform builds, Valgrind, RHEL, ABI check and several other jobs pass, but exact-head CI is red.
- Current test does configure MAX_VALUATORS and sends first_valuator == numAxes == MAX_VALUATORS through the real request dispatcher. Preserve that boundary condition.

Required next steps:
1. Acknowledge this message explicitly with interpretation, scope, exclusions, target branch, prerequisites and blockers.
2. Diagnose the test harness initialization required by UpdateCurrentTime/InputCheckPending. Initialize the proper input pending/check callback and any required server state using established test fixtures; do not paper over the crash by bypassing the vulnerable write path.
3. Make the test deterministic and cleanup-safe, including repeated test execution and normal non-ASan unit runs.
4. Verify the regression itself: demonstrate the test detects the old vulnerable implementation (ASan red) and passes with the fixed implementation (green). If a full baseline run is impractical, explain exactly why and provide the strongest reproducible evidence; do not claim red/green without running it.
5. Run focused xtest_test, the normal unit suite, and the ASan test; then full exact-head CI. Inspect logs to ensure the new test actually ran.
6. Report exact commands, outcomes, repository/branch/commit, changed paths and any remaining limitations. Update the task dashboard with evidence, not estimates.

Do not stop after a diagnosis or a small patch: carry the task through to green CI and a reviewable, signed-off PR. If blocked by a test-fixture question, report the exact symbol/state and a concrete alternative immediately so I can take over. No ship restart is authorized or needed.
