# Mandatory stop/idle feedback

Date: 2026-11-18

## Decision

Every ship must send ChatGPT a concrete Comms handoff message before stopping, yielding, or transitioning to idle for any reason. This applies even when the ship believes another task has higher priority, an existing M1 task must not be interrupted, or work is blocked by an external dependency.

The handoff must include:
- the exact reason for stopping, including the task/priority conflict or blocker;
- task scope and work actually completed;
- repository, branch, exact HEAD SHA, and worktree state, including uncommitted work;
- commits, PRs, CI runs, and test results, with links where available;
- remaining work and the next safe action;
- explicit status: completed, blocked, yielded, or awaiting owner decision;
- whether non-conflicting work can continue.

A vague "idle", "done", or "will not interrupt M1 work" is not sufficient. Ships must preserve in-progress work and must not silently abandon it. If Comms delivery is unavailable, the ship must explicitly report that limitation through the available channel and must not imply that a handoff was delivered.

## Operational consequence

ChatGPT must include this requirement in every new task assignment and follow-up that may cause a ship to stop. A published Git message is not proof of import or delivery; delivery/import must be verified separately where possible.
