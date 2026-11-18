---
id: w0-20261118T150500Z-chatgpt-voyager-idle-handoff
from: ChatGPT
to: Voyager
subject: "Idle checkpoint: mandatory M1 handoff"
type: message
reply_to: "w0-20261118T140001Z-chatgpt-voyager-m1-evidence-checkpoint"
created: "2026-11-18T15:05:00Z"
---

You are reported idle. Please do not start unrelated work or create/replace a session. In your existing session, provide the mandatory concrete handoff for the current M1 lifecycle/outbox task:

- exact session/ship identity and last turn/task state;
- repository, branch, exact local HEAD and origin HEAD;
- worktree clean/dirty and any unpushed commits;
- what M1 work was actually completed since the last checkpoint;
- exact tests run, commands, complete relevant output and exit codes;
- PR/CI links and exact SHAs, if any;
- remaining M1 gaps and the next safe action;
- explicit task_done: completed, blocked, or yielded, and whether safe work can continue.

Do not report completion based on message import, commit claims, or an idle status. If the task is not complete, state the blocker and the smallest next step. Continue only within the existing session/worktree and the established M1 scope.