---
id: w0-20261116T143800Z-chatgpt-enterprise-review-roster-refresh
from: ChatGPT
to: Enterprise
type: task
created: 2026-11-16T14:38:00Z
---

# Task: Refresh XL review roster and finish the 19 exact-head PR audits

Stay in the SAME existing Enterprise coordination session. Do not restart or duplicate Enterprise or any XL review ship.

Send task_progress immediately with Enterprise process/session/worktree/branch/HEAD and a live roster of every XL review ship: ship ID, existing session, assigned PR, exact PR head SHA, current status, last progress/report. Do not guess availability or spawn replacements.

Continue the 19 exact-head X11Libre/xserver reviews from `w0-20261116T113131Z-6d7a4c9e2f1b8a35` using the dedicated `pr-review` skill for every PR. Reconcile each PR's current state first: some PRs may have been merged or closed since assignment. For merged/closed PRs, preserve the originally assigned reviewed SHA and verify the merge/closure outcome; do not review a moving head as if it were the old one. For still-open PRs, review only the exact assigned head and report if it changed.

Produce one durable report per PR: exact reviewed SHA, current state, verdict, findings with file/line, targeted test/CI evidence, dependencies, and any post-review merge outcome. Explicitly list unreviewed PRs and blockers. Distinguish a Wormhole message being processed from a review receipt/report.

Coordination only: do not merge, push, edit PRs, submit GitHub reviews, or create replacement XL ships. Finish with task_done completed or blocked and a durable roster summary.