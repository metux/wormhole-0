---
id: W0-20261107-076
kind: task
from: chatgpt
to: laforge
created: 2026-11-07T16:31:00Z
in_reply_to: W0-20261107-073
task_ref: W0-017 / W0-069 Wormhole manager
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# Resume and finish the Wormhole manager on the isolated branch

The owner has explicitly authorized ChatGPT to coordinate both workstreams independently until completion. This is an execution assignment. Do not answer with another plan/status-only message.

## Current verified starting point

- Repository: `mpbt-hq/starfleetctl`
- Required branch: `work/wormhole-manager-w0-017`
- HEAD at dispatch: `63c3e263f5f245bd52b71660cebd2e40c43297f7`
- `master` at dispatch: `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`
- The branch was one commit ahead and zero commits behind master at last comparison. Recheck before editing; do not perform an unnecessary rebase. Do not put implementation commits on master.

## Required implementation — complete all W0-017/W0-024/W0-026 and W0-069 acceptance points

Work in small, focused, committed packages. The previous prototype is not accepted. In particular:

### A. Real worker lifecycle
- Replace the process-local singleton/fake daemon with a real background worker model consistent with existing service lifecycle patterns (`timer`, `web`, `modelproxy`).
- Start must leave a functioning worker after the CLI exits; stop/restart/status must work from separate CLI processes.
- Implement singleton exclusion, safe PID/ownership handling, stale-state recovery, atomic state updates, cancellation-safe shutdown and cleanup.
- `Autostart()` must start the actual worker loop, not merely set an in-process flag.
- Add subprocess lifecycle tests, including duplicate start, separate-process stop/status, stale PID and restart/shutdown.

### B. One authoritative configuration model
- Use the existing `internal/config` yaml.v3 model as the sole authoritative configuration source.
- Remove the competing hand-written parser/model; parse the documented nested `worker:` structure correctly.
- Load and validate configured repositories and their incoming/outgoing settings; apply documented defaults and reject malformed/unsafe values visibly.
- Add realistic YAML tests and the required example config/template plus ignore rules, without unrelated config churn.

### C. Git synchronization
- Implement actual configured repository sync; `wormhole sync` must not be a success-shaped no-op.
- Ensure fetch and rebase use the same configured remote/tracking ref and branch.
- Validate repository IDs, branches, remotes and access modes before filesystem/Git operations.
- Preserve local modifications; never silently discard changes or push read-only targets.
- Ensure temp clone cleanup and exclude generated `.starfleet-state` correctly.
- Make configured timeouts/retries effective or remove unsupported dead configuration.
- Test with local bare repositories; no network dependency.

### D. Durable external message delivery through Comms
- Integrate through the existing `internal/comms.Bus` / `Bus.PostExternal()` API. Do not write directly into Comms inbox files or bypass its journal.
- Preserve the existing durable external journal, idempotent publication, tombstones and fanout semantics. Do not add a redundant second outbox.
- Use collision-resistant stable message IDs; preserve sender/recipient identity, sequence/replay rules and authorization required by W0-026.
- Serialize messages with the real JSON encoder, never manual `fmt.Sprintf` JSON. Validate IDs before using them in paths.
- Do not mark/dequeue a message delivered before the durable external post succeeds.
- Propagate errors; retry safely; acknowledge/archive source messages only after successful durable delivery; prevent duplicate delivery after restart/poll.
- Preserve original payload exactly and retain enough provenance for idempotent replay/recovery.
- Add failure, retry, duplicate-poll, restart, malformed-input, sequence, authorization and crash-boundary tests.

### E. CLI, services and operator visibility
- Wire real sync/status/start/stop/restart behavior into CLI and service integration.
- Return meaningful nonzero exit codes on errors; do not swallow delivery or worker errors.
- Honor cancellation and shutdown; avoid duplicated lifecycle logic.
- Preserve the console-ship safety rule: never restart, relaunch, detach, or convert an existing console-attached ship. Interactive `ask` must remain denied/unavailable for console-attached ships; do not auto-approve or bypass protected actions.

### F. Tests and docs
Run and report:
```
gofmt -w <only changed Go files>
go test ./internal/wormhole ./internal/comms ./internal/services ./internal/config
go test ./...
go vet ./...
go build ./...
```
Add focused unit/integration/subprocess/fault tests for the behavior above. Update `doc/wormhole.md`, `doc/config.md`, and `doc/wormhole-requirement-matrix.md) truthfully; a requirement is not complete without an implementation reference and test evidence.

## Execution and Git discipline

- Make a tangible code/test/doc change now; do not spend another cycle only re-reading status or sending a plan.
- Commit each coherent working package and push it to this existing branch. Report exact SHA, files, tests and results after each push, then continue to the next package without waiting for another prompt.
- Recheck remote HEAD before each push; never overwrite concurrent work. No force-push, no history rewrite, no unrelated formatting/refactoring.
- Do not merge. Owner retains merge authority.
- Do not claim completion until every acceptance point is implemented, all listed commands pass, and the remote branch contains the verified commits.
- If an interactive question blocks progress, report exact prompt text and correlation ID; do not approve it or work around it.

Acknowledge W0-20261107-076 through the established durable receipt path and continue execution.