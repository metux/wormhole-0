# W0-20261106-003 — Reject header/protocol duplication and restore direct PhysicalDevice registration

Priority: HIGH
Target: Defiant (single writer), Enterprise (read-only reviewer)
Repository: X11Libre/xserver
Branch: wip/xaverius
Baseline for this audit: 5f018603755f0c6b2342ea175d0826a9b8f912f7

## Audit result

The current remote branch is exactly one commit ahead of the baseline (GitHub compare: ahead_by=1, behind_by=0). That one commit changes:
- modified Xext/xaverius/init.c (+21/-15)
- modified Xext/xaverius/screen.c (+16/-7)
- modified Xext/xaverius/xaverius_intern.h (+59/-8)
- modified Xext/xaverius/xaverius_priv.h (+131/-9)
- modified Xext/xaverius/meson.build (+1)
- ADDED Xext/xaverius/xaverius.h (+333)
- ADDED Xext/xaverius/xaverius_device.h (+8)
- ADDED Xext/xaverius/xaverius_registry.c (+259)
- ADDED Xext/xaverius/xaverius_registry.h (+35)

This is not a harmless formatting/refactor commit. It adds a second, incompatible protocol/type vocabulary and introduces a Provider-based registration API contrary to the current architecture.

## Confirmed blockers

### H1. xaverius.h must not exist

The newly added Xext/xaverius/xaverius.h is 333 lines and duplicates protocol declarations already owned by xaverius_proto.h:
- extension name and version
- request/subopcode constants (with a second XAVERIUS_SUBOPCODE_* namespace)
- error/event constants
- property types/status
- device types
- queue flags, command pool/buffer flags and states
- semaphore/wait/fence constants
- old server-side resource/request structs and handler declarations

It also pulls in unrelated server/DDX headers including xf86.h, xf86_OSproc.h and xf86int10.h. This is an architectural and dependency regression, not an acceptable helper header.

Required: remove xaverius.h completely. Do not rename it or split its duplicate protocol definitions into another new header. Protocol wire constants and structs remain exclusively in the agreed xaverius_proto.h.

### H2. No extra ad-hoc headers

The same commit adds xaverius_device.h (8 lines), which duplicates XAVERIUS_DEVICE_TYPE_PHYSICAL/LOGICAL already defined in xaverius.h and is not part of the agreed header set. It also adds xaverius_registry.h, which currently exposes the obsolete Provider API.

Keep exactly the three agreed headers:
- xaverius_proto.h — wire protocol declarations only
- xaverius_priv.h — DDX-facing exported/private API and its public types
- xaverius_intern.h — extension-internal declarations

Do not add xaverius.h, xaverius_device.h, xaverius_registry.h, or substitute headers. Put declarations in the appropriate one of the three existing headers, respecting their existing responsibilities. Do not duplicate declarations across them.

### H3. Direct PhysicalDevice registration was replaced by Provider terminology

Current xaverius_registry.h declares:
  bool XaveriusRegisterProvider(const XaveriusProviderCapabilities *caps,
                                const XaveriusProviderOpsRec *ops);

Current xaverius_priv.h defines XaveriusProviderCapabilities and XaveriusProviderOps and exposes:
  XaveriusRegisterDeviceDirect(identity, caps, native_handle, context)

This is still the superseded Provider/Capabilities/ProviderOps model. The comments saying “once per provider” are also wrong. The DDX registers each served PhysicalDevice directly. There is no provider object, provider candidate, provider callback-dispatch proxy, or provider registration API in the current architecture.

Required canonical model/names from the current spec:
- GpuPhysicalDevice
- GpuDeviceBackendBinding
- GpuDeviceBackendOps
- GpuDeviceBackendRegistration
- GpuModuleRole
- GpuRegistryRegisterDdxDevice()
- separate XaPhysicalDevicePublish()

Do not invent XaveriusRegisterProvider, XaveriusRegisterDeviceDirect, XaveriusProviderCapabilities, XaveriusProviderOps or provider_handle. A DDX registration call is per physical device and submits the exact backend binding for that device. ModuleRole is the lifetime/admission substrate, not a provider dispatch object.

### H4. Unnecessary identifier churn

The baseline uses xaverius_atom_device_uuid etc. The new commit renames these to xav_atom_device_uuid etc. in init.c and xaverius_intern.h, with no demonstrated semantic requirement. Restore the established names unless a concrete ABI or correctness reason is documented and approved. Do not churn identifiers merely for shorter prefixes.

Audit all changes in this commit for similar mechanical renames and revert those that are not required by the normative API. Keep changes minimal and purpose-driven.

### H5. Screen changes must remain narrowly scoped

The changed screen.c adds a ScreenPrivate accessor and changes key access style, while the stated Phase-A goal is DDX-wide PhysicalDevice inventory independent of ScreenInit. Preserve only changes required by the accepted screen-lifecycle contract. Do not expand/rework ScreenAttachment or per-screen interfaces as part of registry Stages 1–4. Screen lifecycle is not a back door for provider discovery.

### H6. Registry implementation remains non-conformant

The current xaverius_registry.c still contains the already reported blockers:
- allocates XID in registration rather than separate publication
- no AddResource()/XaPhysicalDevicePublish() boundary
- no canonical Gpu* types or typed ops
- no ModuleRole lease/quiescence substrate
- generation hard-coded to zero
- weak identity matching (all-zero non-PCI identities can collapse)
- duplicate identity returns true without exact-binding replay/ref semantics
- shallow snapshot struct copy with incomplete pointer/count validation and unsafe cleanup
- no canonical reference/lifetime/tombstone accounting

Do not build on this code by adding more phases. Implement Stages 1–4 against the current spec; the prior detailed audit W0-20261106-001 remains applicable.

## Required action sequence

1. Enterprise records the exact current remote HEAD SHA and checks Defiant's local HEAD, dirty paths and unpushed commits. Do not start a second writer and do not interrupt or overwrite Defiant's worktree.
2. Defiant explains the purpose of every file and every changed identifier in the one-commit delta. No generic “cleanup” justification.
3. Remove xaverius.h and the duplicate/ad-hoc header additions. Retain only the three agreed headers.
4. Remove Provider/Capabilities/ProviderOps vocabulary and replace it with the canonical direct PhysicalDevice + exact per-device binding API from the spec.
5. Revert unneeded identifier renames (including xav_atom_* -> established xaverius_atom_* names) and unrelated Screen changes.
6. Implement the current Stage 1–4 contract, not a local Phase-A approximation that deliberately defers typed ops/ModuleRole to Phase B.
7. Enterprise reviews the diff read-only against the normative spec and the three-header constraint before any further phase is started.

## Guardrails

- Defiant remains the only writer on wip/xaverius; Enterprise is read-only.
- No reset, force-push, blind pull, clean, merge or cherry-pick from legacy branches.
- Preserve unrelated valid commits and all uncommitted work until inventoried.
- No protocol handler implementation, discovery/hotplug integration, modesetting, ScreenAttachment integration, Compute Memory or presentation work before Stages 1–4 are reviewed and accepted.
- Do not reformat or rename unrelated files. Do not change the wire protocol.
- Report exact commit SHA, file list, tests run, and any remaining deviation.

This is a corrective addendum to W0-20261106-002 and the normative audit W0-20261106-001.