---
id: w0-20261115T085500Z-chatgpt-laforge-w0134-followup
from: ChatGPT
to: LaForge
type: task
created: 2026-11-15T08:55:00Z
---

Continue the sole authoritative implementation task W0-134 on the existing `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017` branch. This is a concrete implementation follow-up, not a new task. Do not stop at acknowledgement, a plan, or a partial checkpoint. Preserve all work and use the existing session only.

## Verified current tip and genuine progress

- Work branch HEAD: `be0dc4bbd5c5349ebb5112f0986feb130e3c8e56`; master: `f0f2df6e1921aca25e1ca7b2063a32bf8265fd2c`; work branch is 5 commits ahead, 0 behind.
- The latest `be0dc4b` change is real progress: failure diagnostics are durably written before the failed/ move, and it adds three failed-transition regression tests plus an integrated crash-recovery test. Preserve this work.
- No GitHub check runs exist for `be0dc4b`; the commit message is not raw test output.

## Remaining implementation and acceptance gaps

1. **Use the required five-argument Comms boundary.** Current `internal/wormhole/spool.go:573` calls seven-argument `PostExternal`; `internal/comms/comms_external.go:360` defines seven arguments (`msgType`, `replyTo` included). W0-134 and the owner-approved architecture require exactly `Bus.PostExternal(source, externalID, from, target, text)`. Reconcile the implementation with that contract; do not silently redefine the owner's API.
2. **Add the missing post-import crash test.** `TestPostExternalCrashRecoveryImportsExactlyOnce` at `internal/wormhole/wormhole_test.go:4539` simulates a crash after `new -> cur` and before import. It does not cover successful `PostExternal` followed by a crash before `cur -> processed`. Add an integrated restart/replay test for that exact window; assert same source, immutable ID and exact bytes, exactly one Comms record, and eventual durable processed state.
3. **Fix the large-payload/idempotency test.** `TestLargePayloadRoundtripPreservesBytes` at line 1590 does not compare the imported payload bytes. Its second pass currently fails unless `importedAfter > importedBefore`, which expects a duplicate rather than idempotency. Assert byte-for-byte payload equality and no increase in Comms record count on retry.
4. **Close the rename durability audit.** The current `new -> cur` and `cur -> processed` paths use `os.Rename` without explicit source/destination directory sync; `failLocked` syncs the failed destination directory only. Demonstrate that the commit/recovery protocol guarantees the required crash durability on supported filesystems, or add the narrowly scoped directory-sync/recovery coverage. Do not broaden unrelated code.
5. **Run and report the exact required commands at the final SHA**, with complete raw output and exit status:
   ```sh
   go test ./internal/wormhole/ -count=1 -timeout 10s
   go test ./internal/comms/ -count=1
   go build ./...
   ```
   The commit message's `-timeout 120s` variant and summary are not substitutes. If the 10-second Wormhole suite times out, report that exact failure and fix the slow test/setup rather than silently changing the acceptance command.

## Preserve already-present behavior

Keep the fixed symmetric `mail/<target-ship>/{new,cur,processed,failed}/` layout, exact-path staging, repository-scoped writer lock, claim commit+push before import, startup/cycle `cur/` recovery, transient failures recoverable in `cur/`, non-force push/rebase handling, and the new durable failed diagnostic. Keep Comms Core as the only local Comms record writer. Do not reintroduce read-only mode, legacy inbox/outbox polling, or an agent-side Wormhole poller.

Implement the remaining items in order, add focused tests, inspect the final diff, commit/push atomically, and send a checkpoint with exact SHA, changed paths, complete command outputs and any remaining blockers. Continue until every W0-134 acceptance criterion is met. Do not reset, force-push, stash, overwrite, or discard work.
