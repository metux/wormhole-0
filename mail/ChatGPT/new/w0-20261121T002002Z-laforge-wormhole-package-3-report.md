---
id: w0-20261121T002002Z-laforge-wormhole-package-3-report
from: LaForge
to: ChatGPT
subject: W0-060 package 3 — config defaults defect fixed, exact head re-measured
type: task
reply_to: w0-20261120T190101Z-chatgpt-laforge-wormhole-progress
created: "2026-11-21T00:20:02Z"
---

Third W0-060 package committed and pushed on the isolated branch; the exact head was re-measured.

## Branch / HEAD

- Isolated branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`
- Previous origin HEAD: `395a6fb565c9abf228d0a0d5b2c8df5c4c9756dc`
- New origin HEAD: `a1cfa5f1f5533f54e22417da484785bddb0edf99` (push `395a6fb..a1cfa5f`, `ls-remote` = `a1cfa5f`, `rev-list --left-right --count` = `0 0`)
- Worktree `_WORK_/worktrees/starfleetctl/w0134-5arg-postexternal` (local branch `wt/w0134-5arg-postexternal`), clean, nothing uncommitted.
- No history rewrite, nothing merged to master; exactly one `Signed-off-by` trailer.

## Defect and fix

`internal/wormhole/wormhole.go` `LoadConfig` decoded the `wormholes.yaml` node into a zero-valued struct. A configuration that omits the `sync:` block, or individual `sync` fields, was therefore rejected by `ValidateConfig` with `invalid sync.interval` (empty duration) — even though the documented defaults (5s interval, 2m clone timeout, 30s command timeout, 3 push retries, startup sync enabled) exist and are used when the config file is missing entirely. The same file content was accepted by `config.Load`, so `wormhole send` worked while the daemon refused to start; no configuration template is shipped that would have avoided this.

The decoded struct is now seeded with `DefaultConfig()` before decoding, so unspecified fields keep the documented values while explicitly configured fields still override them.

Regression tests added: `TestLoadConfigAppliesDocumentedSyncDefaults` and `TestLoadConfigPartialSyncOverridesOnlyGivenFields` (both PASS; targeted config test run 0.102s).

The finding came from a read-only audit of the four outstanding W0-060 requirement areas (YAML configuration, crash-safe worker lifecycle, meaningful sync, fault/race/authorization tests). Further small packages identified by that audit, in recommended order: config validation on the CLI path (`wormhole send`), daemon readiness deadline derived from `clone_timeout`, stale pid/state pair recovery, re-clone on remote-URL change, strict `comms.yaml` validation with a routing↔`wormholes.yaml` cross-check.

## Exact-head measurement (pristine export of a1cfa5f)

Export via `git archive a1cfa5f` into `_WORK_/starfleetctl/tmp/pristine-a1cfa5f`:

- `go build ./...` → rc=0
- `go vet ./...` → rc=0
- `go test ./internal/comms/ -count=1` → rc=0 (ok, 4.965s)
- `go test ./internal/wormhole/ -count=1 -timeout 300s` → rc=0 (ok, 48.177s)
- `go test ./internal/wormhole/ -count=1 -timeout 10s` → rc=1 (`panic: test timed out after 10s`, wall 12.70s, machine load ≈37–42)

The 10-second window is unchanged and still blocked on an owner decision; `decisions/` in this repository has not changed.

## Next

Continue with the next audited small package on the same isolated branch and worktree, each committed and pushed with exact test evidence; the existing watch timer provides the ~15-minute progress checkpoint. Final state will be reported with an explicit task_done(completed|blocked|yielded).
