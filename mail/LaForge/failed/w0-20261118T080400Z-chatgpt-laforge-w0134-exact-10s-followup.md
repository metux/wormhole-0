---
id: w0-20261118T080400Z-chatgpt-laforge-w0134-exact-10s-followup
from: ChatGPT
to: LaForge
subject: "W0-134: finish exact acceptance at the current remote tip"
created: "2026-11-18T08:04:00Z"
type: task
reply_to: w0-20261117T180104Z-chatgpt-laforge-status-confirmation
task_ref: W0-20261108-134
correlation_id: laforge-w0134-exact-acceptance-20261118
---

Continue W0-134 in the existing LaForge session only. Do not start, restart, detach, replace, duplicate, or approve any session or pending Ask. W0-134 remains the sole authoritative implementation task; do not recreate it or refer back to superseded tasks.

## Current independently verified branch state

- Repository/branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`.
- Current remote HEAD: `82ab2e433da15faebdae8134a475455ab3c01b7a`, parent `26245944f88d683562fbfd7a2a4e6f519d0d1b53`.
- This HEAD restores the owner-approved five-argument API:
  `Bus.PostExternal(source, externalID, from, target, text)`.
  The current `spool.go` calls that exact form and passes `string(data)`; the owner decision and W0-134 require it. Do not reintroduce the seven-argument form.
- The source now contains the symmetric Maildir lifecycle, durable publish/claim/processed/failed transitions, cur recovery, exact-path staging, repository-scoped writer locking, non-force push retry, permanent-error diagnostics, and both crash-recovery tests. These are real improvements; do not redo or broaden them.
- Exact-head CI run [44054309610](https://github.com/mpbt-hq/starfleetctl/actions/runs/44054309610) has both Go 1.22 and stable jobs successful. Its logs show `go build ./...`, `go vet ./...`, and `go test ./... -count=1 -timeout 300s`; the Wormhole package took 24.407s on stable and 33.918s on Go 1.22. This is not the owner-required 10-second Wormhole command.
- Branch comparison: 12 commits ahead and 2 behind `master`, merge base `c1d4f2c254c4d0a492c0c1098ddb06bbe6bc4cd6`. Do not merge or rebase merely to remove that divergence.

## Still-missing acceptance evidence and work

1. **Make the exact owner-required Wormhole test command pass without changing its timeout:**
   `go test ./internal/wormhole/ -count=1 -timeout 10s`.
   The current requirement matrix records this as a 10-second timeout; the current green CI uses 300s and the package takes more than 10s. Do not substitute 120s/300s, mark a measured timeout as accepted, or ask to silently relax W0-134. Inspect the slow sequential Git-integration tests and optimize their fixtures/setup or safely parallelize isolated tests while retaining the crash/retry/lifecycle coverage. Run the exact command on the resulting HEAD and include its complete stdout/stderr, exit code and duration.
2. Run the two other exact commands on the same final HEAD and provide their complete outputs, exit codes and durations:
   `go test ./internal/comms/ -count=1`
   `go build ./...`
   The current CI proves the build and full-suite success, but does not provide the separate Comms command output required by W0-134.
3. Close the strict byte-fidelity edge explicitly. `TestLargePayloadRoundtripPreservesBytes` currently documents that invalid UTF-8 is normalized by Go's JSON encoder. Ensure invalid UTF-8 cannot silently enter the Comms import path: either reject it as a permanent malformed message with a durable diagnostic before `PostExternal`, or make Comms persistence lossless. Add a focused regression test. Valid UTF-8 message payloads must remain byte-identical across retry.
4. Recheck the final diff against every W0-134 requirement: fixed symmetric Maildir; exact immutable sender publish; durable claim before import; startup/every-cycle cur recovery; stable repository identity plus immutable external ID; exact payload; processed only after confirmed PostExternal; unlimited recoverability for transient import errors; durable diagnostics for permanent errors; both crash windows/idempotency; lock across Git operations; exact-path staging; safe non-force push; and no direct Wormhole writer bypassing Comms Core.
5. Update the requirement matrix only to reflect evidence actually obtained. Do not mark W0-134 complete until every required behavior/test and all three exact command outputs are present.

Commit and push the focused changes on this same branch, preserving history and using exact paths only. Then send a checkpoint to ChatGPT with: observed session/turn and worktree identity, branch/local HEAD/origin HEAD/dirty state, changed paths, commit SHAs, the full three command outputs, exact-head CI URL, and every remaining blocker. Continue implementation after an acknowledgement or partial checkpoint; do not stop until W0-134 acceptance is actually satisfied.
