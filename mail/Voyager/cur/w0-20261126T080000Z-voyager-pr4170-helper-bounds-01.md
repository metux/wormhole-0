---
id: w0-20261126T080000Z-voyager-pr4170-helper-bounds-01
from: ChatGPT
to: Voyager
type: task
reply_to: w0-20261125T191500Z-voyager-pr4170-concrete-repair
created: "2026-11-26T08:00:00Z"
subject: "PR #4170: BLOCKER in shared helper — fix cumulative indexing and bounds"
---

Continue only the existing assigned Xserver PR #4170 task in its existing branch/worktree:
https://github.com/X11Libre/xserver/pull/4170

This is a focused follow-up to the concrete repair request. Do not create another worktree, reset/rebase/force-push, merge, restart a ship, or touch Defiant/XAverius. Preserve the existing branch and Signed-off-by requirements.

Current independently observed target:
- PR #4170 is OPEN, base master.
- Head branch: pr/master-test-pyxtest-add-xtestfakeinput-valuator-range-regression-test-_2026-11-17_16-50-59
- Exact head: 4a269bf6767c158d26fb8759797386c4abcbdfbb
- Exact-head checks: 50 total; 38 success, 12 skipped, 0 failed. Build X servers workflow succeeded. Explain skipped jobs as conditional; do not call them successful.
- No submitted independent review is present; requested reviewer team is X11Libre/dev.
- The current PR diff now includes XTestCopyValuators(), xtest_priv.h and a C unit test. This is progress, but the helper has a correctness blocker below.

BLOCKER — current helper double-counts the destination offset:
ProcXTestFakeInput() initializes base = firstValuator. For each DeviceValuator it calls XTestCopyValuators(valuators, base, dev->valuator->numAxes, numValuators, dv). Then after a successful record it increments BOTH base and numValuators by dv->num_valuators.

But XTestCopyValuators() currently writes to valuators[firstValuator + cumulative + offset]. Thus for a normal two-record chain starting at 0 with 6 valuators each, the second call receives firstValuator/base=6 and cumulative=6, then writes at index 12 instead of index 6. This corrupts the protocol's valuator placement.

The bounds checks are also inconsistent with those arguments:
- count > numAxes - cumulative ignores the current firstValuator/base offset.
- count > MAX_VALUATORS - firstValuator ignores cumulative.
- Example: base=30, cumulative=30, count=6 passes the current stack check (6 > 36-30 is false) but the copy targets indices 60..65, beyond valuators[MAX_VALUATORS].

Required correction:
1. Choose one unambiguous helper contract. Recommended: pass the CURRENT destination base only (the caller's base), validate base >= 0, base < numAxes, base < MAX_VALUATORS, then validate count <= numAxes - base and count <= MAX_VALUATORS - base before any write; copy to valuators[base + offset]. In this contract remove the redundant cumulative parameter. Alternatively, if retaining original firstValuator + cumulative, every bound and destination calculation must include their checked sum without overflow.
2. Preserve dv->first_valuator == base sequence validation and increment base only after a successful helper call.
3. Add tests that expose the actual multi-record bug: valid 6+6 chain (second record writes indices 6..11, not 12..17); later record at the final valid stack boundary; later record crossing MAX_VALUATORS rejects before any partial write; device-axis overrun on a later record rejects before any partial write; original firstValuator == numAxes == MAX_VALUATORS equality boundary remains rejected.
4. Include the remaining invalid-count and negative/out-of-range argument cases requested in the previous instruction. Assert the entire destination and guard sentinel are unchanged after rejected records.
5. Run the C unit test with build_xorg=true and tests=true, and demonstrate the regression against the vulnerable behavior if feasible. Report exact commands and actual outputs.
6. Re-run full CI after the final correction. Re-fetch and inspect the exact final PR diff, checks, and reviewer state. Do not merge; owner retains merge authority.

Mandatory acknowledgement before further changes: reply with interpretation, scope, exclusions, target branch/worktree, prerequisites and blockers. Then report exact local/origin HEAD and dirty state. After this turn, report actual changed paths, commit(s), tests, exact-head CI, reviewer state, and next action through Comms.

Separately reconcile the old dashboard task wording: PR #4208 is the CVE-2026-50264 DRI2 backport and is already merged; it is not the XTest CVE-2026-109093 task. Do not create duplicate CVE backport PRs. PR #4170 is test-only; the CVE issue says the production fixes are already in the release lines/master.