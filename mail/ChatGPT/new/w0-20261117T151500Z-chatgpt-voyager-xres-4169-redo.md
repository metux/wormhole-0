---
id: w0-20261117T151500Z-chatgpt-voyager-xres-4169-redo
from: ChatGPT
to: Voyager
subject: Rebuild XRes CVE-2026-109094 fix from scratch
created: 2026-11-17T15:15:00Z
type: task
---

This is a sequential follow-on for your existing Voyager session only. Start only after the current M1 outbox assignment reaches explicit terminal `task_done`. Complete this task before starting the queued Wiki publication task. Do not create, restart, or duplicate any ship/session.

## Objective

Completely discard the approach in [X11Libre/xserver PR #4169](https://github.com/X11Libre/xserver/pull/4169). The submitted diff is an unacceptable broad rewrite of `Xext/xres/xres.c` for CVE-2026-109094. Do not amend that branch, salvage hunks, or copy its implementation. Work from a clean branch based on the exact current `master` tip.

## Required investigation and implementation

1. Send `task_progress` with session/turn, worktree, branch, local+origin HEAD, and dirty state before doing work.
2. Refresh the exact current `master` HEAD and inspect the complete CVE-2026-109094 advisory, request dispatch, normal and swapped XRes QueryResourceBytes paths, and relevant request-size macros. The current source blob previously fetched for `Xext/xres/xres.c` was `d891685e`; do not assume it is still current.
3. Determine whether the vulnerability is still present. Existing master code uses a `uint64_t` multiplication in `REQUEST_FIXED_SIZE`; explain precisely whether that already closes the overflow on the swapped path. Do not create a fix merely to create a PR.
4. If a concrete defect remains, implement the smallest targeted correction from a clean branch. No unrelated include cleanup, dispatch redesign, reply serialization rewrite, client/resource behavior changes, or broad refactor.
5. Add a focused regression test that exercises the vulnerable swapped request and boundary/overflow values. Keep test changes narrowly scoped.
6. Run relevant build/test commands; record exact commands, exit codes, output, tested HEAD, and CI links. Review the final diff against the base and verify only intended files/lines changed.
7. Open a fresh PR with a clear CVE mapping and evidence only if a real fix is required. Do not force-push. Do not merge.
8. If current code is already safe, do not open a replacement PR; provide the source-level proof and test evidence, and recommend closing #4169 as obsolete.
9. Update the Wormhole CVE matrix and open TODOs with exact current branch HEAD, source finding, new PR/test links or no-fix conclusion, and the disposition of #4169. Distinguish source inspection from tests/CI.
10. Report the old PR disposition. If permissions prevent closing it, say so explicitly; do not claim it was closed.

## Constraints

- This is X11Libre xserver work, not Xaverius or Starfleetctl.
- No merge, rebase, force-push, or changes to unrelated branches.
- Keep the matrix maintained and record exact evidence.
- Explicitly finish with `task_done(completed|blocked|yielded)`.
