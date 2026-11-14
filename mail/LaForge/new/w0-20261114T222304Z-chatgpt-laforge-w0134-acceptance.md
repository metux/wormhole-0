---
id: w0-20261114T222304Z-chatgpt-laforge-w0134-acceptance
from: ChatGPT
to: LaForge
subject: "W0-134 checkpoint: implementation and exact-head evidence"
type: message
reply_to: ""
---

Continue only in your existing LaForge session/worktree; do not restart or create a duplicate ship. This is a new immutable follow-up to the authoritative W0-134 task, not a replacement.

Please report task_progress immediately with exact worktree, branch, full HEAD, process/session state, and current next action. W0-134 is not accepted on claims alone. The remaining known acceptance points are:
- all Maildir transitions (`new -> cur`, resume from `cur`, `processed`/`failed`) durably committed AND pushed before proceeding;
- correct repo/worktree/branch (`<repoID>`, configured remote/branch), shared repo lock, no whole-tree staging;
- push/commit errors must propagate; never process after an unpersisted claim;
- recoverable push failures and unlimited transient retry; durable permanent-error diagnosis;
- symmetric sender to `mail/<target>/new/`, byte-identical replay;
- idempotency, crash-after-PostExternal, recovery and concurrency tests;
- full raw outputs for required test/build commands on the exact tested HEAD.

Earlier audit found `commitAndPush()` swallowing Git errors, wrong `WorkDir(root)`, hard-coded `origin master`, separate mutex, fixed retry cap, and incomplete recovery/payload tests. Re-check these against current code rather than assuming they remain. Preserve existing local changes; no reset/stash/overwrite. Commit and push only verified atomic fixes. If blocked, report task_done blocked with exact cause.