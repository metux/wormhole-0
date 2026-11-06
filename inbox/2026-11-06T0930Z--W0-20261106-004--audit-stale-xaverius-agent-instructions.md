# W0-20261106-004 — Audit and neutralize stale XAverius agent instructions

Priority: HIGH
To: Enterprise (coordination owner), Flagship (fleet task/SOP owner)
Target workspace: X11Libre/mpbt-workspace@mtx/agent-config
Authoritative implementation target: X11Libre/xserver@wip/xaverius
Spec authority: metux/xav-design@docs/restructure-2026-10-21

## Why this request exists

The latest Xserver commit reintroduced xaverius.h, duplicate protocol definitions, Provider/Capabilities/ProviderOps APIs and gratuitous identifier churn. A read-only audit of the fleet's actual agent instructions found that this is not an isolated coding mistake: the installed xaverius-implementation skill still explicitly instructs agents to implement the old per-Screen/Provider/48-request architecture. Several old dashboard tasks are still marked assigned and active.

Do not start another writer or overwrite Defiant's worktree. This is an instruction/configuration audit and cleanup request. Defiant remains the sole writer on wip/xaverius; Enterprise remains read-only reviewer.

## Confirmed stale instruction sources

### S1 — Installed skill is fundamentally obsolete (highest risk)

File:
Actual path: .claude/skills/xaverius-implementation/SKILL.md
Observed blob: f3b065fb0cab6efc97c2ae8a240010913d854dde

It explicitly tells agents:
- current architecture is per-Screen XaveriusDDXOps / XaveriusScreenInit;
- implement Device/Buffer/Queue/Submit/Presentation handlers through per-screen callbacks;
- create resource_types.h and xaverius_model.c;
- implement requests in the old 1–47 subopcode phase table;
- create client state in xaverius.h;
- model XaveriusDevice with generation and Provider capabilities;
- use old per-screen structure and ScreenInit/Fini as DDX integration.
It even contains contradictory internal sections: early sections say xaverius.h is a shared header and show adding declarations there; a later section says xaverius.h does not exist. It calls the three-header layout, but incorrectly assigns per-Screen XaveriusDDXOps to xaverius_priv.h and describes the old API as normative.

This skill must be replaced, not patched with another short note at the bottom. The first lines and every operational instruction must agree with current Stages 1–4.

Required skill content:
- Registry-first only; current active task Stages 1–4.
- Exact canonical API/type names: GpuPhysicalDevice, GpuDeviceBackendBinding, GpuDeviceBackendOps, GpuDeviceBackendRegistration, GpuModuleRole, GpuRegistryRegisterDdxDevice, XaPhysicalDevicePublish.
- DDX registers each served PhysicalDevice directly; no Provider object, candidate, ProviderOps, ProviderCapabilities, provider_handle, or dispatch proxy.
- Binding carries exact native handle/context and immutable typed operation tables; ModuleRole is lifetime/admission/quiescence only.
- Preserve exactly the three agreed headers: xaverius_proto.h (wire only), xaverius_priv.h (DDX-facing API/types), xaverius_intern.h (internal declarations). No xaverius.h, xaverius_device.h, xaverius_registry.h, resource_types.h or other new header unless owner explicitly approves a demonstrated need.
- No protocol handler implementation, discovery/hotplug, modesetting, ScreenAttachment, compute/memory, or backend operation implementation before Stages 1–4 accepted.
- No gratuitous renames/reformatting or broad refactors. Keep existing atom names unless a justified interface requirement says otherwise.
- Current branch safety and fleet ownership rules from W0-20261106-002/-003.
- Remove obsolete subopcode phase plan and all code skeletons referring to old API.

Restart affected agent processes only after skill replacement is committed; record the skill commit and loaded hash/version. Do not restart Defiant in the middle of an active write unless Enterprise confirms a safe checkpoint and coordinates it with Defiant.

### S2 — Old dashboard tasks remain assigned/runnable

Observed files and blob SHAs:
1. .starfleet-ai/dashboard/topics/task-xaverius-ddx-registration-migration-phases-1-3.md
   6b2b6a36d2f830b80443ac1022c24226be70decb
   Says “ProviderCapabilities -> Snapshot-only, ProviderCallbacks löschen” and still uses old phase names/legacy migration plan. This is not the canonical current API and invites Provider terminology.
2. .starfleet-ai/dashboard/topics/task-xaverius-xid-dix-resource-integration-screenattachment-phases-4-6.md
   c9ea965bf47771dff6f1e5db85f6b169f00501fe
   Old Phases 4&6; must not be runnable during Stages 1–4.
3. .starfleet-ai/dashboard/topics/task-xaverius-hotplug-unload-shutdown-request-snapshot-semantics-phases-5-7.md
   92d893a1c69114c54bc734c2773e809f2fca35c2
   Old Phases 5&7; must not be runnable now.
4. .starfleet-ai/dashboard/topics/task-xaverius-physical-device-operations-auf-inventory-api-finalisieren.md
   31a73eacf18092ec2562c5e954a2fbaaeffd6c0e
   Refers to old xaverius_inventory API, three request handlers, Screen iteration, MakeAtom and commit b55b3fe8f0. Superseded.
5. .starfleet-ai/dashboard/topics/task-xaverius-compute-baseline-handlers-subopcodes-8-15.md
   7c60665fb932499144fb681b520f495514dd99f8
   Directly authorizes Compute handlers and backend operation tables now. Explicitly out of scope until registry Stages 1–4 accepted.
6. .starfleet-ai/dashboard/topics/task-xaverius-implementation-guidance-follow-up-6-fragen-vom-design-agent.md
   1fa699a7a6a012c5378d9f535c90fd69dc692406
   Discovery/hotplug/loader/Screen lifecycle evidence questions are future follow-up, not current registry implementation work.

Use the Starfleetctl task CLI to mark all these tasks superseded/parked and unassign them. Preserve their history; do not delete completed or historical records. Ensure exactly one XAverius implementation task is runnable/high priority: active-physical-device-registry-task-2026-11-05.md plus its W0-20261105-003 handoff, with W0-20261106-002/-003/-004 as corrective addenda. Report exact dashboard task state and prove no legacy XAverius task remains runnable.

### S3 — Coordination board contains contradictory stale rollback record

In metux/xav-design@docs/restructure-2026-10-21, IMPLEMENTATION_COORDINATION.md begins with a “Rollback record — completed” claiming current wip/xaverius is at f6f3e87 and that the seven-file tree is live. This is stale and directly contradicts the active assignment below it, which correctly says the live implementation base was 5f018603... and explicitly forbids resetting to f6f3e87.

The historical audit sections are labelled forensic-only, but the opening rollback record is not clearly marked historical and may mislead agents. Rewrite it as a dated historical note or remove it from the live board; retain the current active assignment and branch-safety correction. Do not alter technical history or imply that f6 is current.

### S4 — Header-count contradiction in active implementation task

The current active task says “Preserve the established two header scopes” in Stage 1, while the user-approved implementation layout is exactly three headers:
- xaverius_proto.h
- xaverius_priv.h
- xaverius_intern.h

The same task also says “add no convenience/duplicate headers,” which is correct. Correct “two header scopes” to “three agreed headers with distinct responsibilities” and state those responsibilities. Do not let this ambiguity justify xaverius.h or any additional header.

### S5 — Historical Provider naming in normative file names/text

The file name spec/provider-operation-matrix.md is retained as a historical path and is referenced by active instructions. The current content must be treated as the typed per-device backend operation contract, not as authorization for a Provider object. Likewise GpuModuleRole may appear only as shared module lifetime/admission/quiescence control, never as a per-device provider or dispatch proxy.

Do not perform a broad cosmetic rename of normative files now; instead add a short explicit warning in the skill/task entry point and ensure all agent prompts use direct PhysicalDevice registration terminology.

## Required execution and report

1. Inventory all other XAverius-specific installed skills, agent prompts, SOPs, task templates, dashboard topics, and active ship assignments. Search for at least these stale tokens/phrases:
   XaveriusDDXOps, XaveriusScreenInit, XaveriusRegisterProvider, XaveriusRegisterDeviceDirect, XaveriusProviderCapabilities, XaveriusProviderOps, provider_handle, XaveriusPhysicalDeviceOps, xaverius.h, xaverius_device.h, xaverius_registry.h, resource_types.h, xaverius_model.c, xaverius_inventory, old subopcode phase tables, “extend the existing wip/xaverius foundation”.
2. Classify each match as: current normative; historical/forensic and clearly non-runnable; or stale/runnable. Do not blindly replace harmless historical records.
3. Replace the stale skill and park/unassign stale dashboard tasks through the supported CLI.
4. Correct the active task's two-vs-three-header wording and stale rollback text on the live coordination board.
5. Restart/reload only the agent processes that need the new skill, safely coordinated with Defiant.
6. Report exact paths, commits, task IDs/statuses, agent/session IDs and loaded skill hash. Include proof that no runnable instruction tells an agent to create xaverius.h, use Provider APIs, implement per-Screen device registration, or start protocol/Compute work before Stages 1–4 acceptance.

## Safety

- No second writer on wip/xaverius.
- No destructive edits to agent worktrees.
- No reset, force-push, blind pull, branch merge, or cherry-pick from legacy branches.
- Preserve historical reports and completed task records.
- Do not treat old implementation tasks as current merely because they are still marked assigned.
- No code changes in Xserver as part of this instruction audit.