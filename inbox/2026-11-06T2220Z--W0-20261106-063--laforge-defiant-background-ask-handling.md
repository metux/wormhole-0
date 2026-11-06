---
id: W0-20261106-063
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T22:20:00Z
in_reply_to: null
task_ref: W0-060
workspace: mpbt-hq/starfleetctl
---

# W0-063 — Diagnose and eliminate Defiant's console-bound interactive-ask stall mode

Owner reports a possible operational failure mode: Defiant currently runs attached to a local console. It may occasionally stop making progress because an interactive `ask`/approval prompt is waiting on console input. This is a follow-up engineering task for Laforge, separate from the Wormhole protocol implementation but related to fleet reliability.

## First: establish facts, do not assume

1. Coordinate with Enterprise's W0-062 Comms probes and inspect the actual Defiant runtime/launcher configuration and logs available in the local fleet workspace.
2. Determine whether Defiant is actually blocked on an interactive ask, what component emits it, whether stdin/TTY is required, and whether the process remains alive but idle.
3. Identify all other blocking input paths (approval, permission, tool confirmation, terminal prompt, auth prompt), not only the literal `ask` command.
4. Record reproduction steps, exact prompt text/type (redact secrets), process state, and relevant timestamps. Do not infer a hang from a missed heartbeat alone.

## Required design and implementation

5. Move Defiant execution to a properly supervised background mode that does not depend on an attached interactive console remaining available. Preserve current worktree, branch, environment, and session state.
6. Replace console-only questions with a durable, correlated Comms interaction: publish the question and required decision context, wait asynchronously for a response, and resume the same task/session when a response arrives.
7. Define safe timeout and restart behavior. A timeout must not silently approve, deny, discard, or skip a question. Persist pending questions and their correlation IDs across process restart.
8. Preserve human approval boundaries. Never auto-approve destructive operations, credential access, publication, force-push, merge, or other privileged actions just to keep the worker moving. Escalate such decisions through the configured owner route.
9. Add a watchdog/health state that distinguishes running, working, awaiting external input, restarting, and failed. Heartbeats must continue while awaiting input so Enterprise can distinguish a healthy wait from a deadlock.
10. Add tests for detached stdin/TTY, pending ask, delayed response, process restart while awaiting input, timeout, duplicate response, and protected-action approval. Verify no task loss or duplicate execution.
11. Keep the change isolated and reviewable. Do not change Wormhole protocol semantics or unrelated agent behavior. Document deployment/migration and rollback. Commit and report exact SHAs, tests, and remaining limitations.

Do not implement an unsafe blanket "non-interactive auto-approve" mode. If the root cause is outside Laforge's ownership, provide a precise diagnosis and the smallest actionable handoff rather than claiming completion. ChatGPT/project owner performs the technical audit.
