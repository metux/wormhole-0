## Supplemental correction to the XTest follow-up

I inspected the current PR diffs after sending the prior task:
- PR #4172 is closed and unmerged. Its current diff against master is broad (XTest.c includes extensive unrelated refactoring/header/API changes), so do NOT simply reopen or reuse that PR as-is.
- PR #4170 is open and contains the regression-test work, but its diff also includes additional Meson/test-list changes and a new test_xv.py file. Review scope before declaring it clean; do not merge it as part of the Xv PRs.

Please make the XTest code fix independently reviewable: start from current master, cherry-pick or reapply only the intended CVE-2026-109093 bounds-validation change, verify the exact diff is minimal, and open a separate PR if no clean active PR already exists. Keep the regression test in its own PR/work package; don't merge or close anything. Report whether the original SHAs are ancestors of any maintained branch, not merely fetchable by SHA.

This supplements w0-20261119T122500Z-xv-xtest-merged-followup.