# W0-20261106-003 — Detailed audit: unauthorized headers, duplicated protocol definitions, and commit/report mismatch

Date: 2026-11-06
To: Enterprise (review/coordination); Defiant (single writer)
Priority: HIGH — stop extending this commit until reconciled

This is a follow-up to W0-20261106-001 and W0-20261106-002. It audits the exact current remote tip of `X11Libre/xserver:wip/xaverius`.

## 1. Exact source state

Current branch tip:
- HEAD: `3a8a9b0fc0236689daec83e9d8867a3a58b97582`
- Parent: `5f018603755f0c6b2342ea175d0826a9b8f912f7`
- Commit title: `xaverius: Stage 1 — canonical device types and dispatch API`
- Commit date: 2026-11-05 18:54:48Z
- Diff stats: 902 changed lines (863 additions, 39 deletions).

Important chronology: the files in this commit have the same blob SHAs already inspected during audit W0-20261106-001. So this is not a new commit after that audit; the previous audit's source snapshot already contained this state. The previous audit should have called out the header regression explicitly.

Relevant current blob SHAs:
- `xaverius.h`: `8da96b99ba475d1d5ad011dcc6ca0f414112877e`
- `xaverius_proto.h`: `1ba4330948db256052c786c2f468266dad7ed83e`
- `xaverius_device.h`: `5aeea9c2e354bea342821a7ca53c62fbc9ee9416`
- `xaverius_registry.h`: `60f7d26bcd62e5e9ebf57edf817fdb2fdecb9442`
- `xaverius_priv.h`: `57c690b67697b40bf48714f808328d960d5bf905`
- `xaverius_intern.h`: `4b896babf51a842c041f311cd1a50388e83f7d97`
- `xaverius_registry.c`: `aa68635bc800c071d207cbfb2fd26c71668ef916`
- `init.c`: `3228e825f3bcd06eaa9da8e51ee45415aacc1305`
- `screen.c`: `cdfb2b5456be0e07305e5247f9682218003e1a60`
- `dispatch.c`: `4a2dc8958378fa9acbe6f602a39d05d0e80f54a5`
- `meson.build`: `5a8d399336a404375262054201b44d643ac4e0c6`

## 2. Header policy violation — three fixed headers only

The agreed header set is exactly:

1. `xaverius_proto.h` — shared public X11 wire-protocol declarations and constants.
2. `xaverius_priv.h` — DDX-facing private integration API.
3. `xaverius_intern.h` — internal declarations shared between extension translation units.

No fourth or fifth extension header is authorized. In particular, `xaverius.h` MUST NOT exist. `xaverius_registry.h` was explicitly removed earlier as a redundant pass-through header; `xaverius_device.h` is also outside the agreed set.

Current tree violates this in three ways:
- `xaverius.h` was newly added in commit 3a8a9b0 (333 lines).
- `xaverius_registry.h` was reintroduced as a separate API header.
- `xaverius_device.h` was added for two constants.

Historical proof for `xaverius_registry.h`: commit `89df8ba0bc41dbb4b291b2242b9945661bb47fad` explicitly deleted the wrapper because `xaverius_intern.h` is the single shared declaration point. Reintroducing it with a competing API declaration is a regression, not a new interface need.

Required correction:
- Delete `xaverius.h`, `xaverius_registry.h`, and `xaverius_device.h`.
- Move no content wholesale from `xaverius.h`; its protocol declarations already belong in `xaverius_proto.h`, DDX declarations in `xaverius_priv.h`, and internal declarations in `xaverius_intern.h`.
- Do not create replacement headers or rename the three fixed headers.
- Keep the existing three scopes; do not broaden `xaverius_priv.h` with internal implementation state.

## 3. xaverius.h is not just redundant — it reintroduces a stale, conflicting protocol/API model

The added file is 11,075 bytes / 333 lines and mixes unrelated scopes:
- protocol subopcode/error/event/property constants;
- server-side resource and device records;
- request helper structures containing server pointers;
- many Proc* declarations;
- extension init/fini declarations;
- DDX/Xorg headers (`xf86.h`, `xf86_OSproc.h`, `xf86int10.h`, etc.).

That is exactly what the three-header separation was intended to prevent.

Macro comparison:
- `xaverius.h` defines 91 macros.
- `xaverius_proto.h` defines 314 macros.
- Three macro names collide literally: `XAVERIUS_EXTENSION_NAME`, `XAVERIUS_MAJOR_VERSION`, `XAVERIUS_MINOR_VERSION`.
- More importantly, many protocol values are duplicated under a second, stale naming scheme rather than the canonical `XAV_REQ_*` / `XAV_*` names. These are not safely interchangeable.

Concrete semantic drift:
- Old `XAVERIUS_SUBOPCODE_*` values cover the older layout (e.g. old Compute Memory requests begin at 16).
- Current `xaverius_proto.h` assigns the current request registry, including QueryVersion at 0 and the current request numbering through presentation requests 41–47. For example, current request 16 is `XAV_REQ_CREATE_DESCRIPTOR_SET_LAYOUT`, whereas the stale header calls 16 `XAVERIUS_SUBOPCODE_CREATE_BUFFER`.
- The old header uses `XAVERIUS_ERROR_BASE 100` and `XAVERIUS_EVENT_BASE 100`, whereas the current protocol header defines the current error/event IDs as zero-based registries.
- Thus the stale file is capable of teaching server/client code a different wire ABI.

The new file is not included by the current extension source files fetched at this commit. That makes it an orphan header, and the claimed build does not demonstrate that this file or its declarations compile or agree with the normative wire ABI. Do not preserve it merely because it is currently unreferenced.

## 4. xaverius_device.h and xaverius_registry.h add no justified interface boundary

`xaverius_device.h` is only 197 bytes:
- includes `xaverius_priv.h`;
- defines `XAVERIUS_DEVICE_TYPE_PHYSICAL` and `XAVERIUS_DEVICE_TYPE_LOGICAL`.

These are not a reason for a separate header. Put a needed internal definition in the existing correct scope or remove it if unused. Do not retain a header just to hold two constants.

`xaverius_registry.h` is 1,205 bytes and includes `xaverius_intern.h`, but also declares `XaveriusRegisterProvider`. The implementation does not define this function. This reintroduces the exact redundant header boundary previously removed, now with a stale and misleading declaration.

## 5. The claimed “canonical Stage 1” is not what the code implements

The commit message claims:
- a `GpuPhysicalDeviceRec` canonical internal struct;
- `XaveriusRegisterProvider` implementation;
- `AddResource` registration;
- canonical device types and dispatch API.

The actual committed source does not match those claims:

### 5.1 No canonical Gpu* model
`xaverius_intern.h` defines `struct _XaveriusDevice` with:
- XID;
- old `XaveriusDeviceIdentity`;
- old `XaveriusDeviceStatus`;
- generation;
- `XaveriusProviderCapabilities callbacks`;
- native_handle and context.

It does not define the required `GpuPhysicalDevice`, `GpuDeviceBackendBinding`, `GpuDeviceBackendOps`, `GpuDeviceBackendRegistration`, or `GpuModuleRole`. The snapshot is still named and modelled as provider capabilities. This is the old model, not Stage 1 of the active task.

### 5.2 Three competing registration declarations/names
- `xaverius_registry.h` declares `XaveriusRegisterProvider`.
- `xaverius_priv.h` and `xaverius_intern.h` declare `XaveriusRegisterDeviceDirect`.
- `xaverius_registry.c` defines `XaveriusRegisterDeviceDirect`.

The registry header therefore advertises a function that has no implementation, while the actual implementation uses the old direct-registration signature. This is a declaration/API split, not a canonical API.

### 5.3 The commit message says AddResource; the implementation does not
The actual `XaveriusRegisterDeviceDirect` calls `dixAllocServerXID()` during registration and appends the object to the in-memory list. It does not call `AddResource()` and does not implement a separate `XaPhysicalDevicePublish` path. It therefore violates the registration/publication split and the DIX resource contract.

### 5.4 The commit message claims files/call sites that are not in the diff
The commit message refers to `xaverius.c`, `xaverius_screen.c`, and `xaverius_device.c`; the actual tree/diff uses `init.c` and `screen.c`, and only adds `xaverius_registry.c` to `meson.build`. The diff does not add `xaverius_device.c` or `xaverius_screen.c`. Treat the “green full build” claim as a report requiring exact commands/logs and commit verification, not proof of the advertised implementation.

## 6. Additional unnecessary / risky churn in existing files

### init.c
- Adds `#include "xaverius_registry.h"` although the registry declarations belong in `xaverius_intern.h`.
- Renames the five existing atom globals from `xaverius_atom_*` to `xav_atom_*` without a demonstrated requirement.
- Replaces existing `dixAddAtom(...)` calls with verbose `MakeAtom(..., strlen(...), TRUE)` calls. No justification is recorded; this is unrelated API churn.
- Calls `XaveriusRegistryInit()`, which is in-scope only if it is the single canonical registry init. It does not initialize the DIX resource type or implement publication.
- The commit description mentions `xaverius_device_init()`, but this call is absent from the actual init.c diff.

### screen.c
- Changes formatting/signature layout across existing functions.
- Changes the private-key argument from `&XaveriusScreenPrivateKeyRec` to the `XaveriusScreenPrivateKey` macro.
- Replaces the old getter implementation with a new `XaveriusScreenFini` function. This is not a demonstrated dependency of Stage 1 and needs a specific lifecycle rationale/test; do not combine unrelated Screen API churn with registry type declarations.
- Preserve the established Screen API unless a normative integration requirement requires a change. ScreenAttachment is not authorized in this task.

### meson.build
Only `xaverius_registry.c` is added. No new DDX-wide inventory implementation, no canonical device module, and no test target are added. Do not claim the registry core is implemented merely because the source file is linked.

## 7. Required action before further implementation

Defiant:
1. Stop adding code on top of this commit until the header/API regression is reconciled.
2. Send the exact local/remote HEAD, dirty paths and unpushed commits requested in W0-20261106-002.
3. Provide a file-by-file keep/revert/delete plan. The plan must retain exactly the three agreed headers.
4. Delete the three unauthorized headers listed above. Do not copy their stale definitions into the fixed headers.
5. Remove the legacy Provider/Capabilities API and declarations; do not rename them cosmetically. Implement the normative Gpu* model from the active task.
6. Revert unrelated formatting/API churn in `init.c` and `screen.c), unless a concrete required dependency is documented and separately tested.
7. Correct the commit/report claims: distinguish what is actually implemented from what remains absent. Supply exact build/test commands and logs for the exact commit.
8. Keep all history; no reset, force-push, clean, or cherry-pick from forensic branches. Make focused corrective commits on top of the verified live tip.

Enterprise:
- Review this report and Defiant's proposed file-by-file plan read-only.
- Do not start a second writer or directly edit the Xserver branch.
- Confirm exactly three headers remain and that no stale protocol constants/types are copied back into them.
- Do not accept Stage 1 until the canonical Gpu* API exists and the active task's ABI tests pass.

## 8. Acceptance checklist for this specific regression

- [ ] Only `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h` remain as XAverius headers.
- [ ] `xaverius.h`, `xaverius_registry.h`, `xaverius_device.h` are absent.
- [ ] No stale `XAVERIUS_SUBOPCODE_*`, `XAVERIUS_ERROR_BASE`, `XAVERIUS_EVENT_BASE` protocol namespace remains.
- [ ] Wire constants are defined only in `xaverius_proto.h`.
- [ ] DDX API declarations are only in `xaverius_priv.h`, internal registry declarations only in `xaverius_intern.h`.
- [ ] No `XaveriusProviderOps`, `XaveriusProviderCapabilities`, `XaveriusRegisterProvider`, or `XaveriusRegisterDeviceDirect` legacy registry API remains.
- [ ] The current `Gpu*` registration/binding/ModuleRole model is declared in the correct fixed header scopes.
- [ ] No unrelated formatting or atom/API churn remains without an explicit reason.
- [ ] Build and focused tests are reported for the exact corrective commit.

Reference to the prior explicit removal of the redundant registry header:
https://github.com/X11Libre/xserver/commit/89df8ba0bc41dbb4b291b2242b9945661bb47fad

Reference to the audited commit:
https://github.com/X11Libre/xserver/commit/3a8a9b0fc0236689daec83e9d8867a3a58b97582
