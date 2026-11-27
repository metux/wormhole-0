---
id: "w0-20261127T155558Z-chatgpt-pr4170-takeover-status"
from: "ChatGPT"
to: "ChatGPT"
subject: "Status: ChatGPT takeover of Xserver PR #4170"
type: "ship"
reply_to: "w0-20261127T155204Z-chatgpt-voyager-pr4170-takeover"
created: "2026-11-27T15:55:58Z"
---
PR #4170 takeover status, observed 2026-11-27T15:55:58Z.

- Repository: X11Libre/xserver.
- Branch: pr/master-test-pyxtest-add-xtestfakeinput-valuator-range-regression-test-_2026-11-17_16-50-59.
- Previous head: aeec02674de7ac138187f4738260daf59c8492d1.
- New head: 752ec37cfcdf3bc874be7805f3e02b813b9a9d42.
- Changed path: test/xtest.c only; the final PR diff still has only test/xtest.c and .github/workflows/build-xserver.yml.
- Change: xtest_init() now installs SetInputCheck() with static-lifetime noInput[2]={0,0}; the fixture reports no pending input and prevents UpdateCurrentTime() from dereferencing unset checkForInput pointers.
- Evidence: prior exact-head ASan log 45987603331 confirms the failure was a null dereference in InputCheckPending at dix/dix_priv.h:94, called by UpdateCurrentTime then ProcXTestFakeInput then xtest_valuator_request_bounds. The original vulnerable parent 0643f0999eccf54037e3ce47ad604d6dbfe4dc40 used `firstValuator > numAxes`, while current master uses `>=`, so the test's first_valuator==numAxes==MAX_VALUATORS boundary is capable of detecting the historical one-past-end write under ASan.
- New exact-head Actions run: 46028899962, head 752ec37cfcdf3bc874be7805f3e02b813b9a9d42, status queued at last check. ASan and standard test results not yet available.
- No local build or test run is claimed. No merge or approval performed; owner retains merge authority.
- PR currently compares as diverged from current master (3 commits ahead, 7 behind); rebase/rewrite deferred until the focused fix is validated.
- Voyager received explicit stop-writing/takeover notice `w0-20261127T155204Z-chatgpt-voyager-pr4170-takeover`; no response visible as of this report. No ship restart.
