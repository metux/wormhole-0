---
id: w0-20261116T173500Z-chatgpt-voyager-xlibre-backport-repair
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T17:35:00Z
---

The user has explicitly authorized implementation. This supersedes the read-only restriction in the previous PR audit task. Continue in the SAME existing Voyager session/worktree; do not restart, detach, background, or duplicate the ship. Send task_progress immediately with session/process, worktree, current model-turn state, and whether any previous audit work remains active. If a model turn is active, do not start conflicting work; report the blocker.

## Scope: rebuild three broken backport PRs from clean target-branch bases

Rebuild these PRs independently, each as one clean squashed commit, then update the existing PR head branch using a safe force-with-lease equivalent. Do not change target branches or close/recreate PRs.

1. #4116 — https://github.com/X11Libre/xserver/pull/4116
   - Base: release/25.0
   - Current broken head: c263b96018fc49d2a1c01d336f52627ad28213ba
   - Rebuild only the CVE-2026-109093 XTestFakeInput valuator bounds fix. Start from the exact current release/25.0 base, inspect the original upstream fix (CVE advisory references commit 43b4064a) and adapt narrowly to this branch. Preserve all existing branch-specific semantics. Do not copy the broad unrelated include/API changes from the current PR diff.
   - Add a regression test if a suitable test harness exists; otherwise explain precisely why not.

2. #4147 — https://github.com/X11Libre/xserver/pull/4147
   - Base: release/25.1
   - Current broken head: 9cb37bdc8b6e882eee06596eeadc64ba1441802a
   - Rebuild the intended KdCreateScreenResources return-value fix from the exact release/25.1 base. The current diff contains a literal conflict marker and duplicate function declaration; do not salvage that patch. Inspect the original upstream commits 550b9ed0dd38546b973a0b567868119422d1f370 and 47d68e7c57ef7a341ff254d3f3fb4eb6f72c3ce3, then make the smallest correct adaptation.
   - Produce exactly one clean squashed commit on the existing PR branch and force-update that PR branch. Verify the pushed SHA and ensure no conflict markers or duplicate declarations remain.

3. #4157 — https://github.com/X11Libre/xserver/pull/4157
   - Base: release/25.2
   - Current broken head: 44e95c181768e5d008b974914ff51e95a7d753f7
   - Rebuild the intended KdCreateScreenResources fix from the exact release/25.2 base. The current diff removes hundreds of lines and changes unrelated kdrive behavior; do not salvage it. Inspect the original upstream change and adapt only the required function/struct contract for this branch.
   - One clean squashed commit on the existing PR branch; force-update only that PR branch. Verify the pushed SHA.

For all three:
- Re-fetch the current target branch tip immediately before preparing each patch; do not assume the recorded base SHA is still current.
- Preserve separate PRs/branches and do not mix fixes between release branches.
- No broad header migration, cleanup, unrelated refactor, or opportunistic fixes.
- Review the complete resulting diff against the target branch before pushing.
- Run the narrowest relevant tests and a clean build with -Dwerror=true where practical. Record exact commands, exit codes and raw relevant output.
- Verify exact pushed HEAD, parent, changed paths, commit count, and fresh exact-head CI/check status after each push.
- If a patch cannot be made safely or exact tests cannot run, stop that PR and report the precise blocker; do not force-push a speculative result.

## Separate CVE audit only — do not modify #4153

User asks whether release/25.2 and master are affected by CVE-2026-109094. I inspected current source:
- release/25.2 Xext/xres/xres.c and master Xext/xres/xres.c both have ProcXResQueryResourceBytes() using ((uint64_t)stuff->numSpecs) * sizeof(xXResResourceIdSpec) in REQUEST_FIXED_SIZE.
- Neither current source contains SProcXResQueryResourceBytes; the current dispatch path handles swapped clients inside Proc after the length check.
- Thus the vulnerable 32-bit multiplication in the old SProc path is absent on both current branches; the 64-bit cast prevents the described wrap. The enormous #4153 rewrite is not an acceptable fix and appears unnecessary for these current branch tips.
Independently verify those exact source paths/branch tips and the dispatch registration. Report whether you agree. Do not touch #4153 or invent a replacement patch unless the current exact branch source contradicts this finding; if safe, recommend closing #4153 as obsolete, but leave the close action to the user.

## Explicit hold

Do not edit or close #4125, #4148, or #4156 (the kdrive/ephyr fbGetScreenPixmap -> GetScreenPixmap changes). They are held pending Stefan's answer about backport-worthiness. Do not include them in this task.

## Reporting

Send task_progress at start and at meaningful checkpoints. When complete, commit a durable report in wormhole-0 and send task_done completed only after all pushed SHAs, diffs, test results and CI states are verified. If any part remains, task_done blocked/yielded with exact reason and list of finished vs unfinished PRs.