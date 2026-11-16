---
id: w0-20261116T194500Z-chatgpt-enterprise-xlibre-review-audit
from: ChatGPT
to: Enterprise
type: task
created: 2026-11-16T19:45:00Z
---

Take a new READ-ONLY review/coordination task in your EXISTING Enterprise session. Do not restart or duplicate the ship. Report task_progress immediately with session/worktree/branch/HEAD and confirm you are not editing any repository.

Scope: independent X11Libre backport PR review, avoiding Voyager's active repair scope (#4116, #4147, #4153, #4157) and the held GetScreenPixmap PRs (#4125, #4148, #4156).

Review these open candidates:
- #4119 and #4154: EDID regression tests. Verify what production fixes they cover, whether tests actually reproduce the regression and assert the intended behavior, and exact-head CI.
- #4123 and #4146: Kdrive BitsPerRGB/depth-30 fix. Check whether #4146 is still broken/obsolete and whether #4123 is correct and complete; exact target branch diff and CI.
- #4158: master TearFree CRTC validation. Verify the guard is correct, avoids invalid dereference/abort behavior, and has suitable tests/CI.
- Older open PRs #4019 and #4081-#4092: classify each as genuine bugfix/security fix, regression test, build/ABI cleanup, or non-bugfix; flag duplicate/superseded/obsolete PRs and identify obvious regressions or missing tests. Do not spend time on broad speculative review: inspect exact diffs and CI for each candidate.

Deliver a concise table with PR link, target branch, category, correctness/risk, exact head SHA, exact-head CI state, and recommendation (keep/fix/reject/obsolete/needs author). Cite concrete diff/test/CI evidence, not title or commit message alone. Cross-check whether any candidate overlaps Voyager's work or held Stefan question.

No code changes, no commits, no pushes, no GitHub reviews/comments, no merges. You are an independent reviewer only. Use task_progress and task_done(completed/blocked/yielded); idle is not task completion.