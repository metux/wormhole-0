---
id: W0-20261107-098
kind: checkpoint
from: LaForge
to: owner-coordination
created: 2026-11-07T23:55:00Z
in_reply_to: W0-20261107-096
task_ref: W0-076
correlation_id: laforge-config-validation-096
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-096 — LaForge: pre-implementation checkpoint — config validation fail-safe

## Branch / HEAD / merge-base / dirty status

- Branch: `work/wormhole-manager-w0-017`
- HEAD: `bb2c2e3` (`wormhole: add /var/wormholes/ to gitignore`)
- Merge-base with master: `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b` (origin/master)
- Master: `0765d18` (`session: disable auto-loading CLAUDE.md`)
- Branch is 2 commits ahead, 0 behind master
- Dirty worktree: `M internal/wormhole/wormhole.go`, `M internal/wormhole/wormhole_test.go` (uncommitted changes for W0-096 first milestone)

## Canonical config schema chosen

Schema: `internal/config.WormholesConfig` (canonical, defined in `internal/config/config.go`).

Reasoning: The W0-096 task explicitly states "Do not add a second conflicting schema if the existing `internal/config.WormholesConfig` is the intended canonical schema; inspect and align with it." The canonical schema already exists and is used by `config.Load()` which reads `wormholes.yaml` under the `wormholes:` top-level key. The wormhole package's own `Config` struct (with `ListenAddr`, `AutoStart`, `ForwardMode`, etc.) was a non-canonical duplicate that silently ignored unknown keys and malformed values.

## Changes made (uncommitted, W0-096 first milestone)

### `internal/wormhole/wormhole.go`

- Replaced wormhole package's `Config` struct with `config.WormholesConfig` as the canonical configuration type.
- `DefaultConfig()` now returns `*config.WormholesConfig` with the canonical defaults (`Enabled: false`, `Sync.Interval: "5s"`, `Sync.StartupSync: true`, `Sync.CloneTimeout: "2m"`, `Sync.CommandTimeout: "30s"`, `Sync.PushRetries: 3`).
- `LoadConfig(root) (*config.WormholesConfig, error)` now:
  - Returns defaults only when the config file is genuinely absent (`os.IsNotExist`).
  - Returns a contextual error for permission/I/O failures reading the file.
  - Uses `yaml.v3` node-based decoding into `config.WormholesConfig`, aligning with `config.Load()` pattern.
  - Explicitly rejects malformed YAML (non-nil error from `yaml.Unmarshal`).
  - Calls `ValidateConfig()` and returns validation errors instead of silently continuing.
- Added `ValidateConfig(cfg *config.WormholesConfig) error` which validates:
  - `sync.interval`, `sync.clone_timeout`, `sync.command_timeout` are valid `time.ParseDuration` values.
  - `sync.push_retries` is in range 0–10.
  - Repository IDs are unique, non-empty, and consist solely of `[A-Za-z0-9_-]`.
  - `access` is `read-only` or `read-write` (empty = default read-write).
  - `incoming.initial_scan` is `baseline` or `replay` (empty = default baseline).
  - Enabled repositories require non-empty `remote` and `branch`.
  - When `Enabled: false`, all validation is skipped (defaults apply).
- Added helper functions: `validateAccessMode`, `validateInitialScan`, `validRepoID`.
- `StartDaemon`, `Autostart`, `syncAllRepos` now propagate `LoadConfig` errors instead of silently continuing with defaults.
- `RunLoop` parses `Sync.Interval` into a `time.Duration` for the ticker and respects `Sync.StartupSync` for the immediate first sync.

### `internal/wormhole/wormhole_test.go`

Rewrote tests for the new schema — 22 tests total:
- `TestDefaultConfig`: verifies canonical defaults.
- `TestLoadConfigAbsentFile`: missing file returns defaults without error.
- `TestLoadConfigValid`: full valid config with repositories parses correctly.
- `TestLoadConfigMalformedYAML`: malformed YAML returns error.
- `TestValidateConfigInvalidDuration`: invalid duration strings rejected.
- `TestValidateConfigInvalidPushRetries`: push_retries outside 0–10 rejected.
- `TestValidateConfigEmptyRepoID`: empty repository ID rejected.
- `TestValidateConfigDuplicateRepoID`: duplicate repository IDs rejected.
- `TestValidateConfigInvalidAccessMode`: invalid access mode rejected.
- `TestValidateConfigMissingRemoteForEnabledRepo`: enabled repo without remote rejected.
- `TestValidateConfigMissingBranchForEnabledRepo`: enabled repo without branch rejected.
- `TestValidateConfigInvalidCharsInRepoID`: IDs with `/` etc. rejected.
- `TestValidateConfigDisabledSkipsValidation`: disabled config skips all validation.
- Plus retained tests: `TestNewMessage`, `TestPIDFile`, `TestStateFile`, `TestConfigFile`, `TestMessageQueue`, `TestRouteMessage`, `TestStartStopDaemon`.

## Callers needing signature/error propagation

- `StartDaemon(root)` — now returns config load/validation error.
- `Autostart(root)` — now returns config load/validation error.
- `syncAllRepos(root)` — now returns config load/validation error.
- `RunLoop(root, bus)` — parses interval for ticker, respects StartupSync flag.

## Test results

`go test ./internal/wormhole/ -count=1` — PASS (22 tests, 0.019s)
`go build ./...` — PASS

## Next action

Commit these changes to `work/wormhole-manager-w0-017` and push. Then proceed to W0-096 second milestone: RunLoop/sync error-loss (errors currently logged and discarded in RunLoop, syncAllRepos, processPendingMessages).

Reply via wormhole-0 outbox if acknowledgement or questions.