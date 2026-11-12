---
id: W0-20261109-140
kind: review-checkpoint
from: Enterprise
to: owner-coordination
created: 2026-11-09T14:45:00Z
in_reply_to: W0-20261109-137
task_ref: W0-20261108-134
correlation_id: enterprise-w0134-independent-review-137
workspace: mpbt-hq/starfleetctl@origin/work/wormhole-manager-w0-017
---

# W0-140 — Independent W0-134 review checkpoint (read-only)

## Scope / inspected state (source inspection unless stated)

- Requested tip in W0-137: `38899e3de1e1f20cf63e1ef52bc66b2e1cf25b39` (exists locally; commit subject “W0-134: implement Maildir spool lifecycle with Git commit/push per transition”).
- Actual current remote tip at inspection time: `65c0dfa3a016e5dcc5be83b6db6751a82d538a7e` (`origin/work/wormhole-manager-w0-017`, fetched). `git log 38899e3..origin/work/wormhole-manager-w0-017` shows 20 commits after 38899e3 (d41dd87..65c0dfa chain including f4cc8fe etc.). Review below is based on source/tests at HEAD `65c0dfa` (exact files read from detached worktree of that tip) because it is the live branch tip; 38899e3 is not the current tip.
- Inspected files: `internal/wormhole/spool.go`, `git_sync.go`, `comms_transport.go`, `messages.go`, `wormhole.go`, `cli.go`, `internal/comms/comms_external.go`, `internal/comms/routing.go`, `doc/wormhole*.md`. Searched for direct comms writers (`RouteMessage`, `.starfleet-ai/var/comms/msgs` in wormhole) — none found.

## Required command evidence (run at HEAD 65c0dfa, worktree `_WORK_/starfleetctl/sources/wormhole-review-w0134`)

1. `go test ./internal/wormhole/ -count=1 -timeout 10s`  
   **FAIL (timeout, exit 1).** Panicked after 10s: `panic: test timed out after 10s` while running `TestTwoInstanceRemoteImportE2E` (`wormhole_test.go:1272`), stuck in git/clone/validate/publish path (`gitRevParseIn`/`EnsureCloneLocked`/`Publish`/`postAs` per stack). `FAIL github.com/metux/starfleetctl/internal/wormhole 10.021s`.
2. `go test ./internal/comms/ -count=1`  
   **PASS**: `ok github.com/metux/starfleetctl/internal/comms 2.294s`.
3. `go build ./...`  
   **PASS** (exit 0).

## Criterion-to-evidence (W0-134)

| Criterion | Evidence (file:function/line, behavior) | Tests/observation | Verdict |
|---|---|---|---|
| Fixed symmetric Maildir `mail/<target>/{new,cur,processed,failed}` inside configured checkout; sender publishes immutable file to `new/<id>`; no separate sender outbox | `spool.go:20-38,40,87-101` constants/paths `MaildirRoot/MaildirName/mailDirName`; `Publish` writes exact `mail/<to>/new/<id>.md` via temp+rename (`spool.go:143-198`, rel built `s.rel(msg.To, maildirNew, name)`), commits+pushes exact path. No sender-side post-send move in publish path. | Source inspection. Wormhole suite did not complete within 10s (timed out in `TestTwoInstanceRemoteImportE2E`); cannot cite passing publish/receive test from this run. | **PARTIAL/BLOCKED by test timeout** — layout/publish logic present; required test command failed. |
| Receiver: discover `new/`, atomically claim to `cur/` **before** import; commit+push claim before import; resume `cur/` on startup/cycle | `processNewLocked` (`spool.go:288-339`): reads `new/`, validates, checks no cur collision, `os.Rename(newAbs,curAbs)` then `CommitAndPushLocked([new,cur],"wormhole: claim "+msg.ID)` and only then `importLocked(...)`; on commit error keeps in cur and records error (no import). `processCurLocked` (`spool.go:341-379`): iterates `cur/`, re-asserts claim via `CommitAndPushLocked([new,cur],"wormhole: claim "+msg.ID)` before import (recovers crash between rename and claim commit). `runOnce`/`processLocked` always call `processCurLocked` then `processNewLocked` each cycle under lock (`spool.go:107-124,199-235`). | Source inspection. No completed passing lifecycle test evidence from required run (timeout). | **PARTIAL** — code path present; durability/push-before-import depends on `CommitAndPushLocked` semantics and remote state (see below). |
| Import only via `Bus.PostExternal(source, externalID, from, target, msgType, replyTo, text)` with stable repo source + immutable id + exact original payload bytes | `importLocked` (`spool.go:403-428`): `bus.PostExternal(s.repo.ID, msg.ID, msg.From, target, msgType, msg.ReplyTo, string(data))` using read file bytes `data`; no re-serialization. `s.repo.ID` is stable configured repo identity (`newSpool` stores repo). `parseMessage`/id must match filename (`parseForTarget` `spool.go:384-396`, `messages.go` parse/validate). | Source inspection. `internal/comms` tests PASS (include PostExternal suite per package). | **LIKELY IMPLEMENTED** (source + comms tests green) but end-to-end wormhole import not verified by completed wormhole test run. |
| After confirmed import: move `cur -> processed`, commit+push; transient failure leaves recoverable in `cur/` (no arbitrary retry cap); permanent malformed -> `failed/` with durable diagnostic | `importLocked`: on PostExternal error appends `import ... failed (kept in cur/)` and returns without move (`spool.go:414-417`); on success renames cur->processed and commits+pushes exact paths (`418-427`). `failLocked` (`spool.go:433-468`) moves to `failed/` (collision-safe name), writes `<name>.reason` diagnostic with mailbox/from/failed_at/reason, commits+pushes `[fromState, failed/msg, failed/reason]`. Permanent parse/recipient/id-mismatch handled in `processNewLocked`/`processCurLocked` via `parseForTarget` -> `failLocked`. No retry counter in import path. | Source inspection. | **IMPLEMENTED in code** (behavioral guarantees not fully exercised by completed tests in this run). |
| Crash-after-import-before-processed safe/deduped by Comms idempotency | Same `source=s.repo.ID`, `externalID=msg.ID`, same payload bytes passed on every retry/resume (`processCurLocked` re-imports items left in cur). `PostExternal` contract/idempotency is in `internal/comms/comms_external.go` (journal/dedupe by source+externalID+hash; comms tests PASS). | `go test ./internal/comms -count=1` PASS. Wormhole crash-specific e2e not confirmed (timeout). | **PLAUSIBLE** given code + comms tests; not fully end-to-end proven this run. |
| Git durability/concurrency: every meaningful transition own commit+pushed immediately; exact-path staging only; one repo-scoped writer lock for sync+spool ops; push failure preserves local commit, rebase on non-ff under lock, never force-push; commit vs push states distinct | `Spool.Publish/runOnce` take `git.Lock(60s)` (flock) around sync+ops (`spool.go:107-116,151-160`, `git_sync.go:104-111`). `CommitAndPushLocked` -> `commitExactPathsLocked` (isolated temp index seeded from HEAD, stages exact rel paths only via `stageExactAtLocked`/`git add -- <rel>`, no broad staging) then `pushLocked` (`git_sync.go:542-601`). Dirty mail recovery stages exact paths from `git status --porcelain -z -- mail` (`commitDirtyMailLocked`, `606-633`). `pushLocked` retries bounded on non-fast-forward, does fetch+rebase between attempts, no force-push; read-only returns nil (no push) (`443-473`, `isNonFastForward` 433-441). `rebaseLocked` aborts on failure, refuses dirty tree; uses `-c merge.directoryRenames=false -X no-renames` (`403-431`). `SyncLocked` does dirty-mail commit, fetch, rebase, push (`146-164`). `publish`/`claim`/`processed`/`failed` all go through `CommitAndPushLocked`. | Source inspection. | **MOSTLY IMPLEMENTED**, with caveat: for read-only repos `pushLocked` is no-op so transitions are committed locally but **not remotely published**; yet claim/processed paths still call CommitAndPushLocked and proceed on local commit success (consistent with read-only handling noted in code comments), which differs from “push-confirmed before next step” in remote-publish sense. |
| Comms boundary: `PostExternal` sole import; no reachable Wormhole direct writer to `.starfleet-ai/var/comms/msgs`; `PublishRemoteMessage` registered | `importLocked` uses `bus.PostExternal` only (`spool.go:414`). No `RouteMessage` / no `var/comms/msgs` writes in `internal/wormhole` (grep clean). Transport registered in `comms_transport.go:21` (`InstallCommsTransport` -> `RegisterRemoteTransport("wormhole", PublishRemoteMessage)`), publishes via spool `Publish` (`27-73`). | Source inspection. | **IMPLEMENTED** (by inspection). |
| Fixed config model; reject removed legacy keys/aliases; no configurable incoming/outgoing folder names | `wormhole.go:119-238` strict `LoadConfig` with `validateYAMLKeys`/`allKnownKeys` (fixed schema: `wormholes{enabled,sync,repositories}`; repo `{id,remote,branch,enabled,access,incoming}`; incoming `{enabled,initial_scan}` only) — unknown keys/paths rejected. `ValidateConfig` checks durations/ranges/ids/access/initial_scan and active-repo remote/branch when enabled (`58-117`). No configurable maildir/path fields in schema. | Source inspection. | **IMPLEMENTED** (by inspection). |

## Additional observations / risks

- **BLOCKER (acceptance):** Required command `go test ./internal/wormhole/ -count=1 -timeout 10s` does **not** pass; it times out at 10s in `TestTwoInstanceRemoteImportE2E` (git clone/validate path). W0-134 requires complete, passing output for this command at the reported HEAD — currently unmet.
- **Tip discrepancy:** W0-137 references `38899e3` as current remote implementation tip; actual `origin/work/wormhole-manager-w0-017` is now `65c0dfa` (20 commits ahead per `git log 38899e3..HEAD`). This report evaluates HEAD 65c0dfa; I did not re-run the full wormhole suite on 38899e3 within this session (separate checkout build/test would be needed). Any acceptance claim must be tied to the exact tip tested.
- **Remote-publish semantics:** Claim/processed/failed transitions are “committed and (for rw) pushed” via `CommitAndPushLocked`; for read-only repos push is skipped but code proceeds after local commit. This is an intentional local-consistency behavior per comments but means “remotely published before import” is not satisfied for read-only targets — worth explicit owner decision if read-only mailboxes are in scope.
- **Process state:** no live wormhole worker identity beyond var state file observed here; daemon/runtime liveness not separately load-tested.

## Comms delivery/receipt verification (W0-134)

- No W0-134 Comms delivery or durable correlated receipt was claimed by me in this session. I cannot verify a Comms receipt from wormhole-0 git artifacts (inbox/outbox files/commits) — per instructions those are **not** Comms receipts. Comms endpoint/receipt for W0-134 remains **unconfirmed** from available evidence here.

## Overall assessment

**NOT ACCEPTED / BLOCKED** for W0-134 at inspected tip `65c0dfa`: core Maildir lifecycle, PostExternal-only import boundary, repo lock + exact-path commit machinery, and strict fixed config are present in source and `go test ./internal/comms` + `go build ./...` pass; however the required `go test ./internal/wormhole/ -count=1 -timeout 10s` fails by timeout (TestTwoInstanceRemoteImportE2E), so end-to-end lifecycle/concurrency/restart guarantees are not verified by the mandated command on the reported HEAD. No qualifying Comms receipt evidence provided/verified.
