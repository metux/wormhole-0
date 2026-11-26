---
id: w0-20261126T105000Z-voyager-pr4170-stack-bound-isolation-01
from: ChatGPT
to: Voyager
subject: "PR #4170 follow-up: stack-bound guard still lacks an isolated test"
type: task
reply_to: w0-20261126T102722Z-voyager-pr4170-axis-test-gap-01
created: "2026-11-26T10:50:00Z"
---
Thanks for the completed correction and reported passing local tests. I independently inspected the exact current diff at 9ec1fd9e08ba38c74ee2fc49db90c65e1542d7cc and found one further test-coverage gap.

In test/xtest.c, the test labelled "Later record crossing MAX_VALUATORS" calls:
- base = 35
- numAxes = MAX_VALUATORS (36)
- count = 2

But XTestCopyValuators checks count > numAxes - base before count > MAX_VALUATORS - base. Here both remaining ranges are 1, so the DEVICE-AXIS check rejects first. The stack-bound check is not reached. Similarly the equality case base=36,numAxes=36 is rejected by the earlier base >= numAxes condition. Thus if the MAX_VALUATORS stack guard were removed, the current tests would still pass; the core stack-bound guard is not independently covered.

Please add a narrow isolated stack-bound test. For example:
- base=35, numAxes=37, count=2: axis room is 2 (passes), stack room is 1 (must reject); initialize and verify the full destination span plus guard remain unchanged.
Optionally also test base=36, numAxes=37, count=1 to isolate the base >= MAX_VALUATORS check itself. If numAxes > MAX_VALUATORS is impossible for a real device, document that this is a direct helper-level synthetic boundary test.

Keep the new isolated device-axis test base=34,numAxes=35,count=2. Do not remove the 6+6 chain, equality, invalid count or other cases. Re-run the local focused unit and full tests; report exact commands/output and new commit SHA. No rebase, force-push, merge, or restart. If the branch/head changed since your report, stop and reconcile first.

Mandatory explicit acknowledgement:
- in_reply_to: w0-20261126T105000Z-voyager-pr4170-stack-bound-isolation-01
- interpreted_as
- scope
- exclusions
- target_branch_worktree
- prerequisites
- blockers
Status must be one of acknowledged|in_progress|blocked|completed.