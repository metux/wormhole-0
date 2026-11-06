---
id: W0-20261106-069
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T2350:00Z
priority: urgent
workspace: mpbt-hq/starfleetctl
branch: work/wormhole-manager-w0-017
---

# Owner audit of 8f0d7b4: fix idempotency and complete the real transport

The latest branch tip is 8f0d7b4385abedf2a93e5c187a710c2bad9fc43e. It is NOT accepted as complete. Continue on this branch; do not merge or claim completion.

## 1. Blocking PostExternal CommsID corruption

In `internal/comms/comms_external.go`, `PostExternal` reserves `id := b.nextID()`, stores it in `rec.CommsID`, and then calls `publishExternalRecipientsLocked`. That helper unconditionally calls `b.nextID()` again and overwrites `rec.CommsID`. On retry/recovery it does the same again. This violates the documented invariant that one (source, externalID) maps permanently to one CommsID; a partial fanout retry can publish different recipient copies under different IDs and strand duplicates.

Fix allocation in exactly one place. Reserve only when `rec.CommsID` is empty, persist it before any recipient publication, and never change it on resume. Add tests for:
- initial publication allocates exactly one stable CommsID;
- failure after one recipient then retry preserves the same CommsID for every recipient;
- retry after reconstructing/reopening the Bus preserves the same CommsID;
- complete replay does not allocate another ID;
- changed payload, source/ID mismatch, corrupt journal and filesystem errors fail closed.
Also review create-if-absent and atomic journal writes for concurrent callers/processes; a process-local mutex alone is not sufficient if multiple starfleetctl processes can publish concurrently.

## 2. W0-060/W0-065 remain unimplemented in the manager

The current `internal/wormhole/wormhole.go` still contains a hand-written flat-line YAML parser, same-process global `daemonInstance`, local Comms `unseen/` directory scanning, and a run loop that never fetches or pushes a configured remote repository. `internal/wormhole/cli.go:runSync` is still a success-printing no-op. The current MessageQueue is volatile and RouteMessage still manually formats JSON. The requirement matrix therefore overclaims implementation.

Do not mark W0-017/W0-018/W0-024/W0-026 complete. Implement in small independently testable commits:
1. Wire the worker to the actual `internal/comms.Bus` and `PostExternal` API; remove the parallel local-file routing path.
2. Implement remote checkout/fetch/pull, deterministic inbox discovery, durable sequence/cursor recovery, outbox commit/push, conflict handling, and idempotent retries according to W0-026.
3. Implement a real independently running daemon with inter-process lifecycle/locking and crash-safe state; no same-process singleton masquerading as a daemon.
4. Use the repository's real YAML parser/config conventions; reject malformed and unknown configuration instead of silently defaulting.
5. Replace the in-memory queue and manual JSON serialization; validate recipient identity/aliases and paths.
6. Add process/restart/crash/concurrency/replay/sequence/fanout/remote integration tests. Update the requirement matrix only after each behavior is actually covered.

Before editing, inspect current `internal/comms.Bus` APIs and existing service lifecycle patterns. Do not invent API names. Keep the existing branch and unrelated files untouched.

## 3. Console question policy still not applied

At the current branch tip, `generateOpencodeConfig` still sets `permission["question"]="deny"` only when `launchType != "terminal" && !unrestricted`. Console/terminal ships therefore retain the interactive question prompt. W0-067 is still open: deny the actual OpenCode config key `question` for every non-unrestricted launch type, including terminal/console; preserve unrestricted behavior and all other permissions. Do NOT add a made-up `ask` key. Add terminal/background/auto/unrestricted tests.

Do not restart, relaunch, detach, convert, or duplicate any existing Console ship to apply this. The generated config affects the next normal owner-authorized launch unless a verified safe live-reload path exists. Existing running Console ships must remain untouched.

## Reporting

Commit each coherent fix separately. Report exact commit SHAs, tests and outputs, changed paths, remaining blockers, and the actual worktree/branch/HEAD. Do not claim remote transport, daemon, Comms delivery or probes unless implemented and evidenced.
