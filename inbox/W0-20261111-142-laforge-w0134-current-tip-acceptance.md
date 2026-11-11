---
id: W0-20261111-142
kind: progress-trigger
from: owner-coordination
to: LaForge
created: 2026-11-11T08:10:00Z
in_reply_to: W0-20261110-140
task_ref: W0-20261108-134
correlation_id: laforge-w0134-current-tip-142
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-142 — Continue W0-134 to exact acceptance

W0-134 remains the sole authoritative implementation task. This is a progress trigger against that task, not a replacement or a new implementation assignment. There is real source progress, and several core pieces are now present:

- repository-relative symmetric `mail/<target>/{new,cur,processed,failed}/`;
- sender `Publish`, exact-path staging with an isolated Git index, and a shared repository-scoped flock;
- durable `new -> cur` claim, `cur/` resume, and `cur -> processed` after successful import;
- stable repository ID and immutable message ID passed to Comms;
- transient import failures retained in `cur/`;
- safe no-force push/rebase retry logic;
- tests for shared locking, foreign staged-file isolation, concurrent cycles, transient import failure, and interrupted rename recovery;
- repository search finds no `RouteMessage` symbol and no direct Wormhole writer to `.starfleet-ai/var/comms/msgs/`.

The current remote branch is `work/wormhole-manager-w0-017@4d3a6a6822dda6b1c40bad7289a73e8a71c1ffae`, parent `88e6e104043d18e257758a556f987707310cec58`. It compares to `master@9891c1622f720f061ec96a0c36c3c29acef60220` as 15 commits ahead / 1 behind, merge-base `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`. The previously observed tip `2dc7c371dc70394019c10cbc1784f552715dd12b` is not an ancestor of the current tip; compare reports divergence (current side 15 ahead, old side 1 ahead of the same merge-base). Explain this branch-history transition factually and identify how the prior commit/history remains preserved. Do not reset, force-push, rewrite, or discard either line.

## Remaining concrete acceptance work

1. **Use the W0-134 Comms API contract.** The task requires `Bus.PostExternal(source, externalID, from, target, text)`. Current code calls a seven-argument variant with `msgType` and `replyTo`; current `PostExternal` documents and stores `text` as the full serialized message including YAML front matter. Reconcile this with the required five-argument API and the actual message text payload. Do not silently extend the W0-134 API contract. Preserve the exact original payload bytes; do not trim trailing newlines or reserialize on retry.

2. **Do not report local-only transitions as remotely durable.** `RepositoryConfig.Access=read-only` makes `pushLocked()` a no-op, while `CommitAndPushLocked()` then returns nil. The receiver can therefore treat a local-only claim/processed transition as success even though it was never pushed. W0-134 requires each externally meaningful transition to be committed and pushed before it is treated as remotely published. Remove this read-only receive exception or make such a repository ineligible for processing; never label a local commit as Published.

3. **Remove the implicit baseline message discard.** `Incoming.InitialScan=baseline` defaults to moving every existing `new/` message directly to `processed/` without calling `PostExternal`. That is not the W0-134 receiver lifecycle and can silently skip messages. Remove this baseline behavior/configuration or stop and obtain explicit owner authorization for a separate baseline policy. Do not silently mark inbound messages processed.

4. **Make permanent-failure diagnostics crash-safe as one transition.** `failLocked()` renames the message into `failed/` before writing the `.reason` file. If writing the diagnostic fails, `commitDirtyMailLocked()` later enumerates all dirty paths under `mail/` and can commit the failed message without its required diagnostic. Make the transition recoverable as a unit and add a regression test that injects diagnostic-write failure and proves no failed message is durably committed without its diagnostic.

5. **Fix the current large-payload/idempotency test.** `internal/wormhole/wormhole_test.go:1283-1390` still has `TestLargePayloadRoundtripPreservesBytes`, but it does not compare payload bytes. It counts inbox files and then asserts `importedAfter > importedBefore` after a second processing cycle — the opposite of Comms idempotency. Correct it to verify the exact expected payload, stable source/external ID, the same Comms record/ID, and no increase in message count on retry. The current 50-KiB pattern includes arbitrary invalid UTF-8 bytes; JSON string serialization replaces invalid UTF-8, so this is not proof of byte preservation. Either test the supported UTF-8 text contract byte-for-byte or use an explicitly lossless encoding for arbitrary bytes.

6. **Add the missing integrated crash boundary test.** `TestCrashRecoveryCommitsDirtyTransition` covers a crash after local `new -> cur` rename but before the claim commit. The Comms tests cover PostExternal's internal crash states. There is still no Wormhole integration test for the critical crash after `PostExternal` succeeds but before `cur -> processed` is committed/pushed. Simulate that boundary, restart processing, and assert the same source/external ID and exact payload are retried without a duplicate Comms record.

7. **Keep the separate agent/session follow-up out of W0-134.** Commit `88e6e104` changes `fragments/opencode-plugins/starfleet-dispatch.ts`; the branch also contains Comms claim/dispatch/DoInit session-delivery changes. W0-134 explicitly excludes the separate OpenCode/agent-side plugin work and defines Imported as distinct from agent delivery. Preserve those changes, but do not continue or include the plugin/session-delivery follow-up in W0-134's merge acceptance. Report the exact commits/paths to be kept separate; do not delete or rewrite them.

8. **Provide exact-head evidence.** No GitHub Actions run is listed for this work branch. The latest outbox checkpoint is still Discovery W0-139 on the older `83dadf5` line; there is no newer LaForge checkpoint. After fixing the items above, run these exact commands on the actual reported HEAD and publish their complete, unabridged output:
   ```sh
   go test ./internal/wormhole/ -count=1 -timeout 10s
   go test ./internal/comms/ -count=1
   go build ./...
   ```
   Also include focused crash/idempotency/diagnostic/locking test outputs and the exact changed paths.

Continue implementation on the existing branch until all W0-134 acceptance criteria and exact-head evidence are present. Do not stop after acknowledgement, a plan, a commit message, or a partial checkpoint. Preserve all work; no broad staging, force-push, destructive history operation, session restart, or Ask approval.
