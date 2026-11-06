# W0-20261106-007 — Concrete skill-review corrections and regression examples

Priority: HIGH
To: Flagship (workspace configuration owner), Enterprise (read-only reviewer/coordinator)
Workspace: X11Libre/mpbt-workspace@mtx/agent-config
Spec: metux/xav-design@docs/restructure-2026-10-21
Implementation: X11Libre/xserver@wip/xaverius

This is a concrete supplement to W0-20261106-005. It closes specific gaps found when reviewing the currently published XAverius skill. It does not authorize Xserver implementation work and does not replace the normative specification or the active Stages 1–4 assignment.

## 1. Explicitly remove these stale examples and prescriptions

The replacement skill must not contain any operational instruction, code skeleton, example, checklist, or API suggestion that revives any of the following:

### A. Old object/API vocabulary

The following are stale examples, not acceptable shorthand or compatibility names:

- `XaveriusDevice`
- `XavCreateDevice()`
- `XavClientPriv()`
- `XaveriusDDXOps`
- `XaveriusScreenInit`
- `XaveriusRegisterProvider()`
- `XaveriusRegisterDeviceDirect()`
- `XaveriusProviderCapabilities`
- `XaveriusProviderOps`
- `provider_handle`
- `XaveriusPhysicalDeviceOps`

Do not retain these in “example” code, suggested naming patterns, skeletons, migration targets, or TODOs. If a historical term must be mentioned to explain a rejected design, label it explicitly as obsolete/forensic and state the replacement. Do not present it as an API to implement.

Use the current canonical names from the normative spec and active assignment, including `GpuPhysicalDevice`, `GpuDeviceBackendBinding`, `GpuDeviceBackendOps`, `GpuDeviceBackendRegistration`, `GpuModuleRole`, `GpuRegistryRegisterDdxDevice()`, and `XaPhysicalDevicePublish()`.

### B. Old header examples and misplaced declarations

The skill must not suggest:
- creating or including `xaverius.h`, `xaverius_device.h`, `xaverius_registry.h`, `resource_types.h`, or any replacement convenience header;
- placing duplicate wire constants, request structures, protocol enums, or wire layouts outside `xaverius_proto.h`;
- putting DDX-facing exported API/types in `xaverius_intern.h`;
- putting extension-private implementation state, DIX resource internals, or internal-only declarations in `xaverius_priv.h`;
- splitting one canonical declaration into duplicate definitions across headers.

The only approved headers and exact responsibilities are:
- `xaverius_proto.h`: wire protocol declarations only.
- `xaverius_priv.h`: DDX-facing API and types; no server-private XID/DIX wrapper internals.
- `xaverius_intern.h`: extension-internal declarations and server-private implementation details.

Do not add a fourth header to work around an inconvenient include dependency. If a genuine dependency problem appears, report it and propose the smallest correction within the approved three-header layout; wait for owner approval before adding a file or changing the layout.

### C. Old 48-request / subopcode implementation plan

Remove the old “48 requests”, “1–47 implementation phases”, and per-range request implementation schedules as active guidance. In particular, the former subopcode ranges for device/buffer/queue/submit/presentation and the old Compute handler task sequence do not authorize implementation.

Do not infer that a request is authorized because:
- its opcode/subopcode is listed in an old skill table;
- an old task says to implement subopcodes 8–15, 16–24, 25–40, or 41–47;
- a historical task is marked done;
- a handler skeleton or old branch already exists.

For the present assignment, only the PhysicalDevice registry Stages 1–4 are authorized. Protocol handlers, Compute/Memory, command/sync handlers, presentation, discovery/hotplug, modesetting, and ScreenAttachment integration remain deferred until Stages 1–4 have been reviewed and explicitly accepted and a new owner-authorized task is issued.

### D. Wrong specification path and source of truth

The path `_WORK_/xav/xav-design/spec/` is not an authority and must not appear as the location of the current specification.

The authoritative specification is the repository `metux/xav-design`, branch `docs/restructure-2026-10-21`. For the current registry task, consult the exact normative files and active assignment linked by that task. Do not invent local mirrors, use stale checkout paths as authority, or silently fall back to an old branch or remembered copy.

### E. Skill must not self-authorize architecture/API changes

Remove any wording that allows this skill to independently:
- define or revise XAverius API names, types, header ownership, ABI, lifecycle, identity, or wire protocol;
- create its own implementation phases or decide that a later phase is ready;
- expand the active task's scope;
- treat old examples, completed tasks, old branches, reports, or remembered plans as current authorization;
- update its own technical requirements as if it were a normative source.

The skill is general guidance only. Any proposed change to the architecture, API, ABI, wire protocol, header layout, or stage boundaries must be raised for owner/spec review. Update the skill only after the authoritative spec/owner decision has changed, and keep it subordinate to that decision.

## 2. Preserve the mandatory instruction precedence verbatim

The skill must include this precedence:

1. Explicit owner decisions/current coordination decisions.
2. Normative XAverius specification at `metux/xav-design:docs/restructure-2026-10-21`.
3. Current owner-authorized active implementation assignment and its explicitly linked implementation notes.
4. General repository instructions and installed skills.

It must also include this exact rule:

**The active implementation assignment and its linked instructions take precedence over this skill. The skill is subordinate general guidance, not an independent task, phase plan, API definition, or authorization.**

If the skill conflicts with the spec or active assignment, ignore the conflicting skill text and report it. If the active assignment conflicts with the normative spec, stop and request clarification. Historical tasks and completed records are not current authority.

## 3. Required minimal-change and review discipline

The skill must explicitly require:
- no gratuitous symbol, atom, file, API, or terminology renames;
- preserve established names such as `xaverius_atom_device_uuid` unless a concrete technical requirement is documented and owner-approved;
- no broad formatting churn, include sorting, whitespace-only rewrites, license-header churn, or unrelated refactoring;
- no extra source/header files without a demonstrated need and explicit approval where the three-header boundary is affected;
- keep diffs narrow and directly tied to the active assignment;
- inspect the exact current branch/commit and current file contents before acting; do not rely on stale task text or remembered branch state;
- preserve uncommitted work and report unknown/divergent state rather than resetting, cleaning, rebasing blindly, force-pushing, or cherry-picking legacy branches.

## 4. Verification checklist before declaring the skill replaced

After committing the replacement on `mtx/agent-config`, Flagship must provide:
1. Commit SHA, parent SHA, skill path, new blob SHA, and exact branch.
2. The complete final skill content or an immutable link to that exact blob.
3. A literal search report over the replacement skill for all stale tokens listed in W0-20261106-004 and section 1A–1D above. Every remaining match must be quoted with context and classified as explicitly historical/rejected; otherwise there must be zero matches.
4. Confirmation that no old API skeleton, per-Screen registration example, duplicate-header instruction, wrong spec path, or old 48-request implementation plan remains.
5. A short section-by-section check against the normative registry architecture and active Stages 1–4 assignment.
6. Confirmation that the priority rule appears in the skill and that no text gives the skill independent authority.
7. Do not claim an agent has loaded the replacement merely because the file was committed. Record the actual agent/session and loaded skill revision only after a safe, coordinated reload. Do not restart Defiant during an active write.

## 5. Scope and safety

- This is an instruction correction only. No Xserver source changes.
- Defiant remains the sole writer on `wip/xaverius`; Enterprise remains read-only there.
- Do not interrupt Defiant or alter its worktree to apply this instruction.
- Do not restart/reload Defiant until Enterprise confirms a safe checkpoint and coordinates with Defiant.
- Do not modify unrelated workspace instructions or reformat whole files.
- Keep the current registry Stages 1–4 as the only runnable XAverius implementation assignment until owner review accepts them.
- Report blockers instead of silently improvising.

This addendum supplements W0-20261106-005 and must be followed when producing and reviewing the replacement skill.
