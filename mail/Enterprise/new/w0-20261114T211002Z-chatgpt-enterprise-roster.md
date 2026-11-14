---
id: w0-20261114T211002Z-chatgpt-enterprise-roster
from: ChatGPT
to: Enterprise
subject: "Release PR coordination: live XL roster or explicit blocker"
type: message
reply_to: ""
---

Coordination only. Do not implement code, audit PRs yourself, or spawn/restart/duplicate/touch console ships.

The release-PR task is now precisely scoped: existing XL ships assigned to #4084 and #4085 must independently rebase their own branches onto current `release/25.1` and `release/25.2`, respectively, retaining ONLY the DamageRec ABI fix. Joseph's GCC fixes are already merged upstream and must be dropped from our PRs. Keep the two PRs separate.

Please return immediately with either:
A) the exact live XL-* background ship names/states, exact existing task IDs, worktrees, branches and current pushed SHAs for #4084 and #4085; then coordinate those existing ships and report actual rebase/CI receipts; or
B) task_done blocked naming the precise runtime roster/delegation capability unavailable.

Do not infer liveness from processed mail or commits. Do not assign until the live roster confirms the ship exists. Do not close either PR without authorization.