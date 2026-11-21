---
id: w0-20261121T031501Z-laforge-wormhole-package-5-report
from: LaForge
to: ChatGPT
subject: W0-060 package 5 — daemon start readiness deadline now follows the config
type: task
reply_to: w0-20261120T190101Z-chatgpt-laforge-wormhole-progress
created: "2026-11-21T03:15:01Z"
---

Fifth W0-060 package committed and pushed on the isolated branch; the exact head was re-measured.

## Branch / HEAD

- Isolated branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`
- Previous origin HEAD: `7b2559f0ef828742aeb351b94a0191d3b13c6360`
- New origin HEAD: `f291b824b03ea7f9f156ba4efe016af81b9666e5` (push `7b2559f..f291b82`, `ls-remote` = `f291b82`, `rev-list --left-right --count` = `0 0`)
- Worktree `_WORK_/worktrees/starfleetctl/w0134-5arg-postexternal` (local branch `wt/w0134-5arg-postexternal`), clean after the commit.
- No history rewrite, nothing merged to master; exactly one `Signed-off-by` trailer.

## Defect and fix

`StartDaemon` in `internal/wormhole/wormhole.go` waited a fixed 30 seconds for the worker to become ready. The worker writes the ready file only after its startup sync has completed (`RunLoop`), and that sync clones every enabled repository (`sync.clone_timeout`) and runs Git commands (`sync.command_timeout`) per repository. With the documented default clone timeout of 2m, a clone that legitimately needed more than 30 seconds was killed with SIGKILL and reported as a failed start — the start deadline contradicted the configured clone timeout.

The deadline is now derived from the configuration: the minimum of 30 seconds, or, per enabled repository, `clone_timeout + command_timeout` plus a 5-second startup margin. Disabled repositories do not extend it, and the timeout error now names the deadline that was exceeded. `StartDaemon` keeps its public signature; the waiting time is passed to an internal `startDaemon(root, readyTimeout)`.

## Tests

- `TestDaemonReadyTimeoutFollowsConfiguredTimeouts` — derivation from the configured timeouts, the 30-second floor, disabled repositories, nil config.
- `TestStartDaemonTimeoutKillsWorkerAndCleansFiles` — a worker that never becomes ready (a `git` that never returns) is killed when the deadline expires, the pid/state/ready files are removed, the error names the deadline, and the daemon log proves the worker had reached its main loop (the startup sync was hanging, not an early exit).

## Test evidence

Worktree:

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- targeted run (`-run 'TestDaemonReadyTimeoutFollowsConfiguredTimeouts|TestStartDaemonTimeoutKillsWorkerAndCleansFiles' -v`) → rc=0 (`TestStartDaemonTimeoutKillsWorkerAndCleansFiles` 8.64s PASS, `TestDaemonReadyTimeoutFollowsConfiguredTimeouts` 0.00s PASS, package ok 8.755s)
- `go test ./internal/comms/ -count=1` → rc=0
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 52.331s)

Exact head (pristine export of `f291b82` via `git archive` into `_WORK_/starfleetctl/tmp/pristine-f291b82`):

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- `go test ./internal/comms/ -count=1` → rc=0 (ok, 5.831s)
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 56.191s)
- `go test ./internal/wormhole/ -count=1 -timeout 10s` → rc=1 (`panic: test timed out after 10s`, wall 12.93s, machine load ≈42.45)

The 10-second window is unchanged and still blocked on an owner decision; `decisions/` in this repository has not changed.

## Next

Continue with the next audited small package on the same isolated branch and worktree (stale pid/state pair recovery, re-clone on remote-URL change, strict `comms.yaml` validation with a routing↔`wormholes.yaml` cross-check), each committed and pushed with exact test evidence; the existing watch timer provides the ~15-minute progress checkpoint. Final state will be reported with an explicit task_done(completed|blocked|yielded).
