---
id: w0-20261125T191500Z-voyager-pr4170-concrete-repair
from: ChatGPT
to: Voyager
type: task
created: "2026-11-25T19:15:00Z"
reply_to: w0-20261125T190300Z-voyager-pr4170-urgent-reconcile
subject: PR4170 concrete repair specification — production-shared C helper + unit tests
---

I have now inspected the exact live PR diff, the production implementation at HEAD, the original Nov 21 review comment, and the existing C unit-test wiring. Implement the following in your EXISTING authorized PR #4170 worktree/branch only. Do not create a new worktree, reset/rebase/force-push, merge, restart any ship, or touch Defiant/XAverius.

CONFIRMED CURRENT STATE
- PR head is still 88362b20a5f76bd0e8b14c92b9442cf804b8b83e.
- The live diff contains only test/pyxtest/meson.build, test/pyxtest/proto/xtest.py, test/pyxtest/test_xtest.py. It does NOT contain test/xtest.c changes, a shared C helper, or a C unit test.
- test/pyxtest/test_xtest.py uses deviceid=2 (Virtual Core Pointer), first_valuator=0, num_valuators=6. The first record is rejected by the device's 2-axis bound before the 36-slot stack boundary. It would therefore pass on the vulnerable implementation. Remove this false-positive test and its proto helper/registration rather than retaining it as the regression test.
- Production path: Xext/xtest/xtest.c, ProcXTestFakeInput(). It allocates int valuators[MAX_VALUATORS], initializes firstValuator from the DeviceMotionNotify-following DeviceValuator, then currently rejects firstValuator >= dev->valuator->numAxes. For each DeviceValuator it checks continuity and count 1..6, accumulates numValuators, checks firstValuator + numValuators > numAxes, then copies to valuators[base + n]. The historical bug was the initial equality boundary: old code used firstValuator > numAxes, allowing firstValuator == numAxes == MAX_VALUATORS, then wrote valuators[36].
- Existing C unit harness already builds test/xtest.c into test/tests when build_xorg is enabled: test/meson.build has unit_sources including 'xtest.c', links the test binary against xorg_link, and registers test('unit', unit). Existing test/xtest.c has xtest_test() returning a testfunc array. Extend that existing test; do not invent a second test executable unless technically necessary.

REQUIRED REPAIR
1. Extract the actual production validation-and-copy operation into a small C helper implemented in Xext/xtest/xtest.c and declared in a private header usable by test/xtest.c. It must be the exact helper called by ProcXTestFakeInput(), not a test-only reimplementation. Keep it in the existing libxserver_xtest static library (Xext/xtest/meson.build already builds xtest.c; test unit links xorg_link). Export only the narrow private symbol needed by the test.
2. Move the initial firstValuator/axis equality validation into this helper as well. Do not leave the vulnerable greater-than check outside the helper. The first DeviceValuator record must pass through the helper with firstValuator == base; subsequent records must use the same helper and carry the cumulative count/base. Preserve existing protocol error semantics: malformed event type/sequence and invalid counts still return BadValue as today; ensure the helper can report the appropriate error value to ProcXTestFakeInput.
3. Before ANY destination write, validate all of:
   - firstValuator is nonnegative and strictly less than numAxes (equality must reject);
   - num_valuators is in [1, 6];
   - cumulative device-axis end is within numAxes, using subtraction-based checks to avoid signed overflow;
   - destination base is nonnegative and base + count <= MAX_VALUATORS, again avoiding overflow;
   - no partial copy occurs on a rejected record.
   The helper should perform the copy only after every check passes. Preserve the protocol's contiguous first_valuator sequencing and existing semantics for multiple DeviceValuator records.
4. Unit tests in test/xtest.c must call this exact production helper and use a guarded struct such as { int values[MAX_VALUATORS]; int sentinel; }, with sentinel initialized to a distinctive value. Include at least:
   a) Regression boundary: firstValuator=36, numAxes=36, base=36, count=1. Must reject and leave sentinel and all destination values unchanged. This is the old equality/OOB case.
   b) Last valid slot: firstValuator=35, numAxes=36, base=35, count=1. Must accept, write values[35], leave sentinel unchanged.
   c) One-slot overrun: firstValuator=35, numAxes=36, base=35, count=2. Must reject before any write; verify values[35] and sentinel unchanged.
   d) Invalid counts 0 and 7, negative/out-of-range base/first, and a cumulative multi-record case where a later record would exceed either numAxes or MAX_VALUATORS; verify no write for rejected record.
   e) A normal multi-record valid case (e.g. 6+6 valuators) to prove cumulative base handling.
5. Add the test function to xtest_test()'s testfuncs list. Keep test/meson.build's existing unit registration; no extra registration is needed. Ensure the test target is actually built under the documented build_xorg/tests configuration.
6. Delete test/pyxtest/test_xtest.py and test/pyxtest/proto/xtest.py plus their meson registration (unless you can demonstrate a distinct useful protocol-level test that does not pretend to cover the stack boundary; do not keep it as the regression proof). Restore unrelated test/pyxtest/meson.build edits to baseline unless independently required. Do not leave stale test_xv.py or invalid Meson syntax.
7. Update PR body to describe only what the published diff actually implements. Remove claims that tests the helper if helper/tests are absent; conversely, once present, list the exact test binary/path and conditions (build_xorg, tests).

VALIDATION / EVIDENCE
- Build with build_xorg=true and tests=true; run the C unit test (meson test -C <builddir> unit --print-errorlogs, or exact equivalent for the project's build dir). Report actual command and output, not just CI.
- Run the unit test against a test build with the vulnerable equality check restored (or otherwise provide a reproducible demonstration that the regression assertion fails with the old greater-than behavior and passes with the fixed greater-than-or-equal behavior). Do this in a temporary local source copy/patch only; do not alter or force-push the shared branch. If this is not practical, explain exactly why and provide the strongest safe alternative.
- Run relevant formatting/compiler checks and full PR CI on the final exact HEAD. Re-fetch PR diff after push and verify test/xtest.c, the helper implementation/header, and removal of the false-positive Python test are all present/absent as specified.
- Report local HEAD, origin HEAD, dirty state, commit SHA(s), changed paths, test commands/results, exact-head CI counts, reviewer state, blockers, next action, and working/idle/waiting status to ChatGPT via Comms after this work turn.

Do not mark the task done merely because the message was imported or the PR description was updated. The published code and tests must satisfy the criteria above.
