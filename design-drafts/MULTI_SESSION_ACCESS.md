# Design draft: concurrent multi-session access to Wormhole

Status: **DRAFT — architecture requirement for future design; not implementation authorization.**

## 1. Goal

Wormhole should support multiple independent ChatGPT sessions using the same Wormhole installation concurrently. The owner may use separate sessions to plan, coordinate, and supervise different workstreams at the same time.

This must be a first-class capability, not an assumption that only one ChatGPT conversation is active at a time.

## 2. Identity and isolation

- Assign every participating ChatGPT conversation/session a stable, unique `session_id` within the Wormhole installation.
- Every command, message, event, decision, task mutation, and response records its originating `session_id`, actor, timestamp, and correlation/idempotency key.
- Keep session-local working context, pending questions, drafts, and transient state isolated by default.
- Do not use a process-global or repository-global "current session" variable to route work.
- Session identity must be established by a trusted integration boundary; never trust a caller-supplied session ID alone as authentication.
- Support session lifecycle states (active, idle, closed/expired) and explicit revocation. Do not silently reassign a session's pending work when it disappears.

## 3. Shared projects and resources

- Shared project/task state is explicitly scoped and addressable independently of a chat session.
- A session may read shared state according to its access scope, but writes must identify the target resource and expected version.
- Use optimistic concurrency (version/ETag/compare-and-swap) or an equivalent transactional mechanism for shared mutations. A stale write must be rejected or surfaced as a conflict, never silently overwrite newer state.
- Use append-only events/audit history for important decisions and state transitions so concurrent actions can be reconstructed.
- Distinguish session-owned drafts from committed shared decisions. A draft in one session must not become authoritative merely because it was written last.

## 4. Task ownership, coordination, and handoff

- A task may be session-owned, explicitly shared, or unassigned; represent these states distinctly.
- Define task claims/leases with owner session, expiry/renewal, and fencing/version token if exclusive execution is needed. Expired leases must not allow an old session to continue writing as owner.
- Permit explicit handoff between sessions, including a compact handoff record: objective, current state, decisions, open questions, artifacts/commits, and next action.
- Permit parallel work on independent tasks without a global lock. Lock only the smallest shared resource for the shortest necessary duration.
- When two sessions propose incompatible changes to the same shared decision/task, preserve both proposals and require explicit resolution; do not choose by last-writer-wins.

## 5. Messaging and request/response correlation

- Address messages to a session, a ship, a task, or a project scope as appropriate; avoid implicit broadcast.
- Every request/response pair has a unique correlation ID, and retries use an idempotency key so a retry cannot create duplicate tasks or duplicate side effects.
- Responses must return to the originating session unless explicitly delegated.
- A session must be able to inspect its own pending requests and shared task status without consuming another session's messages.
- Define explicit subscription/notification semantics for shared resources; do not rely on polling a shared mutable inbox without ownership/cursor rules.

## 6. Concurrency, durability, and recovery

- Persist state transitions atomically. A crash between recording a request and dispatching it must be recoverable without duplicate execution.
- Define transaction boundaries for task creation, assignment, message publication, and completion.
- Use per-resource sequence/version numbers and detect gaps or duplicate events.
- Ensure repository-backed synchronization and local workers can operate concurrently without clobbering each other's checkouts, cursors, or outboxes. Use isolated workspaces or another proven concurrency-safe design where Git worktrees/checkouts are involved.
- Test simultaneous writes, stale updates, retries, duplicate delivery, interrupted handoffs, lease expiry/fencing, and concurrent sessions acting on separate projects.
- Provide an audit view that can answer which session initiated, changed, approved, or completed each operation.

## 7. Security and authority

- Multi-session access does not imply that every session has every permission. Apply authorization by actor/session and resource scope.
- Protected actions still require the configured human authorization path; never treat another session's existence or activity as approval.
- Do not let one session read another session's private context unless explicitly shared.
- Session termination/revocation must invalidate its ability to issue new operations and, where appropriate, fence off stale in-flight writes.

## 8. Suggested rollout

1. Inventory current Wormhole state, message, task, worker, and repository synchronization models; identify all singleton/current-session assumptions.
2. Define session identity, resource scopes, ownership semantics, and wire/storage schemas before implementation.
3. Add concurrency-safe persistence and idempotency/version checks.
4. Add session-addressed messaging and explicit handoff.
5. Add isolated worker/worktree coordination and recovery.
6. Add multi-session integration tests, including race and crash-injection tests.
7. Enable concurrent access only after authorization, isolation, and conflict behavior are demonstrated.

## 9. Compatibility and non-goals

- Preserve single-session behavior as the degenerate case of one registered session.
- Do not solve concurrency by serializing all activity behind one global lock or one shared mutable conversation context.
- Do not equate ChatGPT session identity with a Starfleet ship identity; they are separate principals with explicit relationships.
- This document records a future design requirement only. It does not authorize implementation or change existing operational policy.
