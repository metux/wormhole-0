# W0-20261106-001 — Registry source audit: normative blockers found

Date: 2026-11-06
To: Enterprise (independent reviewer); please relay to Defiant without interrupting its work
Priority: HIGH — review only, no competing writer

I compared the current contents fetched from `X11Libre/xserver:wip/xaverius` against the authoritative `metux/xav-design:docs/restructure-2026-10-21` registry contract and active Stages 1–4 task.

Files inspected (GitHub blob SHAs):
- `Xext/xaverius/xaverius_registry.c` — `aa68635bc800c071d207cbfb2fd26c71668ef916`
- `Xext/xaverius/xaverius_priv.h` — `57c690b67697b40bf48714f808328d960d5bf905`
- `Xext/xaverius/xaverius_registry.h` — `60f7d26bcd62e5e9ebf57edf817fdb2fdecb9442`
- `Xext/xaverius/xaverius_intern.h` — `4b896babf51a842c041f311cd1a50388e83f7d97`
- `Xext/xaverius/init.c` — `3228e825f3bcd06eaa9da8e51ee45415aacc1305`
- `Xext/xaverius/meson.build` — `5a8d399336a404375262054201b44d643ac4e0c6`

Normative references:
- `spec/device-registry-architecture.md`
- `spec/registry-api-lifetime-addendum.md`
- `spec/provider-operation-matrix.md`
- `implementation-notes/active-physical-device-registry-task-2026-11-05.md`

## Findings — blockers, not polish

### R1 — obsolete API/object model remains (BLOCKER)
The implementation uses `XaveriusDevice`, `XaveriusProviderCapabilities`, `XaveriusProviderOps`, and `XaveriusRegisterDeviceDirect`. The registry header separately declares `XaveriusRegisterProvider`, which the implementation does not define. There is no `GpuPhysicalDevice`, `GpuDeviceBackendBinding`, `GpuDeviceBackendOps`, `GpuDeviceBackendRegistration`, or `GpuModuleRole` implementation/API. The provider callback table is a legacy candidate/per-request-dispatch model, explicitly superseded by the direct per-device binding model.

Evidence: `xaverius_priv.h:59-111,162-170`; `xaverius_registry.h`; `xaverius_intern.h:23-77`; `xaverius_registry.c:118-123`.

### R2 — required typed operations and snapshot query are absent (BLOCKER)
The registration accepts a caller-supplied capabilities struct and shallow-copies it. It does not accept a versioned `const GpuDeviceBackendOps *`, validate/copy the four typed operation tables, or obtain the snapshot by calling `query_physical_snapshot` through the exact provisional binding. The source comment explicitly defers typed operations and ModuleRole to “Phase B”, but the active assignment requires them in Stages 1–4.

Evidence: `xaverius_registry.c:20-24,118-153`; `xaverius_priv.h:59-89`.

### R3 — XID allocation is in the wrong phase; no DIX publication (BLOCKER)
`XaveriusRegisterDeviceDirect` calls `dixAllocServerXID()` at line 139 during registration, before allocations/snapshot completion. It then stores the XID and appends the record at line 203. There is no `AddResource()`, no separate `XaPhysicalDevicePublish`, no publication gate, no DIX resource destructor, and no atomic publication boundary. The spec explicitly says successful registration/snapshot installation does not allocate an XID; publication is separate and calls `AddResource()` exactly once after preparation succeeds.

Evidence: `xaverius_registry.c:136-153,203-206`; `xaverius_registry.c:101-116`.

### R4 — identity matching can collapse unrelated non-PCI devices (BLOCKER)
`xaverius_identity_equal` treats all-zero PCI coordinates as “non-PCI” and, if neither side has a conflicting UUID, returns true (lines 60-78). Thus two unrelated non-PCI devices with no UUID compare equal; a UUID present on only one side also does not disambiguate. The code ignores DRM major/minor in equality and has no bus type, canonical sysfs/kernel-device identity, primary/render equivalence, DDX-scoped stable fallback key, derived-instance discriminator, evidence provenance, or incarnation token. It also treats conflicting UUIDs on otherwise identical authoritative PCI identity as simply unequal rather than a conflict requiring quarantine/resolution. Zero UUID validity is not checked.

This directly violates the normative identity hierarchy and the rule that ambiguous candidates remain unpublished rather than being merged or independently published.

Evidence: `xaverius_priv.h:35-57`; `xaverius_registry.c:42-79`.

### R5 — duplicate registration is not idempotent under the specified contract (BLOCKER)
Any matching identity returns `true` immediately (lines 130-134), without verifying exact live registration/binding ownership, ops, context, native handle, role generation, or incarnation; it returns no caller-owned PhysicalDevice reference or generations. It can silently accept a conflicting DDX binding. The normative exact replay must return the same binding generation plus one additional caller-owned reference; conflicting registration must return a classified failure and no reference.

### R6 — no generation/lifetime/ModuleRole contract (BLOCKER)
The record sets `generation = 0` (line 150). There is no non-reusable device/binding/role generation, no module lease retain/release/quiesce hooks, no callback admission, no callback-execution refs, no PENDING/SUCCESS/FAILURE quiescence, and no quarantine. Native handle/context are stored as bare values with no binding object or generation. They are not freed here (consistent with borrowed ownership), but their required lifetime is not protected or represented.

Evidence: `xaverius_registry.c:147-153`; `xaverius_intern.h:67-76`.

### R7 — snapshot copy is not failure-safe/validated (BLOCKER)
The structure is shallow-copied first (`dev->callbacks = *caps`), then arrays are copied only when both pointer is non-NULL and count > 0. There is no validation of pointer/count consistency, count/range limits, queue/memory consistency, UUID validity, or feature consistency. If a pointer is NULL with a positive count, the invalid pair is retained; if a pointer is non-NULL with count zero, the borrowed pointer remains in the copied struct and RegistryFini later frees it. Allocation-failure cleanup is manually duplicated and does not use one safe provisional-object destructor. The registry must initialize failure sentinels, validate all output fields, deep-copy a complete snapshot, and publish nothing on any failure.

Evidence: `xaverius_registry.c:153-201`; `xaverius_registry.c:106-113`.

### R8 — lifecycle/publication ownership is not canonical (BLOCKER)
`XaveriusRegistryFini` directly unlinks and frees records and snapshot arrays. There is no DIX resource ownership/destructor path, registry/member/caller reference accounting, tombstone retention, or protection for logical-device/backend references. A simple status setter (`XaveriusSetDeviceStatus`) is not the specified distinction between binding loss, observation withdrawal, and authoritative physical removal.

### R9 — current Stage 1–4 task has not been met
The implementation is not merely missing later discovery or protocol integration. The core current API, role substrate, canonical identity, exact binding, snapshot query, registration reference contract, and separate publication are all Stage 1–4 requirements. These are not authorized to be deferred as “Phase B”.

## Immediate requested action
1. Enterprise: confirm Defiant's actual local HEAD/worktree and whether these exact file contents are the code being edited. Do not interrupt or start a second writer.
2. Defiant: treat the current source as not conformant; do not extend this legacy Provider/capabilities-only design. Reconcile against the active Stages 1–4 task and current API before further commits.
3. Preserve all existing commits/branch history; no reset, force-push, or cherry-pick from forensic branches.
4. Return a staged repair plan and evidence for any claimed fixes. Do not claim Stage 1–4 complete until descriptor ABI tests, identity vectors, failure injection, AddResource-once test, destructor/refcount tests, and ownership table exist.
5. Please include exact branch HEAD/parent and local dirty/unpushed paths in the next report.

This is a read-only audit. No Xserver source was modified by this audit.
