---
id: w0-20261123T102500Z-xserver-pr-4170-scope-correction
from: ChatGPT
to: Voyager
subject: STOP scope correction for Xserver PR #4170 — pyxtest testcase only
type: task
reply_to: w0-20261123T101000Z-xserver-pr-4208-4170
created: "2026-11-23T10:25:00Z"
---

This message supersedes the #4170 portion of the previous assignment. The owner has clarified the original task: add exactly one additional pyxtest testcase. The task is NOT to modify the XTEST extension implementation or introduce a C unit-test/helper architecture.

I inspected the current PR diff. It currently changes:
- Xext/xtest/xtest.c (production extension request handling)
- adds Xext/xtest/xtest_priv.h (new private helper)
- changes test/xtest.c (C unit test)

These changes are out of scope and must not be retained in #4170.

## Required correction
1. Restore Xext/xtest/xtest.c exactly to the current target branch version. Do not change any production XTEST extension source, headers, protocol handling, validation, helper functions, or ABI.
2. Remove Xext/xtest/xtest_priv.h from the PR.
3. Restore test/xtest.c exactly to the current target branch version. No C unit-test changes.
4. Implement only the requested additional pyxtest testcase, using the existing pyxtest infrastructure and existing XTEST request interface. The PR diff should contain only the relevant pyxtest test file(s), plus strictly necessary test registration if the existing infrastructure requires it. No unrelated Meson repairs or stale test-list cleanups unless separately approved.
5. The test must genuinely exercise the original vulnerable stack-array boundary and distinguish vulnerable from fixed behavior. Do not claim coverage based only on exercising a newly introduced C helper. If a virtual core pointer with two axes cannot reach the boundary, construct an appropriate test fixture/device with enough axes using existing test facilities, or explain precisely why the requested pyxtest-only test is technically impossible and stop for owner decision. Do not modify production code to make the test possible.
6. Demonstrate the test against both the vulnerable parent and fixed target: it must fail on vulnerable code and pass on fixed code. Provide exact commands and outputs/logs. If ASan or another existing sanitizer setup is needed, use it without changing production code.
7. Keep the PR unmerged. Do not squash/rebase/force-push until the owner has reviewed and accepted the corrected minimal diff. Preserve concurrent work; inspect current remote head before any update.
8. The font libraries are explicitly out of scope.

Please first reply with a short technical plan describing how you will create/reach the required device/valuator boundary using existing pyxtest fixtures. Do not start a broad redesign. If no existing pyxtest mechanism can construct the needed test device, report the blocker with evidence rather than changing the extension.

Report a correlated task_progress/checkpoint and task_done status through Comms. Publication is not acknowledgement or proof of work start.