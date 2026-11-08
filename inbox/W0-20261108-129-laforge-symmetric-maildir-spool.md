---
id: W0-20261108-129
from: starfleetctl
to: LaForge
task_ref: wormhole-manager
type: architecture-decision-and-implementation-direction
priority: high
---

# W0-129 — Adopt symmetric Maildir-style spool and ship routing

This is the owner's approved architecture direction. Treat it as superseding the earlier over-configurable Incoming/Outgoing/Paths/Messages/Decisions configuration design.

## 1. Fixed repository layout

Use a fixed, protocol-defined layout; do not make each path or lifecycle directory configurable:

```text
mail/
  <target-ship>/
    new/
    cur/
    processed/
    failed/
```

- `<target-ship>` is the canonical logical recipient ship identifier (e.g. `Enterprise`, `LaForge`, `Defiant`), not a host name or transport endpoint.
- `new/`: messages addressed to this ship and not yet claimed.
- `cur/`: messages durably claimed by the receiver but not yet completed.
- `processed/`: successfully processed messages.
- `failed/`: messages whose processing failed and require retry/review policy.
- No separate sender-side outbox or sent directory.

The same layout and lifecycle apply on both sides. There is no Starfleet-specific or remote-side-specific protocol branch.

## 2. Sending is a single spool put

To send a message to target ship X:
1. Create the complete immutable message file directly under `mail/X/new/`.
2. Stage only that exact new path.
3. Commit it.
4. Push it.
5. Return success only after the push is confirmed.

There is no sender-side rename/move after sending and no sender-maintained sent status. The message remains in the recipient's `new/` until that recipient's daemon claims it.

## 3. Receiver lifecycle

For a message addressed to the local ship:
1. Claim: atomically move `new/<id>` to `cur/<id>`, then commit and push.
2. Process idempotently through the existing Starfleet Comms ingestion/routing path.
3. On confirmed success: move `cur/<id>` to `processed/<id>`, then commit and push.
4. On a permanent processing failure: move `cur/<id>` to `failed/<id>`, then commit and push, retaining enough error metadata for diagnosis/retry.

A transient failure must not silently mark the message processed or lose it. Define and test retry/recovery behavior for messages left in `cur/` after crashes. Do not automatically delete messages.

## 4. Daemon and Git writer model

- One daemon instance per local Starfleet installation manages all configured Wormhole repositories and all target-ship directories within each repository.
- Exactly one coordinated Git writer per repository checkout at a time. Serialize fetch/rebase, spool puts, lifecycle moves, commits and pushes with a repository-scoped lock/mutex.
- Do not spawn one daemon per target ship.
- Each side runs its own daemon against its local checkout; both sides use the same protocol.
- The daemon routes messages based on target ship and existing Comms routing. Keep logical addressing separate from physical repository/transport selection.
- Preserve the existing checkout root requirement under `.starfleet-ai/var/wormholes`.

## 5. Durability, idempotency and concurrency requirements

- Every externally meaningful spool put and every lifecycle transition is a separate durable Git commit, pushed immediately.
- Stage exact paths only; never use broad `git add .` or equivalent.
- A transition is locally committed after commit creation and remotely published only after successful push. Do not report push success before confirmation.
- On push/network failure, retain the local commit and retry publication safely; do not recreate a second message or discard the commit.
- Handle non-fast-forward updates by safe fetch/rebase/reconciliation under the repository lock. Never force-push or overwrite remote state.
- Use immutable, collision-resistant message IDs and deterministic filenames. Duplicate delivery/processing must be idempotent.
- Git rename/move should be represented by one commit containing the exact source and destination paths. Crash recovery must handle every boundary (before commit, after commit/before push, after push/before next step).
- Avoid claiming that Git provides a distributed atomic transaction across both peers. The protocol is an idempotent sequence of locally committed and pushed transitions.

## 6. Configuration simplification

Reduce configuration to what cannot be inferred:
- repository identity/ID, remote URL, and branch (retain only fields genuinely required by the current multi-repository design);
- no user-configurable mail paths, lifecycle folder names, incoming/outgoing switches, glob patterns, nested message/decision paths, target alias maps, or per-folder commit-message settings;
- defaults for polling, timeouts and retry behavior remain internal implementation defaults unless an operational requirement demonstrates otherwise.

Do not add a new elaborate schema to replace the old one. Reconcile or remove the over-configurable schema work in a focused, reviewable change. Preserve unrelated useful config/error-propagation work.

## 7. Scope and coordination

- First, report current branch/HEAD/dirty state and identify which in-progress W0-101/W0-096 changes conflict with this decision. Do not discard uncommitted work.
- Pause further work on nested config validation that is now obsolete; do not expand it.
- Propose a small sequence of atomic implementation commits: fixed spool/layout and message format; serialized repository writer and durable put/transition operations; Comms integration and recovery; tests/docs.
- Keep W0-096 error propagation as a separate concern where still applicable; do not conflate it with the protocol redesign.
- Add tests for symmetric send/receive behavior, multiple target ships, exact-path staging, serialization of concurrent operations, duplicate IDs/delivery, crash recovery at each commit/push boundary, retry after push failure, and idempotent Comms ingestion.
- Update the implementation docs and provide a criterion-to-test/evidence table plus exact test/build command output.

## 8. Checkpoint

Acknowledge this decision and provide an initial checkpoint before making broad changes:
- branch, HEAD, merge-base, dirty status;
- current implementation state and any active uncommitted work;
- concrete commit plan and tests;
- any genuine ambiguity/blocker (do not reopen already-decided architecture choices).

This GitHub inbox file is a work instruction, not a Comms message or proof of delivery. Do not use `owner-coordination` as a sender/recipient or invent a Comms identity. Report Comms delivery only with a verifiable receipt.
