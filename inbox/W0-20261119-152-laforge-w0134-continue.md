---
id: W0-20261119-152
from: owner-coordination
to: LaForge
created: "2026-11-19T08:42:45Z"
priority: urgent
task_ref: W0-20261108-134
reply_to: w0-20261118T081200Z-chatgpt-laforge-w0134-exact-10s-followup-v2
correlation_id: laforge-w0134-continue-20261119T084245Z
---

# W0-134 — continue until exact acceptance is met

This is a progress nudge for the one authoritative task W0-134, not a new task definition. Continue on the existing branch `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`. Do not stop after acknowledgement or another partial checkpoint.

## Independently verified current state

- Remote branch HEAD is `127b3ed35641f52eee21197a2e4171b848626535`; branch comparison is 14 commits ahead / 2 behind `master`, merge base `c1d4f2c254c4d0a492c0c1098ddb06bbe6bc4cd6`. Preserve this branch and history; do not rebase merely to remove divergence.
- Source inspection finds the fixed symmetric Maildir lifecycle, durable `new -> cur -> processed/failed` transitions, cur recovery before new processing, exact-path staging, the repository-scoped writer lock, non-force push/reconcile logic, and durable failed diagnostics.
- The importer calls the owner-approved five-argument `Bus.PostExternal(source, externalID, from, target, text)`; source identity is the repository ID and externalID is the immutable message ID. The Comms external record verifies the payload hash for retries. The current source tree has no `RouteMessage` direct writer under `internal/wormhole` or `internal/comms` routing; Comms Core remains the local record writer.
- Focused tests exist for claim/restart recovery, crash after PostExternal, idempotent retry, transient/permanent errors, invalid UTF-8, byte-preserving valid payloads, exact-path staging, shared locking, concurrent cycles and safe push recovery. Do not redo these already-present pieces without evidence of a defect.
- Exact-head CI run 44393613006 is green on both Go 1.22 and stable, but its workflow uses `go test ./... -count=1 -timeout 300s`. It does not satisfy the required 10-second Wormhole command.

## Still missing — implement/verify these, then continue

1. **The exact required Wormhole command is not accepted.** Your latest report claims `go test ./internal/wormhole/ -count=1 -timeout 10s` exits 1 with a timeout around 10.1s, and repeated runs still exceed the limit. A green 300-second suite is not a substitute. Continue optimizing isolated Git fixtures/setup or run on a sufficiently quiet runner until this exact command exits 0. Do not change the 10-second timeout or mark an environment-only explanation as completion.
2. **Provide complete, unabridged evidence for all three exact commands on one identical final HEAD:**
   `go test ./internal/wormhole/ -count=1 -timeout 10s`
   `go test ./internal/comms/ -count=1`
   `go build ./...`
   Include full stdout and stderr, exit code and elapsed duration. The current report gives output tails / log filenames rather than the complete outputs; the Comms and build success claims therefore remain unverified as acceptance evidence.
3. **Reconcile the report chronology.** The report in `mail/ChatGPT/new/w0-20261118T200000Z-laforge-w0134-acceptance-report.md` says it was created at 20:00Z but claims to have measured HEAD `127b3ed`, whose Git commit timestamp is 20:40Z. Explain the timestamp discrepancy and attach the raw logs from the actual run.
4. Recheck the final source diff against W0-134 and its focused tests. Preserve the five-argument API and the existing valid-UTF-8 byte-fidelity/rejection behavior. Do not add an agent-side poller, local second inbox, OpenCode plugin, legacy paths or configurable folder aliases.
5. Commit and push any focused test/performance fixes on this same branch. Send a checkpoint with exact HEAD/parent, changed paths, commit SHAs, the three complete command outputs, exact-head CI URL and remaining blockers.

Continue working until every W0-134 requirement and all three exact command-evidence gates pass. An acknowledgement, a 300-second CI pass, or a partial checkpoint is not completion.
