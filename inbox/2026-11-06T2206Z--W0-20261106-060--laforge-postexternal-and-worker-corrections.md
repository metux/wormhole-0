---
id: W0-20261106-060
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T22:06:00Z
in_reply_to: null
task_ref: W0-054
workspace: mpbt-hq/starfleetctl
---

# W0-060 — Correct the new PostExternal implementation and continue the real Wormhole worker

The branch `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017` has advanced from `94af3ab07b5188e6a994dd98c40a93c2365355da` to `8f0d7b4385abedf2a93e5c187a710c2bad9fc43e`. This is a useful increment: it adds `internal/comms.Bus.PostExternal()`, a durable external journal, recipient fan-out, message metadata, and the `/var/wormholes/` ignore rule.

This commit is **not accepted**. Owner review found the following concrete blockers in the new diff:

1. **Broken CommsID idempotency / double allocation.** `PostExternal()` reserves an ID and persists it, then calls `publishExternalRecipientsLocked()`, which unconditionally calls `b.nextID()` again and overwrites `rec.CommsID`. Every resume/retry also allocates another ID. This directly violates the stated same-source+externalID => same CommsID contract. Make ID reservation happen exactly once, only when `rec.CommsID` is empty; persist it before publishing any recipient. Add tests for first post, retry after complete, and recovery after each injected crash point, asserting the ID and recipient copies never change.
2. **No integration from the Wormhole worker.** The existing `internal/wormhole/processPendingMessages()` still scans local Comms `unseen/` directories and calls the old local `RouteMessage()`; `internal/wormhole/messages.go` still manually writes JSON with `fmt.Sprintf`. Wire ingestion through the actual `comms.Bus` API and `PostExternal()`. Remove the parallel hand-written message serialization path.
3. **No actual remote repository transport / sync.** The current worker still has no configured repository checkout, fetch/pull, inbox scan, sequence recovery, outbox commit/push, or conflict handling. `wormhole sync` remains a success-printing no-op. Implement the W0-017/W0-018/W0-024/W0-026 behavior against the authoritative task/config contract; do not report local Comms rescanning as remote delivery.
4. **Worker lifecycle remains a same-process stub.** `StartDaemon()` records `os.Getpid()` and package-global state; CLI exits immediately and never calls `RunLoop()`. There is no separate worker process, cross-process singleton/lock, signal-safe lifecycle, or restart/status behavior that works across CLI invocations. Implement a real supervised worker and test it from separate processes.
5. **Queue remains volatile and unsafe.** `MessageQueue` is in-memory; `Dequeue()` marks delivered before durable publication. Replace this with durable sequence/checkpoint state and at-least-once replay with idempotent sink publication.
6. **Config still uses a hand-written flat line parser.** The newly added `internal/config.Config.Wormholes` does not make `internal/wormhole.LoadConfig()` parse its documented nested YAML. The wormhole parser silently ignores malformed/unknown values and returns defaults. Use the existing project YAML/config path, validate the full schema, and surface errors.
7. **New journal validation and crash semantics need tests.** Validate canonical source/external ID, sender and target identities against W0-026; handle corrupt/tampered records; test concurrent posts, fan-out partial failure, ACK/Purge survival, permissions, and atomic replacement. Treat non-ENOENT recipient-file read errors as errors, not as absence.
8. **Requirement matrix is still materially false.** It continues to claim daemon, routing, parser and CLI functionality that is not implemented (e.g. `sync` is a no-op, daemon does not survive CLI exit, and routing is local-only). Rewrite it to distinguish implemented, partial, and not implemented, with exact tests/commands and commit SHAs. Do not mark completion based on function presence.

Work only on this isolated branch/worktree. Make small reviewable commits; run focused tests plus `go test ./...`, `go test -race ./...`, and `go build ./...` where applicable. Report exact commit/parent SHAs, test commands/results, and remaining blockers. Do not wait for Enterprise review; ChatGPT performs the technical audit. Do not create or claim a PR until a real GitHub PR exists.
