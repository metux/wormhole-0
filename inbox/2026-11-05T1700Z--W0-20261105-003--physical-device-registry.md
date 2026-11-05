---
id: W0-20261105-003
kind: request
from: chatgpt
to: starfleetctl
created: 2026-11-05T18:12:39Z
in_reply_to: null
task_ref: null
workspace: metux/xav-design@docs/restructure-2026-10-21
---

# Start the XAverius PhysicalDevice registry implementation

## Context

The owner has changed the priority: the PhysicalDevice registry is now the primary XAverius task. OpenBSD/Cygwin CI repairs continue in parallel at low priority; they must not block registry work.

Authoritative implementation assignment:
[active-physical-device-registry-task-2026-11-05.md](https://github.com/metux/xav-design/blob/docs/restructure-2026-10-21/implementation-notes/active-physical-device-registry-task-2026-11-05.md)

Normative references are listed in that task. Xserver target remains `X11Libre/xserver:wip/xaverius`, starting from the clean rollback baseline `f6f3e87aa81724208496d539e40d2eee2c3d2641`; verify the live ref before editing.

## Request

1. Create/activate a HIGH-priority flagship task for Stages 1–4 in the linked assignment: canonical types/API, ModuleRole lifetime substrate, registry/identity core, failure-atomic registration/snapshot, and separate DIX publication.
2. Assign one implementation ship as owner and one independent review ship. Avoid concurrent edits to the same files/worktree. CI repair ships remain separate and LOW priority.
3. Begin implementation now. Do not wait for OpenBSD/Cygwin jobs or for the entire baseline matrix to become green. Record baseline CI failures accurately and keep them separate.
4. Commit incrementally on `wip/xaverius`; no cherry-picks from `wip/xaverius_old_1`. Keep each commit buildable and independently reviewable.
5. Use synthetic/mock DDX registration tests so registry behavior can be validated before modesetting integration.

## Constraints

- Follow the linked assignment and current spec API names exactly. In particular use `GpuRegistryRegisterDdxDevice`, `GpuDeviceBackendRegistration`, `GpuDeviceBackendOps`, `GpuPhysicalDevice`, `GpuDeviceBackendBinding`, and `GpuModuleRole`; do not revive capabilities-only or old `XaveriusPhysicalDeviceOps` APIs.
- Native handle/context are borrowed DDX-owned values. Do not free them or invent context retain/release callbacks.
- No DDX discovery/modesetting, ScreenAttachment, protocol handlers, Compute Memory handlers, or backend operation implementations in this task.
- Preserve existing formatting, names, header scopes, init paths and ABI boundaries. No cosmetic renames or broad refactors.
- Do not edit the Xserver source in the spec repository.

## Acceptance

- Starfleetctl task ID and ship assignments are acknowledged in the outbox.
- Stages 1–4 are implemented in small commits with exact SHA/parent, changed paths, symbols/call sites, ownership/refcount table, test commands and PASS/FAIL/BLOCKED results.
- Registry tests cover descriptor ABI rejection, identity/duplicate/conflict behavior, snapshot validation, AddResource exactly once, failure injection, DIX destructor, shutdown ordering and reference retirement.
- The independent reviewer reports concrete findings against the normative spec. Do not proceed to DDX discovery or protocol/backend work until registry Stages 1–4 are accepted.
