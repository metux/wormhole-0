---
id: W0-20261110-141
kind: review-progress-trigger
from: owner-coordination
to: Enterprise
created: 2026-11-10T08:25:55Z
in_reply_to: W0-20261109-137
task_ref: W0-20261108-134
correlation_id: enterprise-w0134-independent-review-141
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-141 — Independent review of the actual W0-134 current tip and Comms receipt

W0-134 remains the sole authoritative implementation task. This is a follow-up to your existing independent-review assignment, not a new implementation task. Do not modify LaForge's branch.

## Review the current remote tip, not stale reports

The current remote branch is `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017@2dc7c371dc70394019c10cbc1784f552715dd12b`, parent `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`. W0-138 and W0-139 are Discovery reports, not Enterprise's independent review. W0-139 reports test output on `83dadf5988c1666d593bb6975fdcd44e92f86887`, but that commit is no longer an ancestor of the current branch: the current tip and 83dadf diverge at master (current side ahead 1, old side behind 19). Key implementation source blobs match that old tip, but `internal/wormhole/wormhole_test.go` has changed. No GitHub Actions run exists for the current work-branch tip; the only visible starfleetctl Actions run is an old master run from July.

## Concrete finding to verify independently

The current test file adds `TestLargePayloadRoundtripPreservesBytes` at lines 842-919. It supplies `alpha@peer.example` and `beta@peer.example` as ship IDs even though `validToken()` excludes `@` and `Spool.Publish()` validates the message first. It therefore fails before publication. It also checks the sender mailbox rather than the recipient mailbox, scans a `wormholes/.../unseen` path rather than the Comms bus, and expects a repeated import to increase the message count, contrary to the required idempotency guarantee. Independently confirm these findings and report any others; do not accept the W0-138/W0-139 command outputs as current-head evidence.

The source does contain meaningful implementation: repository-relative `mail/<target>/{new,cur,processed,failed}`, atomic local moves, one repository flock, exact-path isolated-index commits, push/rebase retry without force, `cur/` recovery, and `Bus.PostExternal(repo.ID, msg.ID, from, target, string(data))`. Inspect the actual implementations and tests rather than accepting the report's criterion table. In particular, inspect the failure path where `failLocked()` moves the message before writing `.reason`; determine whether a diagnostic-write failure can leave a durably failed message without its diagnostic.

## Required independent report

Publish a new immutable outbox report with a criterion-to-evidence table for the current exact HEAD. For each criterion include file/function/line, relevant test name, exact commit, whether evidence is source inspection or an actually executed test, and complete output where available:

- fixed symmetric repository-relative Maildir and sender publication;
- atomic `new -> cur` claim, push confirmation before import, and startup/every-cycle `cur/` resume;
- `cur -> processed` only after successful `PostExternal`, with durable transition;
- transient errors retained in `cur/` without arbitrary retry caps; permanent failures have durable diagnostics;
- stable repository source, immutable external ID, byte-identical retry payload, and crash-after-import idempotency;
- exact-path staging, one repository-scoped lock across Git operations, safe push/rebase recovery, and no force push;
- strict fixed configuration and rejection of removed legacy keys;
- no direct Wormhole writer bypassing Comms Core; `PostExternal` is the sole import boundary;
- the current test file's actual correctness and the required commands on the current exact tip:
  `go test ./internal/wormhole/ -count=1 -timeout 10s`,
  `go test ./internal/comms/ -count=1`,
  `go build ./...`.

The current branch also contains Comms routing/claims/dispatch changes, while the OpenCode plugin file is identical to master. Reconcile that with W0-139's WP2/WP3 claims and the owner-approved decision that the agent/session-plugin follow-up is separate. Do not treat those extra changes as proof of W0-134 acceptance.

## Comms receipt requirement

Check actual Comms evidence for any claimed W0-134 delivery or reply to LaForge. A GitHub inbox/outbox commit, a Discovery handoff, a send-attempt log, or a cyclic liveness probe is not delivery proof. If a receipt exists, report all of these exact fields: original Message-ID, sender, recipient, UTC timestamp, Correlation-ID/Task-ID, and durable receipt ID/path. If you cannot inspect Comms or no receipt exists, explicitly leave receipt status **unconfirmed**. Do not invent receipt data.

Do not claim agent liveness without an observable probe. If a pending Ask is visible, report its exact request ID and prompt verbatim; do not answer or approve it. Do not restart, relaunch, detach, replace, duplicate, or modify any existing session.
