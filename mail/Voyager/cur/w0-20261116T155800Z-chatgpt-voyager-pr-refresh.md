---
id: w0-20261116T155800Z-chatgpt-voyager-pr-refresh
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T15:58:00Z
---

Continue in the SAME existing Voyager session/worktree. Do not restart, detach, replace or duplicate the ship. Send task_progress immediately with session/process, worktree, current branch/HEAD, last completed work, and whether a model turn is active.

The previous PR audit is stale: the user has edited several open PRs. Refresh the independent read-only audit against the live GitHub state.

New exact heads observed:
- #4116 release/25.0 XTest: c263b96018fc49d2a1c01d336f52627ad28213ba
- #4119 release/25.0 EDID regression test: 7dc2d406763f4c6b4876cc2af5b8c1b5ccb137b1
- #4123 release/25.0 kdrive BitsPerRGB: 98705311b9d0b07fe0baca225095bdf046959cdd
- #4125 release/25.0 Ephyr GetScreenPixmap: b600a4416a5667a7f3b4eec737dd021e509f5c73
- #4148 release/25.1 Ephyr GetScreenPixmap: 363b6dfe7570a64fc58fc6ba317c2a3ada45c052
- #4154 release/25.2 EDID regression test: 097762fce3ac13f4dd0474f9556ae1957053d7cd
- #4156 release/25.2 Ephyr GetScreenPixmap: bd77acfcbf8d9759546cba522757b2decebed328

At the last exact-head Actions query:
- #4123, #4125 and #4148: ABI/build jobs completed successfully.
- #4116, #4119, #4154 and #4156: jobs were queued/pending; recheck current exact-head runs.
- #4116 still appeared to contain the broad upstream header churn including dixevents.h, which may not exist on release/25.0. Verify whether the latest patch actually fixed this; do not rely on old automated review text.
- #4119/#4154 contain a new EDID fixture test; review whether the fixture is valid and exercises the actual extension parsing/overflow path.
- #4123 changed to a small 3-addition/1-deletion diff; confirm semantics and tests.
- #4125/#4148/#4156 are one-line GetScreenPixmap hook substitutions. The user explicitly asked whether these are genuine bugfixes or merely API migration/backports. Determine from upstream origin, target-branch API semantics, and code behavior; distinguish functional bugfix from cleanup/compatibility-only change.

No additional merges were found after #4152 at 15:42:28Z in the prior query; refresh this and record exact merge commits if there are new merges.

Also reconcile all remaining candidate PRs and RFC queues:
#4124, #4128, #4130, #4141, #4142, #4146, #4147, #4151, #4153, #4157, #4062, #4129, #4131, plus relevant older open backports (#4019, #4081, #4087-#4092, #3992, #4009, etc.) where they affect the current queue. Do not treat unrelated repository PRs as part of the user's backport set.

Produce an updated durable per-PR report: merged/superseded, merge-ready, merge-ready only with test companion, needs rework, or reject. Include exact current head, target branch, diff/behavior verdict, exact-head CI status, test gaps, dependencies/duplicates, and concrete next actions. Keep PRs distinct per target branch. Do not merge, edit, push, or submit GitHub reviews. Send task_progress at milestones; task_done only after the report is committed and remotely verified, otherwise task_done blocked with exact reason.