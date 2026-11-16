---
id: w0-20261116T144800Z-chatgpt-voyager-backport-branch-audit
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T14:48:00Z
---

# Task: independent exhaustive audit of new release-branch backport PRs

Read-only review. Do not edit, push, merge, close, or submit reviews. Review every currently open X11Libre/xserver PR listed below at its exact current head, against its actual target branch. Produce a durable report and send task_progress while working, then task_done completed/blocked/yielded.

## Scope

New branch-specific backport PRs:
- release/25.1: #4140–#4151
- release/25.2: #4152–#4157

Links:
https://github.com/X11Libre/xserver/pull/4140
https://github.com/X11Libre/xserver/pull/4141
https://github.com/X11Libre/xserver/pull/4142
https://github.com/X11Libre/xserver/pull/4143
https://github.com/X11Libre/xserver/pull/4144
https://github.com/X11Libre/xserver/pull/4145
https://github.com/X11Libre/xserver/pull/4146
https://github.com/X11Libre/xserver/pull/4147
https://github.com/X11Libre/xserver/pull/4148
https://github.com/X11Libre/xserver/pull/4149
https://github.com/X11Libre/xserver/pull/4150
https://github.com/X11Libre/xserver/pull/4151
https://github.com/X11Libre/xserver/pull/4152
https://github.com/X11Libre/xserver/pull/4153
https://github.com/X11Libre/xserver/pull/4154
https://github.com/X11Libre/xserver/pull/4155
https://github.com/X11Libre/xserver/pull/4156
https://github.com/X11Libre/xserver/pull/4157

Also check related RFC backport PRs #4129, #4130, #4131 for overlap/dependency/consistency, but do not repeat the entire prior RFC audit unless needed to determine whether these individual PRs can merge independently.

## Required checks per PR

1. Capture exact PR head SHA, base branch and base SHA, mergeability, changed files, commit list, current state.
2. Read the complete diff, not just title/description. Detect accidental unrelated changes, malformed hunks, conflict markers, truncated/incorrect cherry-picks, wrong-base context, API/ABI changes, behavior changes and missing dependencies.
3. Compare the source fix and target branch implementation. Determine whether the fix is still needed, already present, superseded, or duplicated by another PR / RFC.
4. Check exact-head CI/workflow runs and relevant job logs. Report which jobs ran, conclusion, and root cause for failures. Never infer green CI from mergeable=true.
5. Assess tests: whether production change has a focused regression test on the same target branch, whether it actually exercises the bug, and whether tests pass. Identify safe pairing requirements.
6. Consider regression/security impact and release suitability. Do not recommend merge if diff is contaminated or unproven.

## Specific red flags already visible in an initial diff scan (verify independently)

- #4141 and #4153 XRes diffs are very large rewrites (~339/403 additions) rather than narrow overflow fixes; check for lost access-control callbacks, request byte swapping, protocol semantics, conditional build guards, and resource accounting regressions.
- #4142 XTest is a large rewrite (~108 additions/63 deletions), not merely the narrow valuator bounds fix; inspect all changed semantics and test coverage.
- #4146 has visibly malformed/duplicated KdAddScreen context and suspicious duplicate AddScreen lines.
- #4147 includes a literal conflict marker and duplicate function declaration.
- #4151 has malformed struct member declarations ending in dots instead of semicolons.
- #4157 is a very large kdrive rewrite (~113 additions/285 deletions) under the title of a narrow return-value fix; assess as likely contaminated until proven otherwise.
- #4140/#4152 glamor size overflow, #4143 Xv SHM bounds, #4144/#4155 EDID bounds, #4145 manpage, #4148/#4156 Ephyr pixmap accessor, #4149/#4150 Xv negative-size checks, #4154 EDID test: verify independently, including exact-base context and tests.

## Deliverable

Write a concise but evidence-rich report in an appropriate durable location (prefer a single report under the existing Voyager outbox/report convention). Include one row per PR with exact SHA, target, verdict (merge-ready / needs changes / reject or superseded), reasons, CI, test coverage and links. Separate:
- independently merge-ready now;
- merge-ready only together with a companion test/fix;
- needs correction/rework;
- do not merge / superseded.
State any uncertainty explicitly. Include exact commands/log excerpts or links and raw test exit statuses where available. Send task_progress at start and at meaningful milestones. Only task_done completed once the report is committed/pushed and remotely verified; otherwise blocked with reason.
