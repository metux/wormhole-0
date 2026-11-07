---
id: W0-20261107-096
kind: task-followup
from: owner-coordination
to: LaForge
created: 2026-11-07T19:00:00Z
in_reply_to: W0-20261107-093
task_ref: W0-076
correlation_id: laforge-config-validation-096
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-096 — LaForge: resume with a bounded, code-level milestone

This is a NEW immutable instruction. Read it in full; do not rely on earlier task files having changed. Acknowledge this exact Message-ID with a durable correlated receipt, then execute the milestone below without waiting for another prompt.

## Corrected branch assessment

The current remote branch is:
- `work/wormhole-manager-w0-017`
- HEAD `bb2c2e3fa6c3fe6cd1e6c2269ff7271b7a4b14e3`
- master `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`
- branch is 2 commits ahead and 0 behind master.

The branch contains the substantial initial Wormhole implementation in commit `63c3e263f5f245bd52b71660cebd2e40c43297f7`; the current tip adds the runtime-directory .gitignore entry. Do not interpret the latest commit's small diff as proof that the entire branch contains no implementation. Also do not use Git author identity to attribute work: agents use the owner's Git identity.

## First milestone — make configuration loading fail safely

Inspection of the current `internal/wormhole/wormhole.go` shows concrete correctness gaps:
- `LoadConfig(root) *Config` silently returns defaults both when the file is unreadable and when parsing fails.
- `parseYAMLConfig` is a line splitter, silently ignores malformed lines and unknown keys, and silently ignores invalid duration/integer/boolean values.
- The documented nested `worker:` structure and `repositories:` configuration are not actually parsed into the current `Config`.
- `StartDaemon`, `Autostart` and `syncAllRepos` therefore cannot distinguish a missing optional config file from a malformed or unreadable one.

Implement this as one small, reviewable commit:
1. Preserve documented defaults only when the config file is genuinely absent. Return a contextual error for permission/I/O failures.
2. Parse the supported schema correctly, including nested worker settings and repository entries, or explicitly reject unsupported fields rather than silently ignoring them. Do not add a second conflicting schema if the existing `internal/config.WormholesConfig` is the intended canonical schema; inspect and align with it.
3. Validate required values, duration syntax/ranges, booleans, concurrency, forwarding/access modes, repository IDs/remotes/branches and duplicate IDs.
4. Propagate configuration errors through every caller; do not silently continue with defaults.
5. Add table-driven tests for absent config, valid config, malformed YAML, unknown keys, invalid durations/booleans/ranges, nested worker values and repository validation.

Do not broaden this first commit into a daemon rewrite. Keep compatibility deliberate and document any schema correction. Run the focused package tests and `go test ./...`; report exact commands and exit codes.

## Next milestone after this commit

Then address the already identified RunLoop/sync error-loss: `RunLoop` currently logs and discards sync/processing errors, while `syncAllRepos` and `processPendingMessages` also suppress per-repository/file/PostExternal errors. Design an explicit error/reporting path and tests; do not claim the worker is healthy merely because the loop remains alive.

## Required checkpoint

Before the first commit, publish a concise checkpoint with:
- exact branch, HEAD, merge-base and dirty-worktree status;
- the canonical config schema you chose and why;
- the callers that need signature/error propagation;
- the tests you will add.

After the commit, publish its full SHA, changed files, exact test commands/results, and the next action. Continue to the next milestone while progress is unblocked.

No restart, relaunch, detach, replacement, duplicate process, force-push, destructive reset or merge. Preserve all existing work. If an interactive Ask blocks you, report the exact request ID and prompt text; do not answer or approve it.

A receipt for this message confirms pickup only, not implementation completion.
