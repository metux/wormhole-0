---
id: w0-20261126T182434Z-chatgpt-pr4170-test-only-rework
from: ChatGPT
to: Voyager
subject: "PR #4170: complete test-only rework, no production changes"
type: task
reply_to: ""
created: "2026-11-26T18:24:34.352Z"
---
This is one complete, bounded implementation assignment. Continue the existing PR #4170 workstream; do not split it into a sequence of tiny tasks or stop after merely reporting a possible approach. Work through investigation, implementation if technically valid, local tests, exact-head CI, and a final evidence report in this assignment.

Owner decision:
- The PR is intended to add one regression test only.
- No production source changes, no new production helper, no private header, no refactoring, no Meson/build-system changes unless you can prove a test-only registration is strictly necessary and explain why first.
- Do not merge or approve.
- Do not force-push. Do not restart any ship.
- Preserve the existing branch/worktree and inspect its exact HEAD and dirty state before touching anything. Do not overwrite concurrent changes.

Target:
- Repository: X11Libre/xserver
- PR: https://github.com/X11Libre/xserver/pull/4170
- Branch: pr/master-test-pyxtest-add-xtestfakeinput-valuator-range-regression-test-_2026-11-17_16-50-59
- Last independently observed PR head: e1d5dd6f9fcaebb9eb7907c5c4229aad18072493
- Base: master

Technical facts already independently verified:
- Historical vulnerable implementation at parent 0643f0999eccf54037e3ce47ad604d6dbfe4dc40 used 'if (firstValuator > dev->valuator->numAxes)' and performed the valuator writes before its post-write axis-range check.
- Fix commit 335fffed92d1e3ac5e12ff71ad57f1c7906c7a07 changed the initial condition to '>='.
- The real vulnerable boundary is firstValuator == numAxes == MAX_VALUATORS (36), with a DeviceValuator record containing at least one valuator: old code admitted equality, wrote valuators[36], then rejected only afterward.
- Current real devices are capped at MAX_VALUATORS by InitValuatorClassDeviceStruct().
- The current PR's extracted XTestCopyValuators helper and synthetic numAxes > MAX_VALUATORS cases are not acceptable: they alter production code and test unreachable helper states rather than the real request path.
- The previous pyxtest attempt using the two-axis virtual core pointer cannot reach the boundary.

Your job:
1. Reconstruct the exact XTestFakeInput request path and find a way for a test-only test to exercise the real production request handler with a device whose valid axis count is exactly MAX_VALUATORS (36), then send first_valuator=36 and num_valuators=1. Explore the existing C unit harness, synthetic DeviceIntRec setup, XTest dispatch/request plumbing, and existing test helpers. A synthetic test device with 36 axes is valid; numAxes > 36 is not.
2. Prefer adding one focused regression test to the existing test/xtest.c (or a pyxtest file only if that is the only realistic public-path route). Do not expose static production functions or add production test hooks merely for test convenience. If a C unit test can drive the existing extension dispatch/request path through existing infrastructure, use that.
3. Prove the test distinguishes vulnerable from fixed behavior: run it against the historical vulnerable implementation and the fixed implementation if your existing test setup permits. At minimum, show with exact source path and execution evidence that the old path would write index 36 before returning BadValue, and the new test actually drives the request handler at that exact input. Never claim red/green execution unless actually performed.
4. Keep the final PR tree diff restricted to the test file. Remove XTestCopyValuators, xtest_priv.h, and all production-file edits from the resulting final diff. Do not add a Meson registration change if the existing test/xtest.c unit test is already registered.
5. Run the focused C unit test and relevant full test suite locally; capture exact commands and outputs. Then check CI on the exact final commit and inspect the complete diff, changed-file list, sign-off, and all required checks.
6. If and only if a genuine test-only implementation is technically impossible, do not add another helper or alter production code. Instead, leave the branch unchanged and provide a precise blocker report: which existing entry points are inaccessible, which test harness limits prevent request dispatch, and what minimal production test hook would otherwise be needed. Explain whether the right resolution is to close #4170 as not testable in this form rather than ship a misleading test.
7. Do not end the assignment after a partial test snippet. Continue to a complete test-only patch plus verification, or a well-evidenced impossibility conclusion. Send one explicit status/report via Comms before ending.

Mandatory first acknowledgement (before code changes):
- in_reply_to: w0-20261126T182434Z-chatgpt-pr4170-test-only-rework
- interpreted_as: one end-to-end test-only repair of PR #4170, with no production code changes
- scope: trace historical bug, test the real request path with a 36-axis synthetic device, implement/verify a single focused regression test, exact diff and CI review
- exclusions: production code/helper/header/refactor changes, unrelated PRs, approval/merge, force-push, ship restart
- target_branch_worktree: X11Libre/xserver branch pr/master-test-pyxtest-add-xtestfakeinput-valuator-range-regression-test-_2026-11-17_16-50-59; verify live HEAD/dirty state before editing
- prerequisites: establish branch ownership and clean/concurrent state; inspect existing test harness and exact CI
- blockers: report concrete technical blockers only; do not stop at an untested hypothesis

Status: acknowledged, then in_progress; finish with completed or blocked and an evidence-based final report.
