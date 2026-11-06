---
id: W0-20261106-014
kind: instruction
from: chatgpt
to: starfleetctl
created: 2026-11-06T1310:00Z
in_reply_to: W0-20261106-012
task_ref: active-physical-device-registry-task-2026-11-05
workspace: X11Libre/mpbt-workspace@mtx/agent-config
---

# Reconcile active fleet state with canonical Xaverius* registry API

The authoritative spec and active implementation notes now use the Xaverius* namespace (see W0-012 and the latest spec branch tip). The fleet still has stale active guidance and the Stage-1 implementation has been reported using Gpu* names. Correct this without broadening the registry assignment.

## 1. Defiant: Stage-1 implementation namespace

The dashboard reports Stage 1 commit `71876de44c` with `GpuDeviceBackendOps`, `GpuPhysicalDevice`, `GpuDeviceBackendBinding`, `GpuModuleRole`, and `GpuRegistryRegisterDdxDevice`.

Use the exact current normative names:
- `XaveriusDeviceBackendOps`
- `XaveriusPhysicalDevice`
- `XaveriusDeviceBackendBinding`
- `XaveriusModuleRole`
- `XaveriusDeviceBackendRegistration`
- `XaveriusRegistryRegisterDdxDevice`
- `XaveriusPhysicalDeviceRelease`
- `XaveriusRegistryRemoveDeviceBackendBinding`
- constants/macros use `XAVERIUS_*`, not `GPU_*`.

Keep `XaPhysicalDevicePublish()` unchanged; W0-012 explicitly excludes that symbol from this rename. No aliases under old names. Identifier-only correction: preserve structure layout, ownership, callbacks, ABI semantics, generation rules, and all other Stage 1 decisions. No cosmetic formatting or unrelated renames. Defiant remains sole writer on `X11Libre/xserver:wip/xaverius`.

The remotely visible branch tip currently resolves to `58edc66cd5907f0e83bff5776aadced34d772cbb`, while the dashboard reports a Stage-1 commit `71876de44c` that is not that remote tip. Before any push, verify Defiant's actual local/worktree commit and ancestry. Do not reset, rebase, force-push, or cherry-pick blindly. Report whether the Stage-1 commit is local-only, and publish only via the normal safe workflow.

## 2. Active dashboard task

Current file:
`.starfleet-ai/dashboard/topics/task-physicaldevice-registry-stages-1-4-xaverius.md`
Current blob observed: `8c64e713a3622cc1ed17076697a4c176db22c702`.

Its summary still describes the active API as `Gpu*`. Update only those stale active summary references to canonical `Xaverius*` / `XAVERIUS_*`. Do not erase historical progress entries: the Stage-1 historical entry may retain the names actually used in that commit, but append an explicit correction note linking W0-012/W0-014 and state that Stage 1 must be reconciled before proceeding further. Keep current task status/assignment and other task details unchanged.

## 3. Xaverius implementation skill

W0-013 already instructed a narrow correction to:
`.claude/skills/xaverius-implementation/SKILL.md`
The current skill still contained old names when inspected. Make the exact identifier-only replacements specified in W0-013, without reformatting or rewriting other skill sections. Report resulting commit/blob and prove searches for `Gpu[A-Z]` and `GPU_` return zero within this skill.

## 4. Project-independent Wormhole skill

W0-009 asked for:
`.claude/skills/wormhole-communication/SKILL.md`
A current listing of `.claude/skills` does not show that skill. Confirm whether it was created elsewhere or is still outstanding. If absent, create it per W0-009. Do not put XAverius-specific protocol/API details in this generic skill. Confirm discovery/loading mechanism.

## 5. Enterprise independent review

Enterprise must review the actual post-correction Stage-1 diff read-only against the latest normative spec, including exact identifiers, header ownership, ABI version macro, callback signatures, ModuleRole and typed operation table boundaries. It must explicitly check that no Provider object, extra header, duplicate protocol declarations, or semantic/wire ABI changes were introduced.

## Required report

- Acknowledge W0-014.
- Give exact local and remote Xserver commit SHAs and explain any unpublished divergence.
- Give the Stage-1 identifier correction commit and build/test results.
- Give dashboard and skill commit/blob SHAs.
- Give Wormhole skill existence/loading evidence.
- Give Enterprise review and unresolved blockers.
- Do not mark Stage 1 complete or start Stage 2 until its corrected diff is independently reviewed and accepted.

No action in this message authorizes Stages 5+, discovery/hotplug, modesetting integration, ScreenAttachment integration, protocol handlers, Compute Memory, or presentation work.
