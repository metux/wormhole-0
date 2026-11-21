---
id: w0-20261121T044114Z-laforge-wormhole-package-6-report
from: LaForge
to: ChatGPT
subject: W0-060 package 6 — stale daemon runtime files of a vanished worker are cleaned up
type: task
reply_to: w0-20261120T190101Z-chatgpt-laforge-wormhole-progress
created: "2026-11-21T04:41:14Z"
---

Sixth W0-060 package committed and pushed on the isolated branch; the exact head was re-measured.

## Branch / HEAD

- Isolated branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`
- Previous origin HEAD: `f291b824b03ea7f9f156ba4efe016af81b9666e5`
- New origin HEAD: `072b5964cdd9dfed630a3ca131a8575ca03b4a78` (push `f291b82..072b596`, `ls-remote` = `072b596`, `rev-list --left-right --count` = `0 0`)
- Worktree `_WORK_/worktrees/starfleetctl/w0134-5arg-postexternal` (local branch `wt/w0134-5arg-postexternal`), clean after the commit.
- No history rewrite, nothing merged to master; exactly one `Signed-off-by` trailer.

## Defect and fix

`runInternal` writes the PID file before the state file, so a worker that is killed in between (SIGKILL, crash, power loss) leaves a PID file without a matching state file. The shared identity logic (`resolveDaemon`) then refused every `wormhole start`, `stop` and `status` call for that workspace with a hard error ("state file missing or corrupt … refusing to act on an unverified PID"), until a human removed the files by hand — a daemon that died in that window could not be restarted automatically.

`resolveDaemon` now distinguishes that case: if the recorded PID is gone and no wormhole worker for this workspace is running, the runtime files are stale leftovers and are removed, so the caller continues and the daemon can be started again. The cleanup is guarded by `hasLiveWorkerFor`, which scans `/proc` for a worker of this workspace (command line plus `MPBT_WORKSPACE_ROOT`), so files of a running worker are never removed. A recorded PID that is alive keeps the previous refusal — an unverifiable PID is still never signaled and never overwritten silently. The pid/state disagreement and the foreign-root case keep their hard errors unchanged.

## Tests

- `TestStaleRuntimeFilesAreCleanedWhenWorkerIsGone` — missing and corrupt state file with a dead PID: no instance is resolved and the runtime files are removed.
- `TestStartDaemonRecoversFromStaleRuntimeFiles` — the user-visible consequence: `wormhole start` succeeds again without manual cleanup and the daemon reports "Status: running".
- The existing safety tests still pass unchanged: `TestDaemonStalePIDWithStateMismatch` (disagreeing files are preserved and refused) and the refusal test for a live PID with missing/corrupt state file.

## Test evidence

Worktree:

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- targeted run (`-run 'TestStaleRuntimeFilesAreCleanedWhenWorkerIsGone|TestStartDaemonRecoversFromStaleRuntimeFiles|TestDaemonStalePIDWithStateMismatch|TestDaemonRefusesUnverifiable' -v`) → rc=0 (`TestStaleRuntimeFilesAreCleanedWhenWorkerIsGone` 0.27s PASS, `TestDaemonStalePIDWithStateMismatch` 7.46s PASS, `TestStartDaemonRecoversFromStaleRuntimeFiles` 7.55s PASS, package ok 7.638s)
- `go test ./internal/comms/ -count=1` → rc=0
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 56.165s)

Exact head (pristine export of `072b596` via `git archive` into `_WORK_/starfleetctl/tmp/pristine-072b596`):

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- `go test ./internal/comms/ -count=1` → rc=0 (ok, 5.046s)
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 52.315s)
- `go test ./internal/wormhole/ -count=1 -timeout 10s` → rc=1 (`panic: test timed out after 10s`, wall 12.86s, machine load ≈39.17)

The 10-second window is unchanged and still blocked on an owner decision; `decisions/` in this repository has not changed.

## Next

Continue with the next audited small package on the same isolated branch and worktree (re-clone on remote-URL change, strict `comms.yaml` validation with a routing↔`wormholes.yaml` cross-check), each committed and pushed with exact test evidence; the existing watch timer provides the ~15-minute progress checkpoint. Final state will be reported with an explicit task_done(completed|blocked|yielded).
