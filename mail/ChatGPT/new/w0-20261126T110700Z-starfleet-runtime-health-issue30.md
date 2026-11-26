---
id: w0-20261126T110700Z-starfleet-runtime-health-issue30
from: ChatGPT
to: ChatGPT
subject: Starfleetctl runtime and model health tracking — task #30
type: task
reply_to: ""
created: "2026-11-26T11:07:00Z"
---
Wormhole task handoff

- task_id: starfleetctl-issue-30
- status: acknowledged
- canonical_task: https://github.com/mpbt-hq/starfleetctl/issues/30
- repository: mpbt-hq/starfleetctl
- target: dedicated follow-up branch/PR after reconciling PR #25 and issue #24 with current master
- scope: track ship runtime (starting/working/waiting/idle) separately from health (healthy/degraded/error/offline) and task lifecycle; derive status from verified OpenCode turn/session events; centralize transition validation and stale-event handling in Go; expose consistent CLI/dashboard state; test error/retry/recovery/crash/heartbeat expiry and event ordering
- prerequisites: verify exact deployed OpenCode and plugin SDK event contract; inspect current master, PR #25 and issue #24; avoid duplicating event-correlation/lifecycle work
- exclusions: runtime telemetry must never accept, clear, or complete tasks; no ship restart/relaunch without owner approval; no force-push
- packaging invariant: whenever a new plugin source/helper/runtime file is added, update canonical bootstrap/install/packaging manifests and regression tests in the same change; verify generated/deployed plugin tree includes all runtime files
- acceptance: see complete scope and test matrix in issue #30

This Wormhole message is an immutable handoff/reference. Starfleetctl issue #30 is the authoritative task record; this repository is not a parallel task tracker.
