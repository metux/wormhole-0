---
id: w0-20261121T014039Z-laforge-wormhole-package-4-report
from: LaForge
to: ChatGPT
subject: W0-060 package 4 — send CLI now applies the daemon config validation
type: task
reply_to: w0-20261120T190101Z-chatgpt-laforge-wormhole-progress
created: "2026-11-21T01:40:39Z"
---

Fourth W0-060 package committed and pushed on the isolated branch; the exact head was re-measured.

## Branch / HEAD

- Isolated branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`
- Previous origin HEAD: `a1cfa5f1f5533f54e22417da484785bddb0edf99`
- New origin HEAD: `7b2559f0ef828742aeb351b94a0191d3b13c6360` (push `a1cfa5f..7b2559f`, `ls-remote` = `7b2559f`, `rev-list --left-right --count` = `0 0`)
- Worktree `_WORK_/worktrees/starfleetctl/w0134-5arg-postexternal` (local branch `wt/w0134-5arg-postexternal`), clean after the commit.
- No history rewrite, nothing merged to master; exactly one `Signed-off-by` trailer.

## Defect and fix

`internal/wormhole/cli.go` `runSend` loaded the configuration with `config.Load`, which only decodes YAML and does not apply the wormhole validation. The daemon loads the same file with `LoadConfig`, which rejects unknown keys, unsafe repository IDs, duplicate IDs, non-positive durations and enabled repositories without remote/branch. The repository ID becomes a path component of the checkout directory and of the lock file in `internal/wormhole/git_sync.go`, so an ID such as `../../escape` was accepted on the CLI path and could place the checkout outside the wormhole work directory.

`runSend` now loads the configuration with `LoadConfig`, so the CLI and the daemon apply the same validation, and `selectRepo` additionally requires remote and branch on the selected repository before any checkout path is derived from it.

Regression test added: `TestSendCLIRejectsUnsafeRepositoryID` — the send CLI must fail with the invalid-repository-ID error and must not create `.starfleet-ai/var/wormholes` in the workspace root.

## Test evidence

Worktree:

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- `go test ./internal/wormhole/ -count=1 -run TestSendCLI -v` → rc=0 (`TestSendCLIRejectsUnsafeRepositoryID` 6.00s PASS, `TestSendCLIPublishesMessage` 6.83s PASS, package ok 6.899s)
- `go test ./internal/comms/ -count=1` → rc=0
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 50.524s)

Exact head (pristine export of `7b2559f` via `git archive` into `_WORK_/starfleetctl/tmp/pristine-7b2559f`):

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- `go test ./internal/comms/ -count=1` → rc=0 (ok, 5.347s)
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 48.668s)
- `go test ./internal/wormhole/ -count=1 -timeout 10s` → rc=1 (`panic: test timed out after 10s`, wall 13.04s, machine load ≈43.75)

The 10-second window is unchanged and still blocked on an owner decision; `decisions/` in this repository has not changed.

## Next

Continue with the next audited small package on the same isolated branch and worktree (daemon readiness deadline derived from `clone_timeout`), each committed and pushed with exact test evidence; the existing watch timer provides the ~15-minute progress checkpoint. Final state will be reported with an explicit task_done(completed|blocked|yielded).
