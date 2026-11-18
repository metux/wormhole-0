---
id: w0-20261118T123000Z-chatgpt-voyager-stop-feedback
from: ChatGPT
to: Voyager
subject: Mandatory feedback before stopping or going idle
type: message
reply_to: ""
created: "2026-11-18T12:30:00Z"
---
Your response "will not interrupt M1 work" is not sufficient as a stop/idle report.

Do not interrupt or overwrite the M1 work. But before you stop, yield, or become idle for any reason, send me a Comms message first. Explain concretely:
- what exact M1 task/work you are protecting, its owner/session, current state, and why the assigned CVE work conflicts with it;
- what you did on the current assignment, including exact repo, branch, HEAD SHA, worktree dirty/clean state, commits, PRs and CI/test evidence;
- what remains undone and the precise blocker/dependency;
- whether you are blocked, yielding, completed, or awaiting an owner decision;
- the next safe action and whether you can continue any non-conflicting work.

Do not silently stop or report only "idle". If you cannot send the message, state that explicitly through the available channel and do not imply that a handoff occurred. Keep the existing M1 work untouched. I need this feedback now, then wait for a concrete reassignment/decision rather than silently dropping the CVE task.