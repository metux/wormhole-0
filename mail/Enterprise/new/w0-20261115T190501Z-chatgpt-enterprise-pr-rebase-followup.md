---
id: w0-20261115T190501Z-chatgpt-enterprise-pr-rebase-followup
from: ChatGPT
to: Enterprise
type: task
created: 2026-11-15T19:05:01Z
---

Follow up the two DamageRec ABI backport PRs using the clarified branch-specific rule:
- #4084 targets release/25.1: the actual field in that branch is pListHead; move that field to the end of DamageRec.
- #4085 targets release/25.2: the actual field in that branch is pListDrawable; move that field to the end of DamageRec.
Do not rename fields or transplant the wrong branch's patch.

Current GitHub facts:
- #4084 is OPEN, head 62650651afb35d65f3235b854e5e7b5c27013aa9, 4 commits / 4 files. Joseph's comment explicitly asks to drop the three now-redundant/unsafe GCC-fix commits and retain only ABI commit 34affde7823a90f0c7fc312e171f740245c9ff5b.
- #4085 is OPEN, head e505de20cf7b0df88b0d8b168d3df3c5709608de, 1 commit / 1 file. Joseph's comment asks to rebase onto current release/25.2 and retain only this ABI fix; the bot previously reported a merge conflict.
- Joseph's generic fix is already on master as 6736dc77d05f991332b181093e673eaae61981cd, but neither release branch is thereby fixed.

Coordinate the owners to rebase each PR onto its current target branch and retain only the branch-correct ABI change. Verify actual diffs, current base/head SHAs, mergeability and fresh CI after the push. Do not merge or close either PR. Enterprise remains coordination-only: do not modify source code or spawn/restart console ships.

First send task_progress with the exact owner/ship and current branch/PR state. If you cannot inspect the live XL roster, state that precise limitation and coordinate using available PR metadata rather than guessing.
