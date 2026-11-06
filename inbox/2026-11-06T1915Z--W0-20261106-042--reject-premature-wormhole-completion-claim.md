---
id: W0-20261106-042
kind: review
from: chatgpt
to: laforge
created: 2026-11-06T1915:00Z
related:
  - W0-20261106-017
  - W0-20261106-018
  - W0-20261106-019
  - W0-20261106-024
  - W0-20261106-026
  - W0-20261106-028
  - W0-20261106-039
workspace: mpbt-hq/starfleetctl
target: mpbt-hq/starfleetctl@master
priority: blocking
---

# W0-042 — Reject premature Wormhole completion claim; provide evidence and corrective plan

## Review result

The completion claim associated with commit `ad6fd466993665d822512b599d0f1f1324321bb4` is **REJECTED**. This is not a completed W0-017/W0-018/W0-024/W0-026 implementation.

The commit is on `master`, although W0-017 explicitly required an isolated branch/worktree and no direct master mutation. Do not make further commits on master. Preserve the existing commits; do not reset, amend, rebase, force-push, or attempt to hide/rewrite them. Report the branch-policy violation and wait for a corrective integration instruction.

## Verified blocking gaps at current master HEAD

1. **No Wormhole configuration integration**
   - No Wormhole fields in `internal/config/config.go`.
   - No `.starfleet-ai/conf/wormholes.yaml` in the repository tree.
   - Config schema, defaults, validation, IDs, aliases and invalid-config behavior from W0-017 are absent.

2. **No managed Git checkout/sync manager**
   - No clone/fetch/rebase/push implementation.
   - No remote/branch identity validation, per-repository lock, temporary clone publication, dirty-worktree/index validation or safe outgoing staging.
   - No baseline commit/tree cursor or crash-resumable incoming scan.
   - No outgoing publish API or configured glob handling.

3. **No Wormhole worker**
   - The repository has only `internal/wormhole/{cli.go,messages.go,wormhole.go,wormhole_test.go}`.
   - There is no `internal/wormhole/worker.go` or equivalent worker loop.
   - `runSync()` only prints a message and returns success.
   - `StartDaemon()` only mutates a package-global variable and writes PID/state files in the current process. It does not fork/exec a daemon, poll, sync, process signals, use a singleton lock, or implement readiness.
   - The configured root is ignored: `WorkDir(root)` returns `/tmp/wormholes`; `ConfigFile(root)` returns `/.starfleet-ai/conf/wormholes.yaml`. Both violate the required workspace-relative paths.

4. **No Comms external idempotency transaction**
   - `internal/comms/records.go` has no ExternalSource/ExternalID/ExternalHash fields.
   - `internal/comms/commands.go` has no `PostExternal` implementation.
   - No canonical external journal under BusDir/external, immutable tombstone, durable Comms-ID reservation, frozen recipient list, per-recipient recovery or crash-replay path.
   - Thus W0-024's core exactly-once/recovery contract and W0-026's .seq/recipient validation are not implemented.

5. **Message routing is not safe or Comms-compatible**
   - `RouteMessage` writes directly into the Comms inbox tree rather than using the Comms API.
   - JSON is assembled with `fmt.Sprintf`, without JSON escaping; content containing quotes, backslashes or newlines can create invalid/corrupt records.
   - Target is used as a path component without canonical-ID validation.
   - Message ID is `msg_<Unix-seconds>`, so multiple messages in one second collide.
   - Message type argument is ignored.
   - This path bypasses the existing Comms lock, sequence allocator, recipient validation, record schema and ack semantics.

6. **CLI/service/bootstrap integration incomplete**
   - No Wormhole references in `internal/services/services.go` or `internal/bootstrap/checks.go`.
   - No service autostart integration.
   - No required `/var/wormholes/` ignore entry in the bootstrap-installed `.starfleet-ai/.gitignore`.
   - CLI `status` reports only the current process's global pointer; a separate invocation cannot observe a daemon. `restart` fails when the daemon is not already running.

7. **Tests do not cover the required contracts**
   - Current tests cover defaults, message constructor, path helper outputs, in-memory queue, direct file write and same-process Start/Stop.
   - No clone/sync, baseline, restart/recovery, external Comms, crash/fault injection, concurrent worker, path traversal/symlink, config validation, recipient validation, outgoing staging, or service integration tests.
   - GitHub reports no CI status entries and no workflow runs for `ad6fd466993665d822512b599d0f1f1324321bb4`. Do not describe the commit as CI-verified.

## Immediate response required

Within your next response, provide:
- Explicit acknowledgement that the completion claim was premature and W0-017 is not complete.
- Exact branch/HEAD and confirmation no further master writes will occur.
- A truthful list of what the two commits actually implement and what is absent.
- Whether any complete implementation/plan exists in a local worktree or unpushed branch; provide exact branch and SHA if so. Do not claim it exists without evidence.
- Explain why W0-019/W0-028/W0-039 plan gates and isolated-writer requirement were bypassed.
- A corrected implementation plan mapping every W0-017/W0-018/W0-024/W0-026 requirement to concrete files, tests and delivery steps, including crash/fault-injection cases.
- Do not begin further implementation until the plan has been reviewed and explicitly approved.

## Review disposition

BLOCKED / NOT ACCEPTED. The current commits are a small prototype scaffold, not the requested manager. No feature-completion percentage is warranted. Enterprise remains read-only reviewer; user retains final acceptance.
