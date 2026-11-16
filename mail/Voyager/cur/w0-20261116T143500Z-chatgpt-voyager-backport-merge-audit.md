---
id: w0-20261116T143500Z-chatgpt-voyager-backport-merge-audit
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T14:35:00Z
---

# Task: Close the backport safety audit after today's merges

Stay in the SAME existing Voyager session/worktree. Do not restart, relaunch, detach, replace or duplicate the ship. This is a fresh assignment because the previous 14:30 message had invalid frontmatter (`correlation_id`) and landed in `failed/`; this message is valid.

Immediately send task_progress with live process/session, worktree, branch, full HEAD and dirty status. Then perform a READ-ONLY audit of the current X11Libre/xserver backport state.

## A. Verify today's merged release/25.0 PRs
These were merged on 2026-11-16:
- #4120 EDID DS_RANGES bounds fix — merge `dfa03a4a45ffd365f0c27dccdc3492e0dc287799`
- #4111 Glamor Xv image-size overflow fix — merge `d53f913acd2ed9dd06f92069300eecafa21ed44b`
- #4127 Xv QueryImageAttributes negative-size fix — merge `0624563ae9ad5f894f45cf4f4c1f2f6efbb32c64`
- #4126 Xv PutImage negative-size fix — merge `8e84a99f0f2a8aae70313e36bb9bf62ca24a20a8`
- #4118 Xv shared-memory bounds fix — merge `f4e33bc8d24fc7f82a687a4e351f8fc4ab390ce5`
- #4114 XRes client-ID request size fix — merge `98421623e3d0c7976b7ccf3a9a5afbdd41334524`
- #4113 XRes resource-bytes swap fix — merge `ac8db972868f2b92c3a60eb71b734d43f5d5901a`

Verify each merge commit is reachable from `release/25.0`, inspect the actual merged diff, and verify the checks were run on the exact PR head. Confirm no duplicate/overlapping fixes and no missing prerequisites.

## B. Regression-test gap audit
- #4119 EDID test remains OPEN and test-only; automated review says it should accompany #4120 rather than be a standalone release backport.
- #4117 XvShmPutImage test was closed without merge.
- #4115 XTestFakeInput test was closed without merge.
- #4112 XRes QueryResourceBytes test was closed without merge.
- #4116 XTest production fix remains open.
Assess whether merged fixes now lack regression coverage in the release branch. Recommend minimal safe follow-up: tests integrated with fix PRs or a properly scoped test-only PR only where justified. Do not change any PR or branch.

## C. Audit RFC #4131 for release/25.2
Exact PR #4131: https://github.com/X11Libre/xserver/pull/4131
Base `release/25.2` = `696a5020444160dc1ccd03ae08369dab69ff985d`
Head `rfc/backport-25.2` = `e06a117bc593b622025bc490bc8795628eb7ec9b`
It is 15 commits ahead, 0 behind. Its exact-head CI is failing: ABI check and Build X servers. Build logs show compile failures including missing `registry.h` in XRes and missing `dixevents.h` in XTest on Cygwin, plus XRes/XTest object failures on Ubuntu. Determine root causes, distinguish source incompatibilities/missing prerequisites from environmental failures, and assess the whole 15-commit diff and commit dependencies. Do not assume 25.0 merge results prove 25.2 is safe.

## D. Deliverable
Durable report to ChatGPT with exact SHAs, commit reachability, exact-head CI evidence, regression-test coverage gaps, per-PR verdict and prioritized next steps. Distinguish confirmed from theoretical risks. Include task_progress during work and task_done completed or blocked. Read-only: no edits, pushes, merges or review submissions.