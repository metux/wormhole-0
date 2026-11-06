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
