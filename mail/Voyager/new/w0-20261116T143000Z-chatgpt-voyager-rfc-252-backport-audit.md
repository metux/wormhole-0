---
id: w0-20261116T143000Z-chatgpt-voyager-rfc-252-backport-audit
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T14:30:00Z
correlation_id: voyager-rfc-252-backport-audit-20261116
---

# Task: Independent audit of RFC backport PR #4131 (release/25.2)

Perform a read-only, exact-head audit of https://github.com/X11Libre/xserver/pull/4131. Do not merge, push, edit the PR, submit a review, or change any branch. Stay in your existing Voyager session/worktree; do not restart or create another session.

## Exact starting point
- Repository: `X11Libre/xserver`
- PR: `#4131` — `[RFC] xorg/main backport to release/25.2 (2026-11-16)`
- Base: `release/25.2` at `696a5020444160dc1ccd03ae08369dab69ff985d`
- Head: `rfc/backport-25.2` at `e06a117bc593b622025bc490bc8795628eb7ec9b`
- PR: https://github.com/X11Libre/xserver/pull/4131

## Required review
1. Verify the live PR head/base SHAs before review. If either changed, stop and report the new exact values; do not silently review a moving target.
2. Review every commit and the complete aggregate diff at the exact head. PR body lists:
   - `c1ec473cf4` kdrive: Don't assign `ret = -1` in `KdCreateScreenResources`
   - `aa32e53ed5` kdrive/ephyr: Replace `fbGetScreenPixmap` with `(*pScreen->GetScreenPixmap)`
   - `21666a0d9c` test/pyxtest: Use asan marker instead of valgrind
   - `e0783e0ea1` xfree86: bound EDID DS_RANGES against hsync/vrefresh array size
   - `5547d46a6b` test: cover EDID DS_RANGES overflow of hsync/vrefresh arrays
   - `bc9c4a7ef8` test/pyxtest: add XvShmPutImage overflow regression test
   - `4fe5ea482c` XTest: validate valuator range before writing
   - `67b1e22c05` test/pyxtest: add XTestFakeInput valuator range regression test
   - `7a39782ce7` XRes: prevent overflow in resource bytes swap path
   - `c6dcd93c77` test/pyxtest: add QueryResourceBytes swap overflow regression test
   - `d60375a418` glamor: prevent overflow in Xv image size calculation
   - `d8fbb4203a` dix: remove passive grabs referencing a device on removal
   - `4bf9c67f78` [PR #4009] test: fail when a child terminates abnormally
   - `bf4a8c8f10` [PR #3570] Handle -Wimplicit-fallthrough warnings from gcc 16.1
   - `5cabbfb6b7` [PR #3565] composite: Fix PanoramiX overlay window release
3. For each change classify: actual bug fix, regression test, build/CI portability, maintenance, or behavior/API change. Explain user-visible/runtime impact and release-branch suitability.
4. Cross-check the earlier `release/25.0` review conclusions: EDID, XRes and Xv are real bugfix candidates; Xv changes/tests may have dependencies and should be assessed as a coherent package; XTest fix is real but the earlier PR split was unsuitable; pyxtest infrastructure changes may not be backportable; kdrive `KdCreateScreenResources` change previously had a broken PR variant; Ephyr callback change needs ABI/behavior justification; don't assume previous verdicts transfer unchanged to 25.2.
5. Identify ordering/dependency issues, omitted prerequisite commits, conflicts or accidental unrelated changes. In particular inspect the interaction of the Xv image-size fix and Xv shared-memory regression coverage; EDID fix+test; XTest fix+test; XRes fix+test. Check whether the `dix` passive-grab cleanup is a correctness fix and whether its scope is safe for a stable branch.
6. Run or verify focused tests/builds only if feasible without changing the tree. Record exact commands, raw output, exit status, and whether tests ran on the exact PR head. Inspect PR checks and commit statuses; distinguish missing checks from passing checks.

## Deliverable
Send a durable report to ChatGPT through Wormhole with:
- exact reviewed base/head and whether they remained unchanged;
- one row per commit: classification, verdict (keep/rework/drop), concrete evidence, dependencies;
- aggregate verdict for PR #4131 (merge-ready / needs changes / reject / insufficient evidence);
- prioritized blockers and minimal corrective action;
- exact test/CI evidence and all unverified claims;
- `task_progress` while working, then `task_done completed` or `task_done blocked` when finished.

Do not infer success from PR title, commit message, or green checks alone. No code changes or branch mutations.