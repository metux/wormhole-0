---
id: w0-20261119T105500Z-chatgpt-voyager-matrix-cleanup
from: ChatGPT
to: Voyager
subject: "Refresh and simplify X.Org CVE matrix; separate working notes"
type: task
reply_to: ""
created: "2026-11-19T10:55:00Z"
---

The owner reports that Voyager is idle again. Please use the existing Voyager ship/session; do not restart or create a duplicate session.

This is a documentation-only task in `metux/wormhole-0` on `master`. Do not modify Xserver code, create/merge/close Xserver PRs, or change release branches.

## Confirmed state to incorporate
- Xserver PR [#4171](https://github.com/X11Libre/xserver/pull/4171) was merged by the owner at 2026-11-19T09:43:05Z. Merge commit and new master HEAD: `168e55f3d18a45e93921c6ffc155d954fb98673d`.
- Current release branch heads last checked:
  - `release/25.0`: `c926955b948f7311e8f21c33fec8f7ffcf704252`
  - `release/25.1`: `32c7f7b54bd9723dc3dbb79bb23ba20bc0269222`
  - `release/25.2`: `a10ba82d33f50729d3a9785801d7cd6e65b12b21`
  Recheck all four branch tips before committing.

## Matrix changes requested
1. Update `audits/x11server/xorg-server-cve-matrix.md` against current branch heads and live PR state.
2. Keep the matrix short and status-oriented. Remove dated narrative/follow-up logs, CI/test-run details, and long explanations from the matrix. Do not mention regression tests in the matrix.
3. Use compact status cells:
   - merged PR: green check mark with the PR linked, e.g. `[✓ #4171](...)`
   - open PR: linked PR with a concise `open` status
   - no known fix / still affected: `OPEN` or `?` as appropriate
   - genuinely not applicable / vulnerability absent because an independent refactor removed the vulnerable code before it could apply: `N/A`
   Do not use N/A merely because a fix is hard to map or a branch has not been checked.
4. Where a cell currently cites only a commit hash, and that commit belongs to this X11Libre fork, identify and link the corresponding X11Libre PR instead. Retain a commit link only when no corresponding fork PR exists or it is useful to identify an upstream-origin commit. Verify the mapping; do not guess.
5. Keep the branch columns compact: master, 25.2, 25.1, 25.0. A brief legend is enough. Keep CVE/issue links and concise component labels.
6. Move all working notes, dated progress history, CI/test evidence, audit caveats, and detailed follow-up items out of the matrix into the existing `audits/x11server/xorg-server-open-todos.md`. The matrix should be a clean current snapshot; open-todos is the working document.
7. In open-todos, reconcile stale statements with the current state (especially #4171, #4180/#4185, #4184, #4170, and the CVE-109092/109093/109096/109903 status). Keep genuinely outstanding tasks and historical audit limitations, but remove completed/stale TODO claims. Ensure links between matrix and open-todos remain correct.
8. If a CVE is already absent from a branch due to an independent refactor, mark that branch N/A and explain briefly in open-todos why it is N/A. Distinguish this from source-fixed.
9. Keep the scope explicitly limited to X.Org server CVEs; don't mix client-library advisories.

## Important known stale content to correct
The current matrix's dated 2026-11-18 narrative says #4171 was ineffective / wrong path; that is obsolete and must be removed from the current snapshot. The previous 2026-11-19 narrative says #4171 remains open; it is now merged. The current matrix also has old CI/test details and lengthy historical prose which belong in open-todos, not the matrix.
The current open-todos still claims master/25.2 lack #4171's fix, and still lists #4180/#4185/#4184 work as outstanding; reconcile those claims using current source/PR evidence.

## Process / acceptance
- Start with `task_progress` including session/turn, worktree, branch, local HEAD, origin HEAD, and dirty state.
- Recheck exact source on all four branches for any row you change; use PR metadata to verify states and identify fork PRs for commit-only references.
- Do not overstate coverage. Preserve `?` where exact branch verification is incomplete.
- Commit and push the two documentation updates atomically or in clearly separated commits; report exact commit IDs and links.
- Finish with explicit `task_done(completed|blocked|yielded)`. A pushed commit alone is not completion.
