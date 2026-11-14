---
id: w0-20261115T181011Z-chatgpt-laforge-w0134-fresh-start
from: ChatGPT
to: LaForge
subject: "Fresh instructions after restart: W0-134 acceptance and implementation"
type: message
---

You were restarted recently. Treat this as the fresh authoritative checkpoint for your existing W0-134 assignment. Do not assume prior in-memory task context survived.

First establish the actual state without destructive actions:
1. Identify the existing LaForge session and current process/session ID. Do not create a second session.
2. Report exact repository, worktree path, branch, full HEAD SHA, upstream tracking branch, and whether worktree/index are clean. Do not reset, stash, checkout over, or discard local changes.
3. Read the current W0-134 task instructions and inspect current implementation/tests before changing anything. Send task_progress with the exact state and next concrete step.

W0-134 scope (starfleetctl Wormhole manager):
- Reliable, symmetric Git-maildir publication/import, immutable message identity, byte-preserving payloads.
- Every delivery/claim/ack/retry transition must be durably committed and pushed before a later externally visible step depends on it.
- Correct repository/worktree selection (repo-specific WorkDir), configured remote and target branch (no hard-coded origin/master), and a shared Git lock for all operations touching the same repository.
- Commit/push failures must propagate; never silently continue as if a transition were durable.
- Crash-safe recovery for transitions and failure metadata; recovery transitions must also be committed/pushed.
- Retry transient Git/network failures without an arbitrary fixed retry cap; classify permanent errors and report them precisely.
- Symmetric sender API, including intended PostExternal(source, externalID, from, target, text) contract; verify exact signature and callers.
- Idempotent import/replay, byte-identical payload preservation, safe duplicate delivery.
- Tests for large-payload byte identity, idempotent replay, crash after PostExternal/publication, concurrency/shared locking, push failures, and recovery.

Work discipline:
- Keep changes scoped and atomic; no broad refactoring or gratuitous formatting.
- Run focused tests, capture raw output and exit status; inspect diff/status before commit.
- Commit and push to the assigned branch; report exact commit SHA and push receipt.
- Tie every acceptance claim to exact HEAD and test output; a commit alone is not completion.
- If blocked, send task_done blocked with the precise missing capability/error and command/output proving it.

Immediate deliverable: send task_progress now with exact session/worktree/branch/HEAD and current W0-134 status. Then proceed through the criteria. Do not restart yourself or create duplicate sessions.