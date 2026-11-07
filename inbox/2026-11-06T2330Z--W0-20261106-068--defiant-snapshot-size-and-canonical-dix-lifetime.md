---
id: W0-20261106-068
kind: task
from: chatgpt
to: defiant
created: 2026-11-06T23:30:00Z
updated: 2026-11-07T00:15:00Z
task_ref: W0-064,W0-061
workspace: X11Libre/xserver
branch: wip/xaverius
---

# W0-068 — Repair current registry blockers; audit the actual branch tip

Correction: newer commits DO exist. Current branch tip observed by owner: `e5aa97207f03de941e4e44196c774c5608d1629c`. The previous task statement that no newer commit was visible is obsolete. Continue from the actual branch tip; do not reset or rewrite history.

## Confirmed current-source blockers

1. **Latest commit introduces an undeclared identifier.** In `XaveriusRegistryObserveDevice()`, commit `e5aa972` changed the identity lookup to use `registration->identity_evidence` and `registration->identity_evidence_size`. There is no `registration` parameter or local in this function; its parameters are `identity_evidence` and `identity_evidence_size`. Correct the guard and lookup to use the actual function parameters. Do not claim the commit builds until a real build at this exact HEAD proves it.
2. **DIX destructor still destroys the canonical record.** Current `XaveriusPhysicalDeviceDIXDestroy()` unlinks `dev->list`, decrements `xaverius_physical_device_count`, frees `identity_evidence`, and calls `free(dev)`. This contradicts the normative lifetime addendum: DIX wrapper/resource destruction must not unlink or free the server-lifetime PhysicalDevice/tombstone/XID mapping. Separate DIX wrapper ownership from registry membership and preserve canonical state until ordered server shutdown.
3. **Snapshot ABI has no size contract.** `query_physical_snapshot(binding, &dev->snapshot)` returns a pointer but no size. `snapshot_size` is initialized to zero and is used in `calloc`/copy. Commit `eaff9a2` removed the zero-size rejection, making the situation worse, not fixing it. Define a size-aware, lifetime-safe result contract consistent with the normative spec; validate non-NULL pointer, nonzero bounded size and all arithmetic before copying. If the normative ABI needs amendment, propose it separately and do not silently diverge.
4. **Init/fini remain incomplete.** `XaveriusRegistryInit()` reinitializes global lists/counters without an initialization guard. `XaveriusRegistryFini()` only resets counters and does not perform the required ordered shutdown/release. Implement only the current Stage 1–4 subset, with an explicit safe lifecycle and no double cleanup.

## Work order

- First fix the undeclared identifier in `e5aa972` and run the actual focused build at the resulting HEAD.
- Then make one small commit for snapshot result ABI/validation and tests.
- Then make one small commit for DIX wrapper versus canonical registry/tombstone lifetime and tests.
- Then address init/fini safely, with tests for repeated init, shutdown, outstanding DIX wrappers, and tombstone retention.
- Audit the remaining W0-022/W0-029 requirements against the current normative spec, including required callback/table validation, deep-copy ownership, ModuleRole reference balance, generation domains, observation/removal authority and idempotence. Report each remaining item; do not silently mark all A–G complete.

## Evidence/reporting

For every commit report parent and SHA, changed paths, exact build/test commands and outputs, and actual branch/HEAD/dirty state. A commit message saying "Build green" is not evidence. GitHub status/workflow results currently show no checks for `e5aa972`.

No Stage 5, Screen/DDX integration, modesetting, Proc handlers, Vulkan backend, unrelated renames, formatting or broad refactors. Preserve the three designated headers and current code style. Do not wait for Enterprise.


## 2026-11-07 owner re-audit of current branch tip

Branch still points to `e5aa97207f03de941e4e44196c774c5608d1629c`; no newer commit is visible. I re-fetched the actual `Xext/xaverius/registry.c` and `xaverius_intern.h` from that branch. The following are still present:

- `XaveriusRegistryObserveDevice()` around lines 708–760 references `registration->identity_evidence` and `registration->identity_evidence_size`, although its actual parameters are `identity_evidence` and `identity_evidence_size`. This remains a definite compile error.
- The registration path's new length guard is incomplete: it conditionally performs identity lookup only for an exact-size evidence buffer, but later unconditionally casts any non-NULL `registration->identity_evidence` to `XaveriusDeviceIdentity *` when assigning `dev->identity`. Reject non-NULL evidence with any size other than exactly `sizeof(XaveriusDeviceIdentity)` before any cast/read. Also define and validate the absent-evidence case explicitly; do not silently interpret malformed evidence as a zero identity.
- `XaveriusDeviceBackendOps.query_physical_snapshot` in `xaverius_intern.h` still has signature `XaveriusBackendResult (*query_physical_snapshot)(void *, void *)`; the call in `registry.c` returns only a pointer through `&dev->snapshot`, with no size output or owned-result contract. `dev->snapshot_size` remains initialized to zero and no size is assigned before validation/copy. This does not satisfy the normative immutable snapshot value contract.
- `XaveriusPhysicalDeviceDIXDestroy()` still unlinks `dev->list`, decrements the canonical registry count, frees `dev->identity_evidence`, and frees `dev`. The comments claim canonical retention, but the code still destroys the canonical record. DIX resource-wrapper ownership and canonical PhysicalDevice lifetime must be separate.
- `XaveriusRegistryInit()` still unconditionally reinitializes all global lists/counters. `XaveriusRegistryFini()` still only resets counters; it does not perform ordered registry cleanup or role/binding release.
- The backend ops descriptor still uses a two-argument snapshot callback, whereas the normative addendum requires the versioned descriptor/callback ABI and deep-copied operation tables. Do not resolve this by an undocumented local ABI change: propose/record a normative spec amendment if required, then implement the agreed contract.

Immediate order remains: (1) fix compile error and exact evidence validation, build; (2) settle snapshot ABI against normative spec and implement size/lifetime-safe snapshot ownership; (3) separate DIX wrapper lifetime from canonical registry record/tombstone; (4) implement guarded init and ordered fini. Small commits, tests and actual build logs for each. No Stage 5/DDX integration or broad refactor.


## Owner re-audit — current branch still unchanged at e5aa972 (2026-11-07)

No newer commit exists on `X11Libre/xserver:wip/xaverius`. Rechecked the exact current source; the previous commit did not fix the compile blocker or the registry lifetime defects.

1. **Immediate compile blocker remains:** `XaveriusRegistryObserveDevice()` parameters are `identity_evidence` and `identity_evidence_size`, but its body still references undeclared `registration->identity_evidence` and `registration->identity_evidence_size` around the canonicalization lookup. Use the function parameters and keep the exact-size validation already performed at entry. Do not introduce a registration object.
2. **Registration evidence is still unsafe:** after the lookup's exact-size check, registration later casts any non-NULL `registration->identity_evidence` to `XaveriusDeviceIdentity *` to initialize `dev->identity`. The copy path accepts arbitrary positive sizes. Reject non-NULL evidence unless size is exactly `sizeof(XaveriusDeviceIdentity)` before any cast, copy, or read. Define absent-evidence behavior explicitly; do not silently manufacture an all-zero identity and register it as canonical.
3. **Snapshot ABI/ownership still broken:** `XaveriusDeviceBackendOps.query_physical_snapshot` remains `(void *, void *)`; no size is returned. `dev->snapshot_size` is initialized to zero and never assigned before `calloc(1, dev->snapshot_size)` and `memcpy`. The normative immutable snapshot contract cannot be met with this signature. Reconcile the ABI with the spec/addendum first, then implement a versioned, size-safe callback/result contract and deep-copy server-owned snapshot data. Do not paper over this by guessing a size.
4. **Canonical DIX lifetime still broken:** `XaveriusPhysicalDeviceDIXDestroy()` still unlinks `dev->list`, decrements the canonical registry count, frees identity evidence, and frees `dev`, despite comments saying the canonical record survives as a tombstone. Separate the DIX wrapper/resource lifetime from the canonical registry record/tombstone. DIX destruction must not destroy the canonical registry identity.
5. **Init/fini remain unsafe/incomplete:** `XaveriusRegistryInit()` unconditionally reinitializes all global lists/counters; `XaveriusRegistryFini()` merely zeroes counters and performs no ordered cleanup/release of records, bindings, ModuleRole refs, snapshots, observations or sources. Implement guarded init and ordered teardown consistent with DIX resource destruction and module quiescence.
6. **Descriptor contract:** top-level and typed backend ops ownership/deep-copy and mandatory callback validation still need to match the normative versioned ABI. Keep the operation tables immutable and server-owned for the lifetime of the binding.

Required order: (a) fix ObserveDevice compile error and exact evidence validation; build with the actual Xserver build command and provide captured output; (b) agree and implement snapshot ABI/ownership; (c) separate DIX wrapper from canonical record/tombstone; (d) guarded init and ordered fini. Small coherent commits, focused tests, exact build/test output. No Stage 5/DDX integration, broad refactor, or unrelated formatting. Do not claim build-green from a commit message; CI status checks for e5aa972 are empty.


## Owner re-audit — unchanged current HEAD e5aa972 (2026-11-07 08:45Z)

Fresh fetch confirms branch `wip/xaverius` still points to `e5aa97207f03de941e4e44196c774c5608d1629c`; no new commit. Exact current source findings remain unchanged:
- `registry.c:708-760`: `XaveriusRegistryObserveDevice()` parameters are `identity_evidence` and `identity_evidence_size`, but lines 756-759 reference undeclared `registration->...`. Definite compile blocker.
- `registry.c:516-534`: the copy path accepts arbitrary positive evidence sizes, then line 531 casts every non-NULL buffer to `XaveriusDeviceIdentity`. Validate exact size before copy/cast/read, or reject unsupported absent evidence according to the agreed identity contract.
- `xaverius_intern.h:108-114`: snapshot callback remains `(void *, void *)`; `registry.c:541-542` supplies only `&dev->snapshot`. `snapshot_size` remains zero; no size/lifetime contract.
- `registry.c:60-80`: DIX destructor removes list membership and frees identity evidence and `dev`; comments claiming canonical retention do not match executable behavior.
- `registry.c:100-124`: init reinitializes globals unconditionally; fini only zeroes counters.
No code/build/test evidence arrived. Keep task open. First commit must fix compile/evidence validation and provide actual build output; do not bundle unrelated work.


## Owner refresh — 2026-11-07 10:58Z

Fresh fetch confirms `X11Libre/xserver:wip/xaverius` remains at `e5aa97207f03de941e4e44196c774c5608d1629c`. Current source blobs: `Xext/xaverius/registry.c` = `77ff067e84231e55c947b8bc367f3a6f66079a68`; `Xext/xaverius/xaverius_intern.h` = `be6e1a8547521a488d0e985ed48f490992133acd`.

Reconfirmed exact blockers: `registry.c:756-759` references undeclared `registration->identity_evidence{,_size}` inside `XaveriusRegistryObserveDevice()`, whose actual parameters are `identity_evidence` and `identity_evidence_size`; this is a compile blocker. `XaveriusPhysicalDeviceDIXDestroy()` lines 60-79 still unlinks and frees the canonical `dev` and its identity evidence despite comments claiming canonical retention. `XaveriusRegistryInit()` still unconditionally resets global lists/counters, and `XaveriusRegistryFini()` only resets counters. Snapshot callback remains `(void *, void *)` with no size/result ownership contract; `snapshot_size` is not populated. No new code commit or CI status evidence exists. Keep W0-068 open and follow the previously specified ordered, small-commit repair sequence.
