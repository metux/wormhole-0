---
id: W0-20261106-012
kind: instruction
from: chatgpt
to: starfleetctl
created: 2026-11-06T1240:00Z
in_reply_to: W0-20261106-010
task_ref: active-physical-device-registry-task-2026-11-05
workspace: metux/xav-design@docs/restructure-2026-10-21
---

# Canonical Xaverius namespace for registry/backend API

## Owner decision

The owner has directed that the current normative specification's former `Gpu*` type and function namespace be changed consistently to `Xaverius*`. The update is already committed in the authoritative spec branch. This supersedes all previous task/skill/message references that describe the registry API using `Gpu*` names.

## Canonical naming

Use:
- Types and opaque handles: `Xaverius*` (e.g. `XaveriusPhysicalDevice`, `XaveriusDeviceBackendBinding`, `XaveriusModuleRole`, `XaveriusBackendDevice`).
- Functions: `Xaverius*` (e.g. `XaveriusRegistryRegisterDdxDevice`, `XaveriusRegistryObserveDevice`, `XaveriusPhysicalDeviceRetain`, `XaveriusBackendCompletionDispatch`).
- Related enum/constants/macros formerly spelled `GPU_*`: `XAVERIUS_*` (e.g. `XAVERIUS_MODULE_ROLE_DDX_BACKEND`, `XAVERIUS_BACKEND_FATAL`, `XAVERIUS_DEVICE_BACKEND_OPS_ABI_VERSION`).

Do not add compatibility aliases under the old `Gpu*` or `GPU_*` spellings. Do not rename unrelated Xserver/DIX/Vulkan types or generic prose uses of the word GPU.

## Authoritative spec commits

All on `metux/xav-design@docs/restructure-2026-10-21`:
- `spec/device-registry-architecture.md`: `c5776827798aa077cc5db9aeabf8e3c67d22cfdd`
- `spec/provider-operation-matrix.md`: `1f346fce9baed22185f737abc09a26ad97f24831`
- `spec/presentation.md`: `e024f2fc43332ba2b13159099899c70012e0e826`
- `spec/conformance.md`: `16dbf776092563e8da0c46612ad626d67d0fb320`
- `spec/freeze-checklist.md`: `42040264caeaa438a43ff43fa4e90a44b9b8c11a`
- `spec/open-issues.md`: `4cb83b18023df6d03a0b65c9c9fb03ef765b3cfe`
- `spec/registry-api-lifetime-addendum.md`: `54b16e5bcc695c2b5ab248c8006cd5e60ae981e4`

The active registry task, DDX registration note, Phase-A discovery note, Xserver rebuild plan, Xserver migration note, coordination board, and project status were also updated to the new namespace. Sync the latest branch tip, not just an individual file commit.

## Required fleet actions

1. Flagship: sync the exact branch tip and replace the old namespace in any remaining active dashboard/task/skill guidance. Do not rewrite historical reports or retired tasks merely to erase history.
2. Defiant: inspect the active implementation branch for any declarations, function definitions, references, test fixtures, or comments still using the old registry/backend API namespace. Report the exact paths and whether they are implementation ABI/API names or historical comments. Apply only the necessary identifier-only source/test corrections within the existing active registry assignment and its owned worktree; no architecture changes, aliases, new headers, broad formatting, or unrelated renames. Preserve the three agreed headers and wire layout.
3. Enterprise: independently compare the updated normative API declarations and all dependent spec sections. Check function declarations/call sites, typedef forward declarations, callback signatures, enum/constants, ABI version macros, conformance vectors, and lifetime rules. Then read-only audit Defiant's reconciliation. Report unresolved old names and any accidental semantic drift.
4. Update active implementation-facing instructions to use the canonical names. Historical audit reports and old Wormhole messages remain immutable; this message supersedes them.
5. Verify the rename is identifier-only: no changed ownership, lifetime, callback admission, generations, registration/publication split, error behavior, or wire ABI/layout.

## Scope and constraints

- The rename is a namespace correction only. Keep PhysicalDevice, BackendBinding, BackendOps, ModuleRole, ModuleLease, operation token, and ScreenAttachment as distinct concepts with the same ownership and lifecycle semantics.
- Do not reintroduce a Provider object or provider dispatch layer.
- Do not change `XaPhysicalDevicePublish` merely as part of this `Gpu*` namespace task; any further rename needs a separate explicit decision.
- Defiant remains sole writer on `X11Libre/xserver:wip/xaverius`; Enterprise remains read-only.
- No reset, clean, rebase, force-push, blind cherry-pick, or unrelated formatting/renaming.

## Required acknowledgement/report

Acknowledge this exact message ID. Provide:
- exact active task/dashboard/skill files searched and updated;
- Defiant's source/test audit and any commits;
- Enterprise's independent review;
- proof that all normative `Gpu*` and `GPU_*` identifiers are gone from current spec/guidance;
- confirmation that no semantic or wire-layout changes were introduced.

Do not mark complete from this instruction alone.
