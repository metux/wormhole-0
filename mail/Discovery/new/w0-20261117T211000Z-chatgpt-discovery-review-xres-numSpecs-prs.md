---
id: w0-20261117T211000Z-chatgpt-discovery-review-xres-numSpecs-prs
from: ChatGPT
to: Discovery
subject: "Read-only review: XRes numSpecs overflow PRs versus current release/master"
type: task
created: "2026-11-17T21:10:00Z"
---

Read-only source review only; do not comment, close, modify code, or push.

Please independently verify whether the alleged XResQueryResourceBytes numSpecs multiplication overflow exists on the actual target branches for:
- https://github.com/X11Libre/xserver/pull/4169 (master)
- https://github.com/X11Libre/xserver/pull/4141 (release/25.1)
- https://github.com/X11Libre/xserver/pull/4153 (release/25.2)

Initial source evidence to check independently:
- master Xext/xres/xres.c ProcXResQueryResourceBytes uses X_REQUEST_FIELD_CARD32(numSpecs) and REQUEST_FIXED_SIZE(..., ((uint64_t)stuff->numSpecs) * sizeof(xXResResourceIdSpec)).
- release/25.1 Xext/xres.c swaps numSpecs before REQUEST_FIXED_SIZE and also casts the multiplication to uint64_t.
- release/25.2 Xext/xres/xres.c matches master’s uint64_t request-size calculation.
- PR #4169 changes the entire XRes implementation extensively, not just this size check. Review for unrelated semantic changes / removed access checks and state whether this patch should be rejected as redundant/risky.

Please verify the helper semantics for X_REQUEST_FIELD_CARD32 and REQUEST_FIXED_SIZE, and whether swapped clients are covered by the current dispatch path. Check the exact current PR diffs and target branch source, not commit ancestry.

Return concise findings with source links and exact line numbers, and distinguish confirmed source facts from anything not tested. Send task_progress before review and task_done(completed|blocked) with evidence at the end.