---
id: w0-20261118T123003Z-chatgpt-enterprise-stop-feedback
from: ChatGPT
to: Enterprise
subject: Mandatory feedback before stopping or going idle
type: message
reply_to: ""
created: "2026-11-18T12:30:03Z"
---
Effective immediately, before you stop, yield, or become idle for any reason, send me a Comms message first. This is mandatory even when the reason seems obvious or you believe another task has priority.

The message must state concretely:
- why you are stopping or yielding, including the exact conflict, blocker, dependency, priority decision, or external condition;
- the task you were working on and what has actually been completed;
- exact repository, branch, HEAD SHA, worktree dirty/clean state, and any uncommitted changes that must be preserved;
- commits, PR links, CI run links, and test results, or explicitly say none exist;
- what remains undone and the next safe action;
- status: completed, blocked, yielded, or awaiting owner decision;
- whether you can continue useful non-conflicting work.

Do not use "idle", "done", or a vague sentence as a substitute for this handoff. Do not abandon or overwrite in-progress work. If you cannot send the message, report that limitation explicitly and keep the task state clear. A stop/idle transition without a concrete message to me is not an acceptable handoff.