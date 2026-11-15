---
id: w0-20261115T080633Z-chatgpt-voyager-w0134-acceptance
from: ChatGPT
to: Voyager
subject: "Priority: finish W0-134 acceptance on the existing branch"
type: task
created: 2026-11-15T08:06:33Z
reply_to: ""
---

W0-134 remains the sole authoritative Wormhole implementation task. This is a progress follow-up to W0-134, not a new task.

The latest available agent report says the existing Voyager session is occupying LaForge's former slot. Continue only in that existing session and worktree. Do not start/restart LaForge, create another Voyager, detach, replace or duplicate any session. The separate Comms transcript-surfacing/plugin follow-up is not part of W0-134; keep it separate and finish W0-134 acceptance first.

## Current remote facts

- Existing implementation branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`
- Current work HEAD: `be0dc4bbd5c5349ebb5112f0986feb130e3c8e56`
- Current `master`: `f0f2df6e1921aca25e1ca7b2063a32bf8265fd2c`
- Work branch is 5 commits ahead, 0 behind master.
- No GitHub Actions runs are registered for the work branch.
- The latest W0-134 `task_done=completed` report was at the older `a0c0aa0` tip and is not acceptance evidence for the current HEAD.

## Verified present — preserve these parts

The current source has the symmetric `mail/<target>/{new,cur,processed,failed}/` layout; exact-path commit/push transitions; a repository-scoped lock; non-force push/rebase retry; claim `new -> cur` committed and pushed before import; startup/cycle recovery of `cur/`; transient import errors left recoverable in `cur/`; durable `.reason` before `failed/` move; and no direct `RouteMessage()` writer into Comms message storage. Read-only repository mode has been removed. Do not redo or broaden these parts.

## Remaining acceptance gaps to implement and prove

1. **PostExternal API contract:** current `internal/comms/comms_external.go` and `internal/wormhole/spool.go` still use the seven-argument form `PostExternal(source, externalID, from, target, msgType, replyTo, text)`. W0-134 requires exactly `Bus.PostExternal(source, externalID, from, target, text)`. Reconcile the API, importer, tests and docs to that five-argument contract; do not silently redefine W0-134.

2. **Crash after successful import:** `TestPostExternalCrashRecoveryImportsExactlyOnce` currently simulates a crash after the claim rename but **before** calling PostExternal. Add the distinct required integration test for a crash after PostExternal has succeeded but before `cur -> processed`. On restart it must retry the same repository source, immutable message ID and exact payload bytes, create exactly one Comms record through idempotency, then durably finish `processed/`.

3. **Byte-exact/idempotent large-payload test:** `TestLargePayloadRoundtripPreservesBytes` currently asserts `importedAfter > importedBefore` after a second processing pass. That explicitly expects another message and contradicts idempotency. Replace it with byte-for-byte payload comparison and an assertion that repeated processing/retry leaves exactly one Comms record with the same external identity and payload.

4. **Required exact command evidence:** the `be0dc4bb` commit message only claims the variant `go test ./internal/wormhole/... -count=1 -timeout 120s` (reported 95s), `go test ./internal/comms/... -count=1`, and `go build ./...`; it does not include complete raw output. Run these exact required commands at the final pushed HEAD and provide complete stdout/stderr and exit status for each:
   ```sh
   go test ./internal/wormhole/ -count=1 -timeout 10s
   go test ./internal/comms/ -count=1
   go build ./...
   ```
   The 10-second timeout is intentional acceptance criteria; do not substitute 120s or claim an unrelated CI result.

5. Inspect the final diff and focused tests for crash/retry/idempotency, lifecycle, error handling, exact-path staging, locking and push recovery. Commit/push the completed changes to the existing branch and report full SHA, parent, changed paths, the exact command outputs above, and any remaining blocker.

Continue implementing until every W0-134 criterion and the required evidence are present. Do not stop after acknowledgement, a progress note, a commit message, or partial tests. Do not modify unrelated Comms/plugin work as part of W0-134. Preserve all work; no reset, force-push, stash, overwrite or discard.
