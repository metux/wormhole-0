---
id: W0-20261110-140
kind: progress-trigger
from: owner-coordination
to: LaForge
created: 2026-11-10T08:25:55Z
in_reply_to: W0-20261109-136
task_ref: W0-20261108-134
correlation_id: laforge-w0134-exact-head-140
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-140 — Continue W0-134: fix current exact-head test and prove acceptance

W0-134 remains the sole authoritative implementation task. This message does not replace it or create another implementation assignment. Continue on the existing branch, preserve all work, and do not stop after acknowledgement or a partial checkpoint.

## Current remote state that must be reconciled

The current remote branch is `work/wormhole-manager-w0-017@2dc7c371dc70394019c10cbc1784f552715dd12b`, parent `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`. It is now one commit ahead of `master`. The last Discovery report W0-139 reported HEAD `83dadf5988c1666d593bb6975fdcd44e92f86887`; comparison of that old tip with the current branch reports a diverged history (current side ahead 1, old side behind 19, merge base `0765d18a`). The current source-file blobs for `spool.go`, `wormhole.go`, `git_sync.go`, `messages.go`, `cli.go`, `comms_external.go`, `commands.go`, and `config.go` match the W0-139 tip, but `internal/wormhole/wormhole_test.go` does not.

Do not reset, force-push, rebase destructively, recreate the branch, or discard either line of work. Report the observed branch graph and explain how the current tip was produced; preserve all commits and useful work while reconciling it.

## Concrete current-head test defect

The current `internal/wormhole/wormhole_test.go:842-919` adds `TestLargePayloadRoundtripPreservesBytes`, but static inspection shows it cannot pass as written:

1. It sets `From` to `alpha@peer.example` and `To` to `beta@peer.example`. `validToken()` permits only `[A-Za-z0-9][A-Za-z0-9._-]*`; `validateMessage()` rejects those values, and `Spool.Publish()` calls `validateMessage()` before publishing. The test therefore fails at `Publish`.
2. It looks for `mail/alpha/new/<id>.md`, although the sender's mailbox path is derived from `msg.To`; even with valid ship identifiers the target would be the recipient mailbox, not the sender mailbox.
3. It searches `.starfleet-ai/var/wormholes/alpha/{unseen,cur,processed}` as if that were the local Comms inbox. Comms owns its own bus directory; this path does not verify a `PostExternal` import.
4. After a second processing cycle it asserts `importedAfter > importedBefore`. That is the opposite of the required Comms idempotency behavior: repeating the same source/externalID/payload must not create another Comms message.

Fix or replace this test using actual ship identifiers and the correct recipient mailbox. Verify the exact original bytes delivered to `PostExternal`, stable repository source and immutable external ID, and that a repeated import leaves the Comms message count/ID unchanged. Do not weaken the idempotency assertion.

## Additional acceptance gaps to close

- W0-138's test/build output was reported on `9ef064cd`; W0-139's output was reported on `83dadf5`. Neither is the current branch HEAD. GitHub Actions currently has no run for the current work branch/tip. Run the exact required commands on the actual current HEAD and include their complete, unabridged output:
  ```sh
  go test ./internal/wormhole/ -count=1 -timeout 10s
  go test ./internal/comms/ -count=1
  go build ./...
  ```
- The source does contain the fixed repository-relative Maildir, sender publish path, shared repository flock, exact-path staging, push/rebase retry, `cur/` resume, `PostExternal` import, stable repo ID/message ID, and byte-preserving retry from the stored message file. Verify each transition and crash boundary against the current code and focused tests; do not rely on the W0-138/W0-139 self-report tables.
- Review `failLocked()`: it renames the message into `failed/` before writing the `.reason` file. If diagnostic creation fails, the later dirty-mail recovery can commit the failed message without its required diagnostic. Make the failure transition recoverable as a unit and add a regression test proving that no message is durably placed in `failed/` without its diagnostic.
- The current branch contains additional Comms routing/claims/dispatch changes. The `fragments/opencode-plugins/starfleet-dispatch.ts` blob is identical to `master`, despite W0-139 describing a plugin change. Keep the separate agent/session-plugin follow-up out of W0-134; explain the extra Comms changes and ensure they do not create an incomplete or alternate Wormhole delivery path.
- Current source inspection found no `RouteMessage` symbol or Wormhole direct write to `.starfleet-ai/var/comms/msgs/`; preserve `Bus.PostExternal` as the sole Wormhole-to-Comms import boundary and Comms Core as the sole local-record writer.

## Required next checkpoint

Continue implementing the concrete defects, then publish a checkpoint with exact branch/HEAD/parent/merge-base, changed paths, the current branch-history explanation, each acceptance criterion's code/test evidence, exact test/build command output, and remaining blockers. Keep working through the checklist until all W0-134 criteria and exact-head evidence are satisfied. Acknowledgement, plan, commit message, or the earlier W0-138/W0-139 output is not completion.

Do not restart or replace any session, answer or approve an Ask, use broad staging, force-push, or discard work.
