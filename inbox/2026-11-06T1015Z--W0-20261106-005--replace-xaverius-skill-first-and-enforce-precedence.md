# W0-20261106-005 — Replace XAverius skill first; enforce instruction precedence

Priority: HIGH
To: Flagship (workspace configuration owner), Enterprise (reviewer/coordinator)
Workspace: X11Libre/mpbt-workspace@mtx/agent-config
Spec: metux/xav-design@docs/restructure-2026-10-21
Implementation: X11Libre/xserver@wip/xaverius

This is the immediate first action from the owner. Do not begin by changing Xserver code or by starting another implementation agent.

## 1. Replace the obsolete skill wholesale

Replace `.claude/skills/xaverius-implementation/SKILL.md` in place. Do not append a warning to the existing obsolete skill. Its current content still prescribes per-Screen XaveriusDDXOps/XaveriusScreenInit, Provider capabilities, xaverius.h, resource_types.h, xaverius_model.c, and old 1–47 request phases.

The replacement must be concise, internally consistent, and contain only:
- instruction precedence;
- current normative architecture summary;
- exactly the three agreed headers and their responsibilities;
- minimal implementation hygiene and current-revision verification guidance;
- generic Xserver coding guidance without stale code skeletons or API claims.

Use this mandatory precedence:

1. Explicit owner decisions/current coordination decisions.
2. Normative XAverius specification at `metux/xav-design:docs/restructure-2026-10-21`.
3. Current owner-authorized active implementation assignment and its explicitly linked implementation notes (scope, stage gates, target branch, concrete work).
4. General repository instructions and installed skills.

The skill MUST state plainly:

**The active implementation assignment and its linked instructions take precedence over this skill. The skill is subordinate general guidance, not an independent task, phase plan, API definition, or authorization.**

When skill conflicts with active assignment/spec: ignore the conflicting skill text and report it. When active assignment conflicts with normative spec: stop and request clarification. Old tasks, branches, reports, examples, and remembered plans are historical unless explicitly reactivated.

Current architecture summary:
- DDX directly registers every PhysicalDevice it serves; no separate Vulkan/provider inventory.
- Process-wide PhysicalDevice identity/registry is independent of X11 Screens; ScreenAttachment is separate and non-owning.
- Exact per-device `GpuDeviceBackendBinding` carries borrowed native handle/context and immutable typed `GpuDeviceBackendOps`.
- `GpuModuleRole` is shared module lifetime/admission/quiescence only, not a Provider object or dispatch proxy.
- Registration, snapshot installation and DIX/XID publication are separate; successful registration does not imply an XID.
- The canonical API names are those in the current normative spec/active task, including `GpuPhysicalDevice`, `GpuDeviceBackendBinding`, `GpuDeviceBackendOps`, `GpuDeviceBackendRegistration`, `GpuModuleRole`, `GpuRegistryRegisterDdxDevice()`, and `XaPhysicalDevicePublish()`.

Exactly three headers:
- `xaverius_proto.h`: wire protocol only.
- `xaverius_priv.h`: DDX-facing API/types.
- `xaverius_intern.h`: extension-internal declarations.

No `xaverius.h`, `xaverius_device.h`, `xaverius_registry.h`, `resource_types.h`, or other convenience/duplicate headers absent explicit owner approval. No duplicated wire defines. Preserve existing atom/symbol names absent technical necessity.

Do not authorize protocol handlers, discovery/hotplug, modesetting, ScreenAttachment integration, Compute/Memory handlers, or backend operation implementation before Stages 1–4 are reviewed and accepted.

## 2. Verify the replacement before reloading agents

- Commit the replacement on `mtx/agent-config`.
- Record commit SHA, new skill blob SHA, and exact final contents/hash.
- Search the new skill for stale tokens and old API/phase instructions; report zero actionable matches.
- Only then coordinate agent reloads. Do not restart Defiant mid-write; Enterprise must confirm a safe checkpoint and coordinate with Defiant first.
- Record which agent/session actually loaded the new skill revision. Do not claim reload based solely on a file commit.

## 3. Then fix precedence in task entry points

Update the active PhysicalDevice registry task and the live coordination board with a short explicit precedence block matching section 1. Correct the active task's “two header scopes” typo to “three agreed headers” and list their responsibilities. Mark the old rollback record in the coordination board clearly historical/forensic so it cannot be mistaken for the current branch tip.

Do not alter unrelated technical content, rename normative files, or reformat whole documents.

## 4. Then clean runnable task state

After skill replacement and precedence are committed, use the supported Starfleetctl CLI to park/supersede and unassign stale XAverius dashboard tasks identified in W0-20261106-004. Preserve history and completed records. Keep exactly one runnable high-priority XAverius implementation assignment: the current PhysicalDevice registry Stages 1–4 task, with W0-20261106-002/-003/-004/-005 as corrective instructions.

## 5. Safety and report

- No second writer on `wip/xaverius`; Defiant remains sole writer.
- No Xserver code changes in this cleanup.
- No worktree destruction, reset, force-push, blind pull, or legacy cherry-pick.
- Do not restart Defiant without safe-checkpoint coordination.
- Report exact changed paths, commit/parent SHAs, skill hash, task IDs and resulting statuses, agent/session IDs and loaded revision, plus search evidence proving no runnable instruction revives Provider/per-Screen/duplicate-header/old-subopcode behavior.
