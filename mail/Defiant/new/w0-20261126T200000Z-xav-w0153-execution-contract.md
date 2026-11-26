---
id: w0-20261126T200000Z-xav-w0153-execution-contract
from: ChatGPT
to: Defiant
subject: "W0-153: binding execution contract — finish registry Stages 1–4"
type: task
task_ref: "7f540f6aaa"
created: "2026-11-26T20:00:00Z"
---
# W0-153 — execute to a reviewable, tested implementation

This is a clarification and execution contract for your EXISTING W0-153 assignment. It is not a new task and does not change ownership. You remain the sole writer on X11Libre/xserver:wip/xaverius. Do not wait for Enterprise review or another owner prompt.

## Objective (definition of done)

Deliver the complete PhysicalDevice registry Stages 1–4 from the exact normative assignment/spec, as a reviewable commit series on wip/xaverius, with exact-head CI green and evidence sufficient for independent owner review. Do not report completion on the basis of a processed message, task_progress, local test summary alone, or a partial implementation.

Authoritative documents (sync and read current versions before edits):
- metux/xav-design@docs/restructure-2026-10-21:
  - implementation-notes/active-physical-device-registry-task-2026-11-05.md
  - spec/device-registry-architecture.md
  - spec/registry-api-lifetime-addendum.md
  - spec/provider-operation-matrix.md
  - implementation-notes/xserver-registry-rebuild-plan-2026-11-05.md
  - implementation-notes/phase-a-registry-audit-2026-11-05.md
  - implementation-notes/phase-a-ddx-corrections.md
  - implementation-notes/ddx-device-registration.md
- Owner decision and W0-153 acceptance addenda already delivered in Wormhole, including the 2026-11-06 normative registry corrections and subsequent W0-153 clarifications.
The assignment and normative spec override generic skills and historical messages. If exact normative sources conflict, stop only at that conflict, quote both exact passages and ask owner; continue all independent work.

## First action — preserve and establish exact baseline

Before touching files, capture and report:
1. Existing Defiant session ID and current turn ID; do not create/restart a session.
2. git branch, HEAD, HEAD parent, remote branch HEAD, merge-base and ahead/behind.
3. git status --short; exact staged, unstaged, untracked paths.
4. Existing unpushed commit(s), their subjects and changed paths.
5. Confirm no other writer is operating in this worktree.

Do not reset, clean, rebase, discard, overwrite, or silently amend existing work. Preserve all valid dirty/unpushed work. Reconcile it against the current authoritative task before proceeding. If the session has died, report that fact and preserve the worktree; do not invent session recovery or claim work continues.

## Required implementation — all four stages, no scope expansion

Complete the exact Stage 1–4 acceptance in the authoritative task. At minimum the deliverable must demonstrate:

A. Canonical identity and DIX lifetime:
- Canonical PhysicalDevice identity is separate from the DIX resource wrapper.
- DIX wrapper destruction drops only its wrapper/resource reference; it does not destroy/unlink canonical registry identity or release server-owned XID/tombstone retention prematurely.
- PhysicalDevice tombstones/XIDs remain enumerable through confirmed removal until ordered server shutdown.
- Correct rollback, exactly-once destruction, ordered RegistryFini, live-list iteration safety and ref-drain/quarantine semantics.

B. Registry/source/generation semantics:
- Registry-side generation-checked source and observation bookkeeping and normative observe/withdraw/confirm-removal operations only; no udev/platform scanning, subscriptions, DDX-wide enumeration, modesetting or ScreenAttachment integration.
- Full identity evidence/provenance hierarchy, canonical ordering tuple/comparator, configured DDX binding preference using stable non-empty case-sensitive owner_key.
- No post-publication rebinding or transparent failover.
- Distinct nonzero non-reusable generations for ModuleRole, PhysicalDevice incarnation, binding, discovery source, observation and ScreenAttachment.

C. ABI and registration validation:
- Validate all three versioned descriptor headers before reading later fields, acquiring refs/leases or invoking callbacks; exact NULL/size/version result mapping and zero side effects for invalid descriptors.
- Top-level DeviceBackendOps ABI version governs all four nested operation tables and all callback, parameter, result and token types.
- Implement complete normative typed signatures for top-level and all nested operation tables. Do not leave opaque/incomplete tables, generic void dispatch, wrong handle types or by-value output handles.
- Require query_physical_snapshot for every executable binding; validate every other callback against advertised capabilities.
- Validate the complete mandatory v0.1 snapshot including timelineSemaphore=1; do not fabricate or cross-bind capabilities.

D. Ownership and tests:
- Complete ownership/reference table: each ref class, owner, acquire point, release point, protected object and failure/quarantine behavior.
- Focused tests for descriptor ABI, wrapper/canonical separation, tombstones/XID retention, rollback, stale generations, ambiguity, identity ordering, binding preference, exact replay/idempotence, snapshot completeness, capability/callback validation, ref drain and shutdown.
- Compile-time signature/layout assertions for all ABI callbacks, including every member of all four nested tables.
- Build with -Dwerror=true; focused registry and QV tests; full test suite. Record exact commands, counts and outcomes. Separate pre-existing unrelated failures by exact test name and evidence.

## Execution order and commit policy

1. Baseline/preservation report.
2. Reconcile and complete Stage 1 types/API/ABI assertions and tests.
3. Complete Stage 2 ModuleRole lifetime substrate and tests.
4. Complete Stage 3 canonical registry, registration, identity/source/generation/tombstone semantics and tests.
5. Complete Stage 4 DIX wrapper lifecycle, ordered shutdown/ref draining and tests.
6. Run focused tests after each stage; keep commits atomic/buildable. Do not stop after a checkpoint or ask for another "Weiter".
7. Run full required validation at the final exact HEAD.
8. Inspect the complete diff against the verified remote base; verify only task-authorized paths and no accidental scope creep. Push reviewable commits to the existing branch; never force-push or rewrite published history without explicit owner approval.
9. Verify the pushed SHA, PR state and exact-head CI. If no PR exists, open one only after the branch contains the complete reviewable implementation. PR body must map every acceptance criterion to source paths/tests and include exact CI run links.

## Required progress protocol

task_progress is a checkpoint, not a yield request. Continue immediately with the next concrete implementation step in the same sustained turn.

Every checkpoint must include:
- current stage and exact subtask;
- changed paths and commit SHA (or explicit dirty/uncommitted state);
- exact tests run and results;
- next concrete action, started immediately.

Do not send generic "working on it", repeat status-only messages, or stop at a microstep. Stop only when the objective is done, a real normative conflict/owner decision is required, a safety/ownership constraint blocks work, or the runtime forcibly ends. At a forced stop, publish exact HEAD, dirty paths, completed acceptance items, failed/missing tests, and the single next command/action.

## Completion report format (all fields mandatory)

- Session ID / final turn ID.
- Branch, final local HEAD, parent, remote HEAD, merge-base, ahead/behind.
- Exact commit list and changed paths.
- Stage 1/2/3/4: PASS or BLOCK, each with source paths and test names.
- ABI matrix: every top-level and nested callback signature checked; list any deviation (none expected).
- Ownership table path and summary.
- Exact build/focused/full test commands, counts, exit statuses; unrelated failures separately.
- Push result, PR URL, exact PR head SHA.
- Exact-head CI run URLs and conclusion.
- Remaining limitations, each tied to a normative clause; no vague "follow-up" claims.

A stage is PASS only when its code and regression tests exist and pass. Overall W0-153 is complete only when all four stages pass, diff review is clean, branch is pushed and exact-head CI is green.

Do not modify other ships' work, other branches, or unrelated Xserver areas. Do not implement ScreenAttachment integration, protocol handlers, Compute/Memory handlers, modesetting, DDX discovery backends, Vulkan/backend implementations, or any task outside W0-153.
