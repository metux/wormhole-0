---
id: W0-20261106-065
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T23:00:00Z
in_reply_to: null
task_ref: W0-060,W0-063
workspace: mpbt-hq/starfleetctl
---

# W0-065 — Resume Wormhole manager and correct the console-ask follow-up

The isolated branch `work/wormhole-manager-w0-017` remains at `8f0d7b4385abedf2a93e5c187a710c2bad9fc43e`; no newer commit is visible. W0-060 remains active. Continue implementation on that branch/worktree in small commits; do not claim a PR unless a real PR exists.

## Mandatory correction to W0-063 — no console-to-background migration

The earlier W0-063 instruction to move Defiant into background mode is superseded by the owner's explicit operational decision:

- Do NOT restart, relaunch, detach, convert, replace, or duplicate any existing console-attached ship.
- In particular, do not restart Defiant as a watchdog or recovery strategy. Preserve the existing process, session, worktree, branch and state.
- The desired policy is to prohibit the interactive `ask` tool for Console ships exactly as for Background ships.
- Implement this through the generated per-ship OpenCode configuration, reusing the existing Background deny path. Do not invent config keys or tool IDs: inspect the current generator and pinned OpenCode configuration schema first.
- Policy must derive deterministically from authoritative ship mode. Add tests proving both Background and Console generated configs deny `ask`, and that ordinary non-protected tools retain their existing permissions. Never auto-approve protected actions.
- Do not claim the already-running Console process has received the new policy unless a supported live-reload mechanism is demonstrated. The policy applies on the next normal owner-authorized launch if reload is unavailable; that is not permission to relaunch the current ship.

First inspect the actual config-generation path, current Background ask-deny implementation, exact OpenCode config syntax, and existing tests. Report the source paths and pinned OpenCode version before changing it. Plugin interception is optional defense-in-depth, not a substitute for generated config.

## Continue W0-060 independently

Keep fixing the audited Wormhole manager blockers in W0-060:
1. CommsID reservation must be exactly-once and stable across retry/recovery.
2. Worker ingestion must use the real `comms.Bus` / `PostExternal()` path; remove parallel hand-written JSON serialization.
3. Implement actual remote repository fetch/pull, inbox scan, sequence recovery, outbox commit/push and conflict handling; `wormhole sync` must not be a success-printing no-op.
4. Implement a real cross-process worker lifecycle, not same-process PID bookkeeping.
5. Durable queue/checkpoint and at-least-once idempotent sink publication.
6. Strict nested YAML parsing/validation via the existing project config stack.
7. Crash/concurrency/fanout/corruption/permissions tests; non-ENOENT read failures must remain errors.
8. Correct the requirement matrix so it reports implemented/partial/not implemented truthfully.

Work only on the isolated branch/worktree. Make small committed increments; run focused tests and then `go test ./...`, `go test -race ./...`, and `go build ./...` where applicable. Report exact SHAs, parent SHAs, commands/results and remaining blockers. Do not wait for Enterprise review.
