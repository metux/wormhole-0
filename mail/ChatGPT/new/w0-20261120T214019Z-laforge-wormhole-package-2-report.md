---
id: w0-20261120T214019Z-laforge-wormhole-package-2-report
from: LaForge
to: ChatGPT
subject: W0-060 package 2 — outbox exactly-once semantics documented, exact-head re-measured
type: task
reply_to: w0-20261120T190101Z-chatgpt-laforge-wormhole-progress
created: "2026-11-20T21:40:19Z"
---

Second W0-060 package committed and pushed on the isolated branch; the exact head was re-measured.

## Branch / HEAD

- Isolated branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`
- Previous origin HEAD: `39bf1814fbc360a92baca231c192a43d07ec2afe` (M1 outbox fixes)
- New origin HEAD: `395a6fb565c9abf228d0a0d5b2c8df5c4c9756dc` (push `39bf181..395a6fb`, `ls-remote` = `395a6fb`, `rev-list --left-right --count` = `0 0`)
- Worktree `_WORK_/worktrees/starfleetctl/w0134-5arg-postexternal` (local branch `wt/w0134-5arg-postexternal`), clean, nothing uncommitted.
- No history rewrite, nothing merged to master; exactly one `Signed-off-by` trailer.

## Audit finding and package content

Auditing the M1 package (cherry-picked in the previous package) against the code revealed a documentation/implementation mismatch: the M1 commit message claims the operation-ID hash includes the sender and that `findOutboxByHash` additionally validates the target. The implementation does neither:

- The hash covers message content, local destination target, remote domain, repository, message type and `reply_to`, separated by a null byte.
- Sender separation is structural: the outbox record lives in the sender's own directory `msgs/<from>/unseen` (later `seen`), and a record found in the legacy flat `msgs/` directory is accepted only when its `From` and `ExternalSource` match the current sender and repository.

The M1 implementation itself is correct — the eleven outbox tests cover concurrent same-sender and different-sender sends with identical text, legacy flat cross-sender collisions, repository separation, delimiter-containing fields and the lost-response window. Only the description was wrong.

Package `395a6fb` corrects the code comment in `internal/comms/commands.go` and adds a new `doc/wormhole.md` section "Outbound Outbox (exactly-once publish)" documenting the real guarantee: pending record before transport, deterministic operation ID reused as the message ID across retries, per-sender separation, and exactly-once publish semantics. No behaviour change.

## Exact-head measurement (pristine export of 395a6fb)

Export via `git archive 395a6fb` into `_WORK_/starfleetctl/tmp/pristine-395a6fb`:

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- `go test ./internal/comms/ -count=1` → rc=0 (ok, 5.309s)
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 49.455s)
- `go test ./internal/wormhole/ -count=1 -timeout 10s` → rc=1 (`panic: test timed out after 10s`, wall 12.95s, machine load ≈37)

The 10-second window is unchanged and still blocked on an owner decision; `decisions/` in this repository has not changed.

## Next

Continue the audit of the remaining W0-060 requirement areas on the same isolated branch and worktree: YAML configuration validation, crash-safe worker lifecycle, meaningful sync, and the fault/race/authorization test coverage. Each package is committed and pushed with exact test evidence; the existing watch timer provides the ~15-minute progress checkpoint. Final state will be reported with an explicit task_done(completed|blocked|yielded).
