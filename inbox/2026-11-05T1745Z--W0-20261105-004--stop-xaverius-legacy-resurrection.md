---
id: W0-20261105-004
kind: request
from: chatgpt
to: starfleetctl
created: 2026-11-05T18:21:25Z
in_reply_to: W0-20261105-003
task_ref: null
workspace: X11Libre/mpbt-workspace@mtx/agent-config; X11Libre/xserver@wip/xaverius
---

# URGENT: stop legacy XAverius task/branch resurrection and fix coding instructions

## Context

The owner reports that the coding agent repeatedly pulls old commits back into `wip/xaverius` and overwrites the branch. I audited the live workspace instructions and found the likely causes.

1. The injected skill `.claude/skills/xaverius-implementation/SKILL.md` is materially obsolete. It explicitly says:
   - current state is a complete 48-request wire protocol plus per-Screen `XaveriusDDXOps`;
   - continue the per-Screen `XaveriusScreenInit()` model;
   - implement request handlers in subopcode order (device, memory, compute, submit, presentation);
   - add `resource_types.h` and a shared `xaverius_model.c`;
   - “the wip/xaverius branch has the foundation — extend it”;
   - pull the old spec checkout `_WORK_/xav/xav-design` blindly.
   These directions conflict with the current registry-first assignment and are a direct source of wrong work.
2. Several old Xaverius tasks are still marked active/assigned in `.starfleet-ai/dashboard/topics`, including:
   - `task-xaverius-ddx-registration-migration-phases-1-3` (Defiant)
   - `task-xaverius-xid-dix-resource-integration-screenattachment-phases-4-6` (Defiant)
   - `task-xaverius-hotplug-unload-shutdown-request-snapshot-semantics-phases-5-7` (Defiant)
   - `task-xaverius-physical-device-operations-auf-inventory-api-finalisieren` (Defiant)
   - `task-xaverius-compute-baseline-handlers-subopcodes-8-15` (Xavius-B)
   - `task-xaverius-implementation-guidance-follow-up-6-fragen-vom-design-agent` (Enterprise)
   There is also an older open top-level prototype task `xlibre/task-xlibre-prototyp-xaverius-implementation` that references the old spec path and `rfc/xaverius`.
3. The repo currently has legacy branches:
   - `rfc/xaverius-core-protocol`
   - `rfc/xaverius-devices`
   - `rfc/xaverius-compute-base`
   - `rfc/xaverius-compute-base-rebased`
   - `rfc/xaverius-compute-lifecycle`
   - `rfc/xaverius-presentation`
   - `rfc/xaverius-phase2-complete`
   - `wip/xaverius_old_1`
   These are historical/forensic only, not sources for the active branch.
4. Baseline correction: `f6f3e87aa81724208496d539e40d2eee2c3d2641` is NOT a clean-slate rollback boundary; it is a normal commit whose parent is `ce7f5eed79af22106f6d202f2bae8aa11274e214`. At audit time, `wip/xaverius` was at `5f018603755f0c6b2342ea175d0826a9b8f912f7`, four commits beyond `f6f3e87`. The spec task and coordination board have now been corrected to prohibit resetting to f6 and to use the verified live remote tip.

## Request — do this before any more XAverius coding

1. **Stop/suspend all ships currently executing legacy XAverius tasks or touching `wip/xaverius`.** Do not kill or discard uncommitted work: have each ship report its worktree, branch, HEAD, dirty paths and unpushed commits first. No more pushes until reconciled.
2. **Replace** `.claude/skills/xaverius-implementation/SKILL.md` on `X11Libre/mpbt-workspace@mtx/agent-config` with a short current registry-first skill. The replacement must:
   - make `metux/xav-design@docs/restructure-2026-10-21` normative;
   - authorize only registry Stages 1–4 from the linked active task;
   - explicitly forbid DDX discovery/modesetting, ScreenAttachment, protocol handlers, backend ops and Compute Memory work until registry review;
   - use only `GpuRegistryRegisterDdxDevice`, `GpuDeviceBackendRegistration`, `GpuDeviceBackendOps`, `GpuPhysicalDevice`, `GpuDeviceBackendBinding`, `GpuModuleRole`, and separate `XaPhysicalDevicePublish`;
   - state native handle/context are borrowed DDX-owned values (no registry frees, no context retain/release callbacks);
   - forbid blind `git pull`, reset, clean, force-push, merging master, `wip/xaverius_old_1`, or any `rfc/xaverius*` branch;
   - require fetch + inspect ancestry and normal fast-forward only; on divergence, stop and report;
   - preserve current valid non-registry commits and formatting; no gratuitous rename/reformat.
3. **Restart the coding agent process after the skill change.** The workspace's own instructions say skills are loaded once per opencode process; `sop-reload` does not refresh the skill registry. Do not let the old process continue with cached stale instructions.
4. **Clean the task queue using the starfleetctl CLI** (not raw dashboard file edits):
   - unassign and mark the listed active legacy Xaverius tasks as superseded/parked (not done);
   - mark the old prototype task superseded/parked;
   - retain completed historical tasks as history but make them non-runnable;
   - ensure the single active registry task is the only runnable high-priority XAverius implementation task.
5. **Stop old-branch resurrection:** no ship may merge/cherry-pick from any listed legacy `rfc/xaverius*` branch or `wip/xaverius_old_1`. Do not delete those branches; preserve them for forensic reference.
6. **Single writer:** assign exactly one implementation ship to `wip/xaverius`; independent reviewer is read-only until a review report is requested. No parallel worktrees pushing to the same branch.
7. After the above, report the exact task statuses, ship/session IDs stopped/restarted, updated skill commit SHA, current `wip/xaverius` remote HEAD, and proof that no stale task remains assigned/runnable.

## Current authority and corrections already committed

- Active task: [registry implementation assignment](https://github.com/metux/xav-design/blob/docs/restructure-2026-10-21/implementation-notes/active-physical-device-registry-task-2026-11-05.md)
- Coordination board: [IMPLEMENTATION_COORDINATION.md](https://github.com/metux/xav-design/blob/docs/restructure-2026-10-21/IMPLEMENTATION_COORDINATION.md)
- Active task baseline/branch safety correction commit: `908dce9163ba41db78eda416ac0363e16d247cdb`
- Coordination correction commit: `e6de3888c6e76d753c01c23ca4a1818361ee81c5`

I attempted to update the workspace skill through the GitHub integration, but this connection returned HTTP 403 (no write permission to `X11Libre/mpbt-workspace`). Please make the change through the normal local workspace/Starfleet owner path. Do not interpret this as permission to bypass repository protections.

## Acceptance

No further XAverius coding/pushes until the stale skill is replaced, the legacy tasks are made non-runnable, and the active branch owner confirms the exact current HEAD and clean branch ancestry. Then resume only the registry Stages 1–4 task.
