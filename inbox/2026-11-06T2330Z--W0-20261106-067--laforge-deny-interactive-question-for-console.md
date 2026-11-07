---
id: W0-20261106-067
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T23:30:00Z
task_ref: W0-065
workspace: mpbt-hq/starfleetctl
---

# W0-067 — Small first commit: deny interactive OpenCode question for Console ships

This is the first independently reviewable slice of W0-065. No new Laforge commit is visible yet.

## Exact source facts

In `internal/session/launch.go`, `generateOpencodeConfig()` currently contains:
- For `launchType != "terminal" && !unrestricted`, it sets `permission["question"] = "deny"`.
- Its comment says terminal ships retain the default ask behavior.
- Background/auto are denied; terminal/Console are not.

The actual OpenCode permission tool key in this generator is `question`, not `ask`. Do not add an invented `ask` config key. Confirm against the pinned OpenCode schema/version and any installed plugins whether a separate tool ID exists; the existing built-in interactive prompt tool is `question`.

## Required narrow change

- For every non-unrestricted ship mode, including `terminal`/Console, generated per-ship config must set `permission["question"] = "deny"`.
- Preserve unrestricted behavior exactly as currently intended; do not broaden permissions.
- Add/adjust focused tests asserting terminal, background and auto all deny `question`; unrestricted behavior remains unchanged.
- Keep unrelated permissions unchanged.
- Do not restart, relaunch, detach, convert, or duplicate any currently running Console ship. Do not claim live policy change; generated policy applies on next normal owner-authorized launch unless a verified live-reload path exists.
- Commit this slice separately, report SHA/parent, exact test commands/results, pinned OpenCode version/schema evidence, and continue with W0-065 afterwards.

Do not wait for Enterprise.


## Owner refresh — 2026-11-07 12:53Z: claimed fix commit is comment-only

Branch head is 1e72cca3e2b5b53d95efbb18763ae2c826ddf9e2. Its entire diff to internal/session/launch.go changes comments only; executable condition still excludes terminal: if launchType != "terminal" && !unrestricted. internal/session/launch_test.go remains unchanged and expects terminal to have no question rule. Thus the commit title “fix: opencode question denied for console ships too” is not supported by its code diff. W0-067 remains open. Make the actual one-line condition correction, update focused tests, run them, and report output. Do not restart/relaunch/detach/duplicate currently running Console sessions.


## Owner closure — 2026-11-07

**CLOSED / WITHDRAWN FROM LAFORGE SCOPE.** The owner implemented, merged, and bootstrapped this fix on `mpbt-hq/starfleetctl:master` in commit `908f42c0cf7187c6a34df2b94df88d965874963e`. Do not duplicate or cherry-pick this work on `work/wormhole-manager-w0-017`. The remaining W0-069 work is unaffected.
