---
id: w0-20261118T200000Z-laforge-w0134-acceptance-report
type: report
reply_to: W0-20261118-148
correlation_id: fleet-idle-20261118-laforge-w0134
task_ref: W0-20261108-134
---

# W0-134 acceptance report — criterion by criterion

Reported by: LaForge
Exact HEAD measured: `127b3ed35641f52eee21197a2e4171b848626535` on branch
`work/wormhole-manager-w0-017` (mpbt-hq/starfleetctl), pushed and verified via
`git ls-remote` (ahead=0 behind=0). Working tree clean at measurement time.
Environment: 8-core shared box, load average 39–58 during all runs (other
ships active); this materially inflates wall times below.

## Acceptance commands (exact, on the reported HEAD)

| command | exit | output tail | log |
|---|---|---|---|
| `go build ./...` | 0 | (clean) | final-build.log |
| `go test ./internal/comms/ -count=1` | 0 | `ok ... internal/comms 3.971s` | final-comms.log |
| `go test ./internal/wormhole/ -count=1 -timeout 10s` | **1** | `FAIL ... 10.1s` (panic: test timed out after 10s) | final-10s.log, g10c*.log |
| `go test ./internal/wormhole/ -count=1 -timeout 300s` | 0 | `ok ... internal/wormhole 53.083s`, 0 FAIL of 90 tests | final-300s.log |

10s-window runs (3x): rc=1, wall 12.7–14.1s, panic at 10.1s; still-running
tests at panic were ordinary parallel tests (daemon lifecycle + crash
recovery), i.e. the window is exceeded by aggregate load, not by a hang or
a failing test. **Status: BLOCKED on this box under load 40–58.** At lower
load the same HEAD measured ~12s wall (158s before this branch). Per the
standing lesson (2026-11-16), a 10s window for ~90 git-integration tests is
suite-design, not a timeout choice; the remaining gap is environmental load.

## What this branch changed (commits 82ab2e4, d3fafe3, 127b3ed)

- 5-arg owner-approved `Bus.PostExternal(source, externalID, from, target,
  text)` contract (82ab2e4; type/reply_to roundtrip covered by
  TestTypeAndReplyToRoundtrip).
- Suite parallelization + fixture cost (d3fafe3, 127b3ed): TestMain
  neutralizes fleet env and sets cheap-git env; `testing.Init()`+flag.Set
  raises -parallel to NumCPU*8; ship identity via ship-name registry
  (reserveShip) + explicit testBus() pin instead of t.Setenv → 87/89 tests
  t.Parallel(); template bare remote built once; test binary built once and
  exported via STARFLEETCTL_BINARY; bootstrap syncs became clone-only
  ensureClone(); daemon tests use a 500ms-interval config without startup
  sync; fixed sleeps became log/process polls; git maintenance/gc/compression
  disabled via GIT_CONFIG_* env.
- Invalid UTF-8: `parseForTarget` rejects non-UTF-8 payloads as permanently
  malformed before PostExternal (no silent U+FFFD loss); regression test
  TestInvalidUTF8MovesToFailed asserts failed/ + durable .reason and that
  nothing reaches the local bus.

## Criterion-by-criterion

- Durable wormhole transitions (claim/processed/failed, crash recovery):
  covered and green (TestPostExternalCrashRecoveryImportsExactlyOnce,
  TestCrashRecoveryCommitsDirtyTransition, TestFailedTransition* family,
  TestInitialScanCrash*, …) — pass at 300s.
- Exact payload preservation: TestLargePayloadRoundtripPreservesBytes,
  TestTypeAndReplyToRoundtrip — pass.
- Stable repository/message identity: TestExternalRecordKeyMismatch*,
  collision fail-closed in Publish — pass.
- Crash recovery around PostExternal: TestTemporaryPostExternalErrorKeepsMessageInCur,
  TestPermanentPostExternalErrorMovesToFailed — pass.
- Remote push/rebase durability: TestSyncUsesConfiguredPushRetries,
  TestPushRecoversFromConcurrentRemoteUpdate — pass.
- Invalid UTF-8 before PostExternal + regression test — implemented, pass.
- Required acceptance command with -timeout 10s: FAIL under current box load
  (see above); green at -timeout 300s. Not silently reinterpreted.

## Remaining blocker

`go test ./internal/wormhole/ -count=1 -timeout 10s` exceeds the window only
under heavy external load (load avg 39–58 on 8 cores). Suite wall time on this
branch: ~158s (pre-branch, serial) → ~53s at load ~40 → ~12s at moderate load.
Next safe action: re-run the 10s command on a quieter runner, or owner decision
on the acceptance window (suite design vs timeout).

Delivery note: published to the active Maildir (mail/ChatGPT/new/), not the
frozen outbox/ archive.
