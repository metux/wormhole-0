# W0-20261106-043 — Complete Wormhole manager delivery; Enterprise continuous supervision

**Priority:** highest  
**Owners:** Laforge (implementation, single writer); Enterprise (independent supervisor/reviewer)  
**Repositories:** `mpbt-hq/starfleetctl` and `metux/wormhole-0`  
**Authoritative requirements:** W0-017, W0-018, W0-019, W0-024, W0-026, W0-028, W0-031, W0-035, W0-039 and W0-042. This task supplements and does not weaken them.

## 1. Explicit objective

Deliver the complete, production-quality Wormhole manager implementation required by W0-017, including every architecture, durability, crash-recovery, race-safety, security, configuration, CLI, daemon/service, documentation and test requirement in the referenced tasks. The current commits `8b35ffd` and `ad6fd46` on `master` are an incomplete prototype and are **not** an accepted implementation. Preserve them; do not rewrite history.

Laforge is expected to carry this work through to a complete, tested, pushed implementation and a reviewable PR without requiring the user to issue another instruction or approve each small implementation step. This does not waive the required independent review or final user acceptance.

## 2. Immediate recovery / branch discipline

1. On receipt, Laforge must acknowledge W0-042 and this task in a durable Wormhole response. State current repository, branch, HEAD SHA, worktree path, and whether any unpushed work exists.
2. Stop all writes to `mpbt-hq/starfleetctl:master`. Do not reset, amend, rebase, force-push, or otherwise rewrite the two prototype commits.
3. Create/use one dedicated isolated branch and worktree for the complete implementation, based on the current master containing those commits. Use a clear branch name such as `work/wormhole-manager-w0-017`. Laforge is the only writer to that branch/worktree. Enterprise is strictly read-only.
4. If a complete implementation already exists locally or on another branch, identify its exact branch, HEAD SHA and worktree, inspect it against every requirement below, and continue there only if it is isolated and safe. Do not silently discard or overwrite it.
5. No implementation is to be pushed to master. Deliver via a PR targeting master. Preserve all unrelated changes and established project style; no broad formatting, renames, or unrelated refactors.

## 3. Required execution sequence — no stopping after a plan

This is an execution assignment, not another request for a plan-only report.

**Step A — requirements matrix (immediately):**
- Read all authoritative tasks listed above in full, including W0-042.
- Produce a requirement-to-code-to-test matrix. Every normative requirement must map to concrete implementation files and named tests.
- Identify conflicts, ambiguities, and existing repository APIs before coding. Resolve against the stricter safety/durability requirements; if a genuine blocker cannot be resolved from the documents, report the exact question to Enterprise while continuing all independent work.
- Enterprise reviews this matrix promptly and reports omissions. Laforge incorporates corrections and proceeds; do not wait for user input.

**Step B — inspect existing code and establish the implementation branch:**
- Audit the current prototype and the existing Comms/config/CLI/service/filesystem utilities. Reuse established APIs rather than bypassing them.
- Remove/replace prototype shortcuts in the isolated branch as needed. Do not build parallel, incompatible abstractions.

**Step C — implement the complete system:**

### Configuration and paths
- Integrate validated Wormhole configuration into the real application config loader, defaults, schema and reload behavior.
- Use only `.starfleet-ai/var/wormholes/<id>/` for managed checkouts and runtime state. Do not use `/tmp/wormholes` or any global/shared fallback.
- Store configuration at `.starfleet-ai/conf/wormholes.yaml`.
- Add the required `/var/wormholes/` ignore rule to `.starfleet-ai/.gitignore` (correctly scoped so it ignores the intended runtime directory).
- Exclude runtime state and `.starfleet-state` from managed Git changes using the repository-local exclude mechanism specified in W0-018.

### Safe managed Git lifecycle
- Implement clone/fetch/rebase/push with validated remote and branch identity, constrained supported protocols, rejection of credential-bearing URLs, disabled hooks, no submodules, and secret redaction.
- Clone failure-atomically into a temporary sibling and publish by rename only after validation.
- Use per-repository locks and serialize config reload with sync.
- Validate worktree and index before any operation. Never use destructive reset/clean. Never use `git add -A`.
- Stage only explicitly approved outgoing paths; verify the exact staged diff, reject modifications/deletions of tracked incoming messages, and ensure no unrelated user changes can enter a commit.
- Keep operations safe against concurrent processes and interrupted Git operations.

### Comms integration and external import
- Implement W0-024's durable `Bus.PostExternal(source, externalID, from, target, text) (string, error)` using the actual Comms API and its lock/sequence semantics. Do not write directly into Comms message directories or hand-roll Comms JSON.
- Persist the canonical versioned external record/journal under the specified `<BusDir>/external/` location, with source, external ID, payload hash, immutable payload, from/target, durable CommsID, state, frozen recipients and per-recipient progress.
- Implement create-if-absent/idempotency semantics and durable tombstones surviving ACK/Purge. Same source+externalID with same payload must be idempotent; a different payload must fail closed.
- Implement recoverable fanout and the required state transitions. Recover safely after interruption at every persistence boundary; never duplicate a Comms message or silently lose a recipient.
- Implement W0-026 sequence recovery: recover a missing sequence only from validated messages and external records; fail closed on corrupt, empty, negative, out-of-range or unreadable state. Propagate scan/record errors; do not silently skip damaged records.
- Validate external record schema, key/hash, payload and CommsID. Unknown or malformed records must block unsafe continuation.
- Validate every target and broadcast recipient as an exact canonical safe ID (`fsutil.Safe(target)` and unchanged equality); never normalize unsafe IDs silently. Broadcast must fail closed if any recipient is invalid; sort and deduplicate recipients deterministically.

### Incoming/outgoing Wormhole messages
- Discover incoming messages and decisions using the specified baseline/replay semantics. Tie baseline to immutable initial clone commit/tree identity and make initialization crash-resumable.
- Route messages through the Comms external-import API, not direct file writes. Ensure message IDs are collision-resistant and deterministic/idempotent as required; correctly escape/encode payloads; validate all path components and message types.
- Implement outgoing message publication atomically: agents must not expose partially written mutable files to the sync glob. Define/implement the agreed staging/publish mechanism.
- Correctly track outgoing state and only publish intended messages. Ensure retries are idempotent.

### Worker daemon and CLI/service integration
- Implement a real worker lifecycle: singleton lock, PID plus process start-time/workspace identity validation, ready handshake, safe stop, signal handling, polling/sync loop, backoff and recoverable errors.
- `StartDaemon` must actually start and supervise the worker process (or use the repository's established service manager correctly); an in-process global flag or PID file alone is not a daemon.
- Implement CLI commands with accurate exit codes and useful diagnostics. A sync command must perform real sync work and report actual outcome; it must never print success without doing the work.
- Implement service/autostart/bootstrap integration as required by W0-017, including safe behavior when configuration is absent/invalid and no duplicate workers.
- Keep worker state and locks within the required per-workspace runtime directory.

### Documentation
- Update the real user/operator documentation with configuration examples, path layout, lifecycle, CLI, service behavior, recovery, security constraints, troubleshooting and operational limitations. Documentation must describe implemented behavior, not aspirations.

## 4. Mandatory test and verification suite

Add meaningful automated tests, not just constructor/path tests. At minimum cover:

- Config parsing, defaults, validation, invalid values, reload serialization.
- Path layout and root handling; path traversal, symlink and TOCTOU defenses.
- Git clone failure atomicity; dirty worktree/index; exact staging; unrelated modifications; tracked incoming message tampering; hook/submodule/remote URL restrictions; failed fetch/rebase/push and retry.
- Comms integration through the real API; durable sequence; missing-sequence recovery; corrupt sequence; external record schema/key/hash validation; idempotent duplicate; conflicting payload; frozen recipients; invalid broadcast recipient; ACK/Purge tombstone persistence.
- Fault injection/crash recovery after every journal/Comms/fanout/baseline/outgoing-publish persistence boundary. Reopen from disk and prove no loss, duplicate, or recipient drift.
- Concurrent PostExternal calls, concurrent syncs, config reload during sync, lock contention, process crash while holding/releasing locks.
- Daemon singleton, stale PID reuse defense via start time, readiness, startup failure, graceful stop, forced termination recovery, signal handling, polling/backoff.
- Incoming baseline/replay and outgoing atomic publication/retry.
- CLI and service/bootstrap integration.
- Run `go test ./...`, `go test -race ./...`, `go vet ./...`, and all repository CI/lint/build checks. Add targeted repeated/stress tests where races are plausible. Report exact commands and results. Do not claim tests passed unless actually run against the final commit.

Tests must use temporary repositories/workspaces and deterministic fault injection; no live remote or real user data is required. Include regression tests for every defect identified in the prior reviews and for the current prototype's unsafe shortcuts.

## 5. Enterprise — continuous independent supervision and active nudging

Enterprise is assigned as **active supervisor**, not merely a final reviewer:

1. Read this task and every referenced requirement. Remain read-only in `mpbt-hq/starfleetctl`; never edit Laforge's branch or worktree.
2. Immediately audit Laforge's requirement matrix and branch/worktree setup. Publish a concise pass/block report with missing mappings and concrete corrections.
3. Inspect progress at each available Wormhole check-in and every new implementation commit/PR update. Compare actual diffs and tests to the matrix; do not accept self-reported status as evidence.
4. If Laforge stops mid-task, fails to commit/push a completed bounded package, submits only a plan, repeats status without code, or leaves a requirement unaddressed, publish a direct, specific nudge in `metux/wormhole-0` identifying the exact missing deliverable and instructing him to resume immediately. Do not wait for the user to intervene. Escalate repeated inactivity or branch violations with a factual report.
5. Review each logical package promptly. Identify defects with file/line references and required tests. Laforge fixes them on his branch; Enterprise never becomes a second writer.
6. Perform a full final independent audit against every row in the matrix, inspect the complete PR diff, verify CI and test evidence, and publish explicit PASS/BLOCK for every requirement group. A green CI or Laforge's completion claim alone is insufficient.
7. Keep the supervision state visible in Wormhole: report current Laforge branch/HEAD, last concrete commit, completed groups, blockers, next exact action, and whether a nudge was sent.
8. Do not claim continuous background monitoring beyond actual check-ins/tool capabilities. The operational requirement is to inspect on every available poll/check-in and proactively nudge whenever a stall is observed.

## 6. Laforge's required working cadence

- Work in small, coherent, independently testable packages, but continue from one package to the next without waiting for another user prompt.
- For each package: implement → run targeted tests → run relevant full checks → commit → push isolated branch → verify remote SHA → post a concise status with commit SHA, files, tests and next package.
- Keep the requirement matrix current. Do not stop after producing it.
- If a test exposes a defect, fix it before moving on unless it is demonstrably independent; record any remaining blocker precisely.
- Do not report “complete”, “done”, or “ready” until every requirement and verification item above is satisfied and the complete PR is available.
- If the available execution budget/session is about to end, commit and push the current coherent state, publish exact continuation instructions and the next concrete action, then resume automatically at the next available agent turn. Never leave the user with an uncommitted, undocumented stopping point.
- No user intervention is expected for routine implementation decisions. Use the specs and existing project conventions. Ask Enterprise only for genuine normative ambiguity; escalate to the user only if a decision cannot be resolved from the authoritative requirements.

## 7. Completion / acceptance criteria

Delivery is complete only when all of the following exist:
- Full implementation on the isolated Laforge branch, not master.
- All mandatory tests and verification commands executed successfully, with exact evidence.
- Complete, accurate docs.
- A PR targeting `mpbt-hq/starfleetctl:master` containing the final implementation.
- Enterprise's independent requirement-by-requirement PASS report with no unresolved blocker.
- No history rewrite and no unrelated changes.

The user retains final acceptance and merge authority. Until then, status is **IN PROGRESS / NOT ACCEPTED**.

**First response required:** Laforge posts the recovery acknowledgement, branch/worktree/HEAD details, and requirement matrix; Enterprise posts its initial supervision audit and confirms the nudge/escalation procedure. Then implementation proceeds immediately without another user prompt.
