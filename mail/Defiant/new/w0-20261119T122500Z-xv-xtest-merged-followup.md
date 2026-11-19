# Defiant – continue Xv/CVE follow-up after Xv PR merges

The two Xv PRs are now merged:
- #4171 -> master, merge commit 168e55f3d18a45e93921c6ffc155d954fb98673d
- #4180 -> release/25.2, merge commit a10ba82d33f50729d3a9785801d7cd6e65b12b21

The failed job from run 44438712223 is understood: job 135275844852 (video-sunffb:25.0.0) failed during apt package installation because the configured Ubuntu 26.04 package sources could not locate a large set of required packages. It never reached compilation. No code change is indicated.

XTest/CVE-2026-109093 preservation check:
- The original commits 5a52e270c937f7e6eebcff6329db0a2a6ea3fd2c (code fix) and 28c8d69ec3cc1e475e52d71fa4a2e452c5151ec5 (regression test) are still retrievable by SHA.
- A separate code-fix branch exists: pr/master-xtest-validate-valuator-range-before-writing-_2026-11-17_16-52-06, tip dad0c3e529ff8486708779b550cc98da2fb77b3f; its PR #4172 is CLOSED and NOT merged.
- A separate regression-test branch exists: pr/master-test-pyxtest-add-xtestfakeinput-valuator-range-regression-test-_2026-11-17_16-50-59, tip a213bc3220377aa236b4cbfb85d556fab0fdfe10; its PR #4170 is OPEN.
- Thus the test is still in an open PR, but the code fix is only on a retained branch behind a closed, unmerged PR. Do not assume it is preserved in master.

Please continue using the existing Defiant session; do not restart or create a duplicate session.
1. Inspect #4172's close reason and current code branch/diff. Do not reopen it blindly if its closure was deliberate. If there is no active PR carrying the XTest code fix, create a separate PR from the existing safe branch (or a clean replacement branch if required), keeping it separate from Xv.
2. Verify #4170 still contains the intended regression test and report its current head and CI.
3. Verify whether the two original XTest commit SHAs are ancestors of any maintained branch; distinguish "fetchable by SHA" from actual branch reachability.
4. Publish a concise completion report in outbox with exact PR states, heads, ancestry evidence and CI. No merges, no PR closures, no changes to the already merged Xv fixes.
