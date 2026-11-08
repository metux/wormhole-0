---
id: W0-20261108-134
from: owner
to: LaForge
created: 2026-11-08
priority: urgent
workspace: mpbt-hq/starfleetctl
branch: work/wormhole-manager-w0-017
supersedes: all earlier LaForge-directed Wormhole implementation instructions in this inbox
---

# W0-134 — One authoritative task: finish the Wormhole Maildir transport

This is the **only current implementation instruction** for your Wormhole work. It replaces all earlier LaForge-directed task, correction, review, nudge, checkpoint, and resume messages in this inbox. Do not try to reconcile those old instructions; they have been removed. The owner-approved architecture decision remains binding:
`decisions/2026-11-08--wormhole-is-comms-transport.md`.

## Goal

Finish a small, correct, recoverable Wormhole transport implementation on the existing branch. Wormhole is a transport backend for Comms, not a second messaging system. Do the implementation, not just an acknowledgement, status report, or plan.

## Preserve the existing work

- Continue on `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`.
- The last owner-observed tip was `421652ef8f2d037d1766bb110ba2bca0ef9a8565`; fetch the branch and inspect its actual current HEAD, parent, merge-base, and working-tree state before editing.
- Preserve all existing commits and useful code. Do not reset, force-push, rewrite history, replace the branch, discard changes, or perform broad formatting.
- If the branch advanced, work from its actual tip. If the working tree is dirty, inspect and preserve those changes; do not overwrite them.
- Keep commits small, atomic, and limited to the Wormhole task.

## Required behavior

### 1. Fixed symmetric Maildir spool

Use this exact repository-relative layout; do not add configurable directory names or a separate sender outbox:

```text
mail/
  <target-ship>/
    new/
    cur/
    processed/
    failed/
```

Both peers use the same layout. A sender publishes by writing one immutable message file to `mail/<target-ship>/new/<id>`, staging that exact path, committing, and pushing. No sender-side post-send move is needed.

### 2. Durable receiver lifecycle

For each mailbox belonging to the local ship:

1. Discover messages in `new/` and atomically claim each one into `cur/` **before** importing it.
2. Commit and push the claim transition. Do not start import until the claim is durably published.
3. On startup and every processing cycle, resume messages already in `cur/`; never strand them merely because a prior process stopped.
4. Import through the existing Comms Core API only:
   `Bus.PostExternal(source, externalID, from, target, text)`.
5. Use a stable identity for the Wormhole source/repository and the immutable message ID as `externalID`. Every retry must reuse the same identity and exact original payload bytes. Do not normalize, truncate, or reconstruct the payload.
6. Only after `PostExternal` confirms successful local Comms publication, move `cur/<id>` to `processed/<id>`, then commit and push that transition.
7. A transient import, Git, or network failure must leave the message recoverable in `cur/` (or retain its unpushed local commit for safe publication retry). Do not convert transient errors into permanent failure after an arbitrary retry count.
8. A permanently malformed or rejected message may move to `failed/` only with a durable diagnostic that explains the reason. Never silently delete a message.

A crash after Comms import but before the processed transition must be safe: retry the same `source`, `externalID`, and exact payload so Comms idempotency prevents duplicate publication.

### 3. Git durability and concurrency

- Every message put and every externally meaningful lifecycle transition is its own Git commit and is pushed immediately.
- Stage exact paths only. Never use `git add .`, broad directory staging, or equivalent.
- Serialize fetch/rebase, puts, claims, transitions, commits, and pushes with one repository-scoped writer lock. No concurrent Git writers for the same checkout.
- On push failure, preserve the local commit and retry publication safely. On non-fast-forward, fetch and reconcile/rebase under the lock; never force-push or overwrite remote work.
- Treat commit and push as separate states. Report a transition as remotely published only after push confirmation.
- Make file moves atomic locally and recoverable across crashes before commit, after commit/before push, and after push.

### 4. Comms boundary

- `Bus.PostExternal` is the sole Wormhole-to-Comms import boundary and Comms Core remains the sole writer of local Comms message records.
- Remove or refactor any reachable Wormhole direct writer such as `RouteMessage()` that writes under `.starfleet-ai/var/comms/msgs/`. Do not leave an alternate path that can bypass Comms Core.
- A successful `PostExternal` means **Imported** only. It does not mean an agent session received, processed, or completed the request.
- Do not implement an agent-side Wormhole poller, a second local inbox, or the separate OpenCode session-plugin follow-up in this task.

### 5. Configuration

Keep the simplified, fixed configuration model. Only retain repository identity/ID, remote, branch, and genuinely necessary operational settings. Reject removed legacy keys and nested path/alias configuration. Do not revive configurable incoming/outgoing folders, message/decision paths, or per-folder commit settings. Preserve useful existing error propagation and unrelated configuration work.

## Implementation order — do not stop after a plan

1. Inspect the live branch and preserve its state.
2. Implement and test the fixed mailbox discovery plus atomic `new -> cur` claim and durable Git transition.
3. Implement restart recovery from `cur/` and the `PostExternal` import using stable identity and byte-identical payload.
4. Implement durable `cur -> processed` after confirmed import, transient retry behavior, and permanent-failure diagnostics.
5. Serialize all repository Git operations and make push/rebase failure recovery safe.
6. Remove the direct Comms writer and close strict-config validation gaps.
7. Add/adjust focused tests for the actual behavior, including:
   - symmetric send/receive and multiple target mailboxes;
   - exact-path staging and repository writer serialization;
   - atomic claim and restart recovery from `cur/`;
   - successful import then processed transition;
   - transient import/push failure remains recoverable;
   - crash after import before processed move deduplicates on retry;
   - stable source/externalID and byte-for-byte payload preservation;
   - permanent malformed input reaches failed with a durable diagnostic;
   - concurrent workers cannot process one item twice;
   - no direct Comms-record writer;
   - removed legacy YAML keys are rejected.
8. Commit and push each coherent implementation slice. Continue to the next slice without waiting for another prompt.

## Verification and reporting

At the final acceptance checkpoint, run these exact commands on the exact reported HEAD and include their unabridged output:

```sh
go test ./internal/wormhole/ -count=1 -timeout 10s
go test ./internal/comms/ -count=1
go build ./...
```

Also report any additional focused test commands and their output. Do not claim completion based only on compilation, old tests, a clean working tree, or a GitHub inbox entry.

After each substantive code commit, publish a concise checkpoint with:
- exact branch, HEAD, parent, and changed paths;
- lifecycle behavior now implemented;
- tests actually run and their complete results;
- remaining concrete work and the next code change;
- any blocker with the exact command/action and relevant output.

Acknowledging this instruction is not progress. A plan is not implementation. If blocked, state the precise blocker and keep the task open; otherwise continue coding and pushing reviewable commits.
