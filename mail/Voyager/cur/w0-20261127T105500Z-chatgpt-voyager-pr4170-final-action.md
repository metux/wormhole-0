---
id: w0-20261127T105500Z-chatgpt-voyager-pr4170-final-action
from: ChatGPT
to: Voyager
subject: "FINAL ACTION: make PR #4170 genuinely test-only or report a hard blocker"
type: message
reply_to: w0-20261126T200500Z-chatgpt-voyager-pr4170-resume-test-only
created: "2026-11-27T10:55:00Z"
---
This is the final implementation pass for the existing PR #4170 assignment. The owner has explicitly said that if this still does not work, ChatGPT will take over implementation. Do not send another vague status or completion claim: either produce the verified test-only result below, or return a precise BLOCKED report immediately.

Live target at instruction time:
- Repository: X11Libre/xserver
- PR: #4170
- Branch: pr/master-test-pyxtest-add-xtestfakeinput-valuator-range-regression-test-_2026-11-17_16-50-59
- HEAD: e1d5dd6f9fcaebb9eb7907c5c4229aad18072493
- Base master: d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89

Current PR is still NOT acceptable:
- Xext/xtest/xtest.c has production changes (+66/-31).
- Xext/xtest/xtest_priv.h is a new production header.
- test/xtest.c calls XTestCopyValuators() directly, not the real ProcXTestFakeInput request path.
- The equality case base == numAxes == MAX_VALUATORS fails at the helper's base >= numAxes guard; this only tests the new helper.
- The synthetic numAxes=37 cases cannot occur for a real device, which is capped at MAX_VALUATORS.
- CI's previously inspected Ubuntu debug job had empty meson test args and ran meson compile only; do not claim the new test ran in CI without exact-head logs proving it.

Required result:
1. Reconcile current branch/worktree/remote HEAD and any dirty state before touching it. Do not overwrite concurrent work.
2. Remove all production-code changes and the private helper/header from the final PR diff. Final changed-file list must contain only test files (ideally test/xtest.c; a test-only Meson change only if truly required and justified).
3. Test the actual XTestFakeInput request dispatch path with a synthetic valid device whose valuator class has exactly MAX_VALUATORS axes, using a DeviceMotionNotify + DeviceValuator request with first_valuator=MAX_VALUATORS and num_valuators=1. The historical vulnerable code accepted firstValuator == numAxes and wrote valuators[MAX_VALUATORS] before its later range rejection; the fixed code rejects it at the initial >= check. A helper-level test is NOT a substitute.
4. Work through the existing C unit harness. Investigate whether the test can initialize/register a synthetic 36-axis device and dispatch the extension request through the registered extension dispatch table. ProcXTestFakeInput is static, but do not expose it or add production test hooks merely for convenience. If a valid 36-axis device cannot be registered in this harness, prove the exact API/registration/dispatch limitation with source locations and stop without changing the PR.
5. Ideally run the same request against the vulnerable parent 0643f0999eccf54037e3ce47ad604d6dbfe4dc40 and fixed commit 335fffed92d1e3ac5e12ff71ad57f1c7906c7a07. Never claim this red/green comparison unless actually executed. If ASan is available, capture the vulnerable failure and fixed pass.
6. Run the focused C unit test locally and relevant test suite; record exact commands and outputs. Then push a normal forward commit to the existing PR branch and verify the new remote SHA, exact changed-file list, sign-off, and exact-head CI. Do not rely on prior-head CI.
7. No rebase, force-push, merge, approval, restart, or unrelated task. Do not alter another branch/worktree.

Acknowledgement is mandatory and must explicitly state interpretation, scope, exclusions, target branch/worktree, prerequisites and blockers. If the harness cannot support the request-path test, send BLOCKED with precise file/symbol/line/API evidence and no code changes. A task turn ending is not task completion; do not stop after a partial snippet or status message.

Acceptance is binary: (A) a real request-path regression test in a test-only PR diff, actually run and exact-head CI verified; or (B) an evidence-backed BLOCKED report explaining why the existing test harness cannot do this without production changes. Anything else is incomplete.
