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


## 4. Owner re-audit of new commits through 2026-11-07 00:26 CET

New branch commits now exist; do not continue reporting the old 8f0d7b4 tip:
- `ec53c89` adds `internal/wormhole/git_sync.go`.
- `ee5fc25` reverts the Autostart addition from `5a99eee` and restores the old immediate local processing loop.
- `5a99eee` briefly added Autostart, but it still delegates to the process-global daemon singleton.
- `b4ff0d0` adds repository config wiring and calls `syncAllRepos`; current branch tip observed is `b4ff0d00d367595fec637f888d6d961fd78f48b3`.

These are real progress, but not an end-to-end implementation. Current source audit finds:
- `syncAllRepos` silently discards every `GitSync.Sync()` error and always returns nil.
- `parseYAMLConfig` is still the handwritten flat parser. The `repositories` case is only a comment ("handled by caller"); no repository list is parsed. The former `persist_state` case was removed in `b4ff0d0`.
- `GitSync.Sync()` fetches/rebases/pushes, but the current implementation does not stage or commit spool/message changes. It therefore cannot publish local messages as commits.
- `RunLoop` still uses `daemonInstance` and the existing local Comms file processing path; this is not an independently running daemon or a complete remote message worker.
- `internal/comms/comms_external.go` still allocates `CommsID` in `PostExternal` and then unconditionally allocates it again inside `publishExternalRecipientsLocked`. The stable-ID blocker is still present in the current branch source.
- `internal/session/launch.go` still explicitly leaves terminal ships with the default question prompt: `if launchType != "terminal" && !unrestricted { permission["question"] = "deny" }`. W0-067 remains open. Use the actual `question` key; do not restart or duplicate any running Console ship.

## 5. Owner-approved spool direction (2026-11-07)

Record and follow the design draft `design-drafts/PER-SHIP-MAILDIR-SPOOLS.md` in `metux/wormhole-0` (commit `ed737f0b5692470553a538317281aa6c8843b5d3`):
- One Maildir-like spool per recipient ship: `spool/<ship>/new/` and `spool/<ship>/cur/`.
- Sender writes directly into the target recipient's `new/` using a cryptographically random Message-ID.
- No separate `outbox/` and no extra global/unsorted queue.
- Message contents are immutable; retries reuse the same ID and bytes; duplicate same-ID/different-content fails closed.
- Recipient alone owns its `new -> cur` read/pickup transition. `cur` is not proof of processing or acknowledgement.
- Fanout gives each recipient an independent copy/state; push is not proof of delivery/read/processing.
- Git synchronization must preserve all local changes through fetch/rebase/push races; no force-push or destructive reset.
- Do not start a broad rewrite from this draft. First reconcile the current Comms Bus/PostExternal integration, W0-026 idempotency rules, daemon/worktree lifecycle and the draft into a concrete incremental plan. Do not invent a parallel Comms routing path.

Continue with small commits and focused tests. Report actual branch tip, dirty state, exact test output and remaining blockers.

## 6. Mandatory explicit delivery receipts (owner clarification, 2026-11-07)

Implement this as a first-class protocol requirement, not merely an optional note in reporting. Reconcile with the updated `design-drafts/PER-SHIP-MAILDIR-SPOOLS.md` (commit `6843fed85f637741a0d1b95dcf60c72aab02687b`):

- For coordination/task messages that request confirmation, the recipient must publish a separate immutable delivery-receipt message back into the original sender's per-ship `new/` spool after it has fetched the original and durably picked it up (`new -> cur`).
- The receipt must identify the original Message-ID, sender, acknowledging recipient and UTC receipt time, and preserve Correlation-ID/Task-ID. The sender marks delivery confirmed only after it fetches and validates this correlated receipt.
- A successful push, shared-repository visibility, or the recipient's `new -> cur` transition alone is not a delivery confirmation visible to the sender.
- Retries/restarts must preserve one stable receipt identity and exact bytes; duplicate receipt publication is idempotent. Fanout requires one independent receipt per requested recipient.
- Delivery receipt is not processing/completion acknowledgement. Report processing outcome separately and correlate it to the original Message-ID.
- Add tests for missing/delayed receipt, duplicate/replayed receipt, crash between pickup and receipt publication, fanout partial receipt, wrong recipient/original ID, and sender-side validation. Never report delivery as confirmed without the fetched receipt evidence.

Keep the implementation incremental: first produce a concrete Comms Bus/W0-026 integration plan for these receipts and the approved per-ship spool model; do not start a broad rewrite.
