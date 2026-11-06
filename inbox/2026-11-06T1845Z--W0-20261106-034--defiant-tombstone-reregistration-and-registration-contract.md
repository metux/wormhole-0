# W0-20261106-034 — Defiant: tombstone re-add crash and registration blockers

Priority: BLOCKING. Target: X11Libre/xserver:wip/xaverius. Audited HEAD: 06f7d1a90131accf0e9306c18574b25717a5b861.
Normative source: metux/xav-design/docs/restructure-2026-10-21, spec/device-registry-architecture.md and implementation-notes/active-physical-device-registry-task-2026-11-05.md. This is an addendum to W0-027/W0-030/W0-033, not a replacement. Re-read current normative text before editing.

## 1. Tombstone re-registration crash
registry.c:xaverius_find_device_by_identity() searches both live devices and tombstones. RegisterDdxDevice() treats any match as a live exact replay and dereferences existing->binding->binding_generation. A removed tombstone has no live binding: same-identity re-registration can dereference NULL. It also violates the remove/re-add incarnation boundary.

Separate live-incarnation lookup from tombstone history. Confirmed removal closes that incarnation permanently. Re-add with the same BDF/sysfs identity/UUID MUST create a new record and non-reusable device_generation; it MUST NOT reuse/mutate the tombstone or its XID. Delayed old source/device-generation events must remain stale. Exact replay is only the same still-live registration/binding ownership and exact binding generation; equal hardware identity alone is not replay.

Test remove -> retain tombstone -> re-add same identity: no crash, new generation and distinct record, old tombstone/XID unchanged, old-generation event rejected.

## 2. Stage 4 registration contract gaps
At audited HEAD:
- identity_evidence is cast to XaveriusDeviceIdentity without validating its size/contract.
- query_physical_snapshot is called without proving it is non-NULL.
- required callbacks and four typed operation tables are not validated against the enabled profile and advertised capabilities.
- snapshot output is not initialized/validated/deep-copied; snapshot_size is set to zero although callback receives only void **.
- ops descriptor header validation is duplicated.
- existing registration does not validate exact live binding ownership/conflicts and can dereference a missing binding.
- post-retain failure paths do not consistently balance ModuleRole references (also W0-033).

Implement the Stage 4 normative contract: validate descriptors/evidence before reading later fields or calling hooks; validate native handle/context constraints and exact live ModuleRole generation/admission. Validate mandatory callback/table entries and only capability-relevant optional entries. Initialize snapshot outputs to failure sentinels; validate mandatory properties, pointer/count pairs, bounds, overflow, ranges and feature consistency; deep-copy the complete immutable snapshot. Do not use snapshot_size=0 as a substitute for a size contract. Distinguish exact live replay from conflicting registration; conflicts return classified failure and no reference. Balance every retain on every failure. Remove duplicated validation.

## 3. Outputs and lifetime
Initialize outputs before fallible work: out_device=NULL and generation outputs=0. Implement the immediate caller-owned reference/release semantics; a no-op XaveriusPhysicalDeviceRelease() is not sufficient if the API returns a caller reference. Keep caller reference, registry membership, DIX wrapper, binding and ModuleRole references distinct. DIX destruction must not unlink/free server-lifetime live/tombstone identity. Fix Init/Fini with live sources, observations, roles, tombstones and published resources; coordinate with W0-033.

## 4. Header and tests
Remove duplicate XaveriusRegistryRemoveDeviceBackendBinding() prototype in xaverius_intern.h. Keep the three agreed headers; no xaverius.h/fourth header.

Tests: remove/re-add; stale old-generation event; exact vs conflicting replay; malformed identity/ops descriptors; missing required callback/table; snapshot callback failure and malformed counts/pointers/overflow/deep-copy; every failure after ModuleRole retain with exact retain/release balance; output sentinels on all failures; Init/Fini with live objects.

Report exact commands and exit codes, full commit SHA/parent, changed files, defect-by-defect evidence, and CI for exact SHA.

## 5. Scope
W0-027/W0-030/W0-033 remain binding. No Phase A/DDX discovery, Screen integration, protocol handlers, Vulkan backend, Stage 5+, Provider/Candidate APIs or unrelated cleanup. Work only on wip/xaverius; no reset/rebase/force-push/merge or second writer. Enterprise is read-only reviewer. Stage 1-4 acceptance remains blocked until normative requirements and tests pass.
