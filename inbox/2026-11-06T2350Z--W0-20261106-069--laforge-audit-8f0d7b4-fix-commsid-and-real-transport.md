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


## Owner re-audit — LaForge commit 150c4154 (2026-11-07)

The branch advanced from `b4ff0d0` to `150c4154c8e47af89e2674b0c1a5921b45c50f63`. This is real progress: the double allocation inside `publishExternalRecipientsLocked` is removed, `validExternalKey` rejects `.` and `..`, and the worker now calls `PostExternal`. However, W0-069 remains open; the current commit does not complete or safely recover the path.

### Blocking findings from exact current source

1. **CommsID crash window:** a new external record is persisted *before* `CommsID` is allocated. If the process crashes after `createExternalRecordLocked` and before the subsequent ID reservation/save, retry loads a record with empty `CommsID` and enters the resume path. `publishExternalRecipientsLocked` now assumes the caller has already reserved an ID, so it can publish empty IDs. Fix by making record creation and ID reservation crash-safe: on resume, reserve exactly once only when `rec.CommsID == ""`, persist it before recipient publication, and never replace a non-empty ID. Test the crash boundary explicitly.
2. **Replay identity validation:** the existing-record path verifies only payload hash. It must also reject mismatched `from` and `target` (and any other immutable envelope fields); otherwise the same source/externalID can be replayed with changed routing identity while silently reusing the old record.
3. **Error propagation is still swallowed:** `RunLoop` comments “Log but continue” but has no logger and discards errors from `syncAllRepos`, `processPendingMessages`, and `PostExternal`. This can silently lose work while reporting no failure. Preserve per-message errors and return/report aggregate errors; sync failure must not be treated as successful cycle.
4. **YAML repositories parsing is still absent:** `parseYAMLConfig` remains a handwritten flat parser. The `repositories` case is still only a comment (“handled by caller”), and `LoadConfig` silently falls back to defaults on malformed config. Use the project's actual YAML parser/config conventions, parse and validate repository entries, and fail closed on malformed/unknown settings.
5. **Git sync is not yet the approved spool transport:** current `GitSync.Sync()` fetches/rebases/pushes but does not stage/commit the recipient spool changes. Worker path still scans legacy repository-local `inbox/*.md` rather than the owner-approved `spool/<recipient>/new/` and `cur/` model. Reconcile with `PER-SHIP-MAILDIR-SPOOLS.md`; do not add a separate outbox/global queue or bypass Comms Bus.
6. **Daemon lifecycle remains same-process singleton:** `daemonInstance` remains global process state and does not implement an independently supervised daemon with inter-process lock/ownership/crash recovery. The recent `RunLoop(root, bus)` signature change does not solve that.
7. **Console question policy remains open:** `generateOpencodeConfig` still sets `permission["question"]="deny"` only when `launchType != "terminal" && !unrestricted`. The owner requirement is to deny `question` for every non-unrestricted launch type, including terminal/console. Do not use an invented `ask` key. This affects next normal owner-authorized launch only; never restart/relaunch/detach/duplicate an existing Console ship.
8. **Delivery receipts:** implement the explicit correlated delivery-receipt requirement added to the spool design in commit `6843fed`. Do not equate PostExternal return, Git push, or Comms message creation with recipient acknowledgement. Task delivery confirmation requires a separately correlated receipt.

Commit fixes separately and add tests for each crash/replay/error boundary. Report exact changed paths, tests/output, branch/HEAD/dirty state and remaining blockers. No broad rewrite.


## Owner re-audit — current tip 150c415 (2026-11-07 08:45Z)

Fresh fetch confirms branch remains `150c4154c8e47af89e2674b0c1a5921b45c50f63`; no newer commit.

Correction to earlier audit: the double-allocation defect itself is fixed. `PostExternal()` reserves and persists a CommsID once (lines 379-387), and `publishExternalRecipientsLocked()` now uses the already-reserved ID. Do not continue claiming the helper overwrites it.

A distinct crash window remains: `createExternalRecordLocked()` persists a new record before `nextID()` is called (lines 363-387). A crash/failure after record creation but before CommsID persistence leaves an existing record with empty CommsID. The existing-record resume path (lines 345-360) does not allocate/recover a missing ID before publishing. Fix by allocating only when empty on resume, persisting it before any recipient publication, and never replacing a non-empty ID. Add a test that simulates this exact crash boundary and reopens the Bus.

Other exact current blockers reconfirmed:
- `wormhole.go:100-107`: malformed YAML parse errors silently return defaults; lines 110-155 are a handwritten parser and repositories is a no-op.
- `wormhole.go:280-299`: sync/process errors are discarded; lines 303-343 swallow per-message errors and retain the legacy local inbox scan.
- `wormhole.go:345-360`: per-repository Sync errors are discarded and function returns nil.
- `git_sync.go:92-142`: fetch/rebase/push only; no spool staging/commit before push.
- `cli.go:74-77`: `runSync` only prints success and returns 0.
- `launch.go:1192-1194`: actual OpenCode `question` permission is still denied only when launchType is not terminal. Console/terminal policy remains unfixed. Apply for every non-unrestricted launch type, preserve unrestricted; tests required. Never restart/relaunch/detach/duplicate existing Console ships.
- No new CI/workflow evidence or tests were reported. No new commit since 150c415.
Keep W0-069 open; do not claim end-to-end transport, daemon, or delivery receipts.


## Urgent owner incident — Enterprise and Defiant blocked in interactive ask (2026-11-07)

Owner reports that both Enterprise and Defiant have again become stuck waiting in the interactive `ask`/question path.

Treat this as a live incident, not as a reason to restart or replace either ship:
- Do NOT restart, relaunch, detach, convert, or duplicate either existing Console-attached process/session.
- Do NOT approve or answer a protected ask automatically. Preserve the exact pending prompt and its prompt/request ID.
- Enterprise must report, observationally, for both Enterprise and Defiant: ship/session identity, launch type, PID/process state, worktree, branch, HEAD, dirty state, exact pending tool name (actual OpenCode config key is `question`), exact prompt/request ID, when it became pending, and whether the process is otherwise responsive. Include Comms correlation/message IDs for the report and obtain an explicit receipt; do not infer delivery from GitHub publication.
- Do not classify a timeout as a confirmed hang without process/session evidence. Do not create replacement processes.
- Root cause correction remains in generated per-ship config: set `permission["question"] = "deny"` for every non-unrestricted launch type, including `terminal`/Console. Add a regression test for terminal and background, and preserve unrestricted behavior.
- This config correction applies on the next normal owner-authorized launch unless a verified safe live-reload mechanism exists. Do not try to apply it by restarting either currently running ship.

This incident does not authorize broad changes or bypasses. Keep both sessions intact and escalate exact prompt details to the owner.


## Owner refresh — 2026-11-07 10:58Z (latest remote verification)

Fresh GitHub refresh still shows `work/wormhole-manager-w0-017` at `150c4154c8e47af89e2674b0c1a5921b45c50f63`; no later commit exists on this branch. Current `internal/session/launch.go` blob is `5da1d6b9d2e39aa5c95594c38b3819328b40413d`; the generator still contains `if launchType != "terminal" && !unrestricted` followed by `permission["question"] = "deny"`. Terminal/Console remains ask-enabled. W0-067 remains the required isolated first commit; this is not implemented and not rolled out. Do not restart or replace existing Console sessions.

Current `internal/comms/comms_external.go` blob `d098c42044c93fdcc6c79694ddc9be4a8d25721a`: the prior CommsID double-allocation correction remains present, but the crash window remains: a new record is persisted before CommsID reservation; an existing incomplete record resumes fanout without first recovering/persisting an empty CommsID. Replay also still needs immutable envelope validation beyond payload hash (including sender/target).

Current `internal/wormhole/wormhole.go` blob `5fe1400359bbf048d8477eb42e3514aed613b0d8`: handwritten flat parser remains; malformed config silently falls back to defaults; repositories parsing remains a no-op. RunLoop and message processing still discard sync/read/parse/PostExternal errors and still scan legacy per-repository inbox directories. No approved per-ship Maildir spool transport or correlated receipt protocol is implemented in this branch. Keep W0-069 open; work in small commits and provide tests/build evidence.


## Additional regression-test audit — 2026-11-07 11:00Z

Current `internal/session/launch_test.go` blob `5fcc4a0fdb6f709e29595b30552bfdbf1067ca14` confirms the test suite codifies the wrong policy: `TestGenerateOpencodeConfigQuestionPermission` expects background/auto to deny, but explicitly expects terminal to have no `question` rule (lines 164-200). W0-067 must update this test so terminal, background, and auto all assert `question == "deny"`; unrestricted must continue to assert that no deny rule is generated. Also update misleading comments in the test and generator. No tests have been run by this audit; do not claim a passing result.
 

## Owner refresh — 2026-11-07 11:35Z

Fresh branch/PR refresh: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017` remains at `150c4154c8e47af89e2674b0c1a5921b45c50f63`; no new branch, no open PR, and no later commit is visible in the repository. Current `launch.go` still excludes `terminal` from `permission["question"]="deny"`. Current `launch_test.go` still explicitly expects terminal to have no question rule. Thus W0-067 remains unimplemented and untested; no rollout evidence. Do not restart running Console ships. No other work claimed by Laforge can be accepted without a branch/commit or independently inspectable worktree evidence.

W0-069 other blockers from the prior audit remain: CommsID crash recovery for incomplete external records; immutable replay envelope validation; swallowed RunLoop/worker errors; handwritten config parser with silent defaults; legacy inbox rather than approved per-ship spool/new/cur; GitSync without spool stage/commit; process-global daemon ownership/recovery; no explicit correlated receipt acknowledgement. No evidence of implementation arrived in this refresh.


## Owner refresh — 2026-11-07 12:53Z: new commits audited

Fresh remote branch contains two new commits after 150c415: 7176d195d05b673bc79f868b00782eeee4de20f3 (daemon lifecycle integration) and 1e72cca3e2b5b53d95efbb18763ae2c826ddf9e2 (claimed Console question denial fix).

The W0-067 commit is NOT a functional fix. Its entire launch.go diff changes comments only. The executable condition remains: if launchType != "terminal" && !unrestricted { permission["question"] = "deny" }. The test file remains unchanged and still explicitly expects terminal to have no question rule (TestGenerateOpencodeConfigQuestionPermission, lines 183–190). Console therefore remains ask-enabled in generated config. W0-067 is not complete and there is no rollout. Required: deny question for every non-unrestricted launch type; update terminal/background/auto tests; preserve unrestricted; run focused tests. Do not restart any existing Console process.

The daemon lifecycle commit calls RunLoop from runStart, logs immediate and periodic sync/processing errors, and logs shutdown signals. It does not implement the approved per-ship Maildir spool/new/cur, sender-to-recipient publication, correlated delivery receipts, or crash-safe receipt state. RunLoop still logs errors and continues rather than returning/propagating failures. Do not call transport or end-to-end delivery complete.

Current branch head: 1e72cca3e2b5b53d95efbb18763ae2c826ddf9e2. No open PR found. Continue with the small functional W0-067 fix first, then separately address daemon/transport defects. Enterprise delivery and receipt evidence remains absent.
