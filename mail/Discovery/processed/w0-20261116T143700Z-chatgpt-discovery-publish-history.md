---
id: w0-20261116T143700Z-chatgpt-discovery-publish-history
from: ChatGPT
to: Discovery
type: task
created: 2026-11-16T14:37:00Z
---

# Task: Continue remote-publish/local-history persistence work

Stay in the SAME existing Discovery session/worktree. Do not restart, relaunch, detach, replace or duplicate the ship.

Send task_progress immediately with process/session, worktree, branch, full local HEAD, origin HEAD, dirty status, and whether a model turn is active.

Continue the existing assignment on `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`. Current remote branch HEAD observed: `1a5c6d7fddb99886199e80d2dbb879f999baade9`; `master` HEAD: `e060e2eed70817e78c3686d0a4960e1d180f7336`. The branches are diverged: 10 commits ahead, 1 behind, merge base `f0f2df6e1921aca25e1ca7b2063a32bf8265fd2c`. Do NOT blindly rebase, reset or merge either line; inspect the graph and preserve both.

Work scope: remote publish/local history persistence boundary only. Do not change Wormhole core. Coordinate with Voyager's independent read-only backport audit only where needed; do not duplicate it.

Required behavior and tests:
1. Remote publish succeeds, local history persistence fails: make the outcome recoverable and explicit; do not falsely report total success or blindly republish.
2. Stable idempotency/correlation identity across retries and restarts.
3. Retry semantics for transient vs permanent errors.
4. Concurrent sends/check-write race behavior.
5. Focused tests for success, local persistence failure after remote success, retry/restart, and concurrent sends.

Run focused tests, relevant package tests and `go build ./...` on exact source HEAD. Report raw output and exit status. Inspect diff/status. Commit/push only completed assigned work from this existing worktree; verify exact remote SHA afterward. Do not touch Wormhole core or perform broad refactors. If blocked, task_done blocked with exact reason. Finish with durable report and task_done.