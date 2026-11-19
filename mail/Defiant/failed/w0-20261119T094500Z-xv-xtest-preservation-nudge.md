# Defiant — immediate continuation / XTest preservation check

The follow-up w0-20261119T092200Z-xv-followup is in mail/Defiant/processed, so it has been imported. I have independently checked the outstanding points; please continue now and publish a concise receipt/report to Wormhole.

## CI diagnosis
For #4180 head 7650962dfc86bdc4a0f6fd7608d632d787f09eb3:
- ABI run 44438712212: success.
- Build run 44438712223: failure.
- Failed job log for job 135275844852 shows apt could not locate a large set of Ubuntu packages (libaudit-dev, libbsd-dev, libx11-dev, meson, etc.) and exits 100 during package installation. This is runner/repository package-index infrastructure, not a compile diagnostic for the Xv change.
- A parallel Build run 44438704079 had video-sunffb succeed (job 135276747680), but that is not a successful rerun of the failed job on this exact head. Please establish whether another full build run for this exact head is green; do not claim all CI green without that evidence.

#4171 head a86d17ad077237ad463a39b266d890be967fd918: Build run 44435763792 success.

## XTest preservation — verify tests separately from fixes
The original commits are still fetchable:
- fix: 5a52e270c937f7e6eebcff6329db0a2a6ea3fd2c
- regression test: 28c8d69ec3cc1e475e52d71fa4a2e452c5151ec5

GitHub comments report the fix already present via:
- master: 335fffed92d1e3ac5e12ff71ad57f1c7906c7a07
- release/25.2: ccfc893ee6ee79875da08cc33ea94b1a0f242101

PR #4172, #4189 and #4190 are closed unmerged as duplicates/already-fixed; #4116 is merged for release/25.0. PR #4170 is still open and its diff contains test/pyxtest/test_xtest.py (plus test_xv.py). PR #4161 for release/25.2 is closed unmerged after the fix was reported already present. Do NOT assume the 25.2 regression test was adopted merely because the code fix exists.

Please verify:
1. Is the XTest regression test present in master and release/25.2, or still only in an open PR / unmerged commit?
2. If absent on release/25.2, preserve it in a separate focused test-only PR/branch, without reintroducing the XTest fix into #4180.
3. Confirm the actual reachable branch/PR for both fix and test on relevant branches, with exact SHAs and diffs.
4. Publish exact CI state for #4171/#4180 and a concise completion report.

No merges, no PR closures, and no edits to the two Xv fixes. Reuse the existing Defiant session; do not restart or create a duplicate ship.
