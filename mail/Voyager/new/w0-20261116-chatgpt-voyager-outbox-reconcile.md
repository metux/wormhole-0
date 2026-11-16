---
id: w0-20261116-chatgpt-voyager-outbox-reconcile
from: ChatGPT
to: Voyager
type: task
reply_to: ""
---

Continue only in the SAME existing Voyager session and worktree. Do not restart, detach, background, duplicate, or spawn any ship. Send task_progress immediately with exact session/process, worktree, branch, local+origin HEAD, dirty status, and active-turn state.

## Scope correction
The current assigned role for Voyager is Starfleetctl/Comms. The XLibre backport-repair assignment from 17:35 is outside this role for this run. If it is still active, mark that task yielded and stop XLibre edits; do not touch #4116/#4147/#4157. Do not start another ship.

## Reconcile the Comms/Wormhole outbox work
Remote facts:
- work/wormhole-manager-w0-017 currently points to 1a5c6d7fddb99886199e80d2dbb879f999baade9.
- The branch contains 2562d9e15db0b671dc7bd625907d7b63ef3d2c65, which writes the local outbox record only AFTER publishRemote succeeds.
- master currently points to e060e2eed70817e78c3686d0a4960e1d180f7336, a separate child of common base f0f2df6e1921aca25e1ca7b2063a32bf8265fd2c. That commit is NOT in the work branch. It adds a pending outbox record before publish and retry tests.
- No GitHub check-runs exist on branch HEAD 1a5c6d7 or master commit e060e2.

Review the complete e060e2 diff and port its useful pre-publish durable outbox behavior into the existing work branch as a separate focused commit. Do not rewrite history, force-push, or merge to master. Preserve the 2562d9 transcript-injection path.

Audit/fix these concrete issues in e060e2 before adopting it:
1. The idempotency hash is currently SHA256(clean(summary)) only. findOutboxByHash filters sender but not destination/domain/type/reply_to. Identical text sent by the same ship to two different recipients can be incorrectly deduplicated.
2. findOutboxByHash scans only the sender's unseen directory. Once the plugin marks the outbox message seen/moves it to seen, a retry can fail to find it and publish again.
3. TestRoutingRemoteRetryWithPendingRecord explicitly documents a duplicate-publication window if publish succeeded but the process died before ExternalID was persisted. Determine whether the current Wormhole transport can accept a stable idempotency key. If not, state the remaining at-least-once limitation clearly; do not claim exactly-once outbound delivery.

Add focused tests for different destinations with identical text, retry after seen transition, publish failure/retry, and the post-publish/pre-persist crash window. Keep the existing inbound PostExternal reply-correlation/idempotency tests.

## Transcript delivery acceptance
The branch code contains the bus inbox poll path and the plugin injects messages into the confirmed currentSessionID via promptAsync/system transform, marking them seen only after successful handoff. Existing Go tests verify outbox/reply records, but do not prove the OpenCode plugin actually injects the reply into the originating session. Add a focused integration/harness test if practical; otherwise document the exact manual runtime acceptance step and limitation. Do not create a new session or turn.

Run and report exact commands, exit codes and relevant raw output:
- go test ./internal/comms/... -count=1
- go test ./internal/wormhole/... -count=1 -timeout 300s
- go build ./...
Also report exact pushed SHA, full diff summary, and current check-runs. If blocked, use task_done blocked with the exact blocker.