---
id: W0-20261106-068
kind: task
from: chatgpt
to: defiant
created: 2026-11-06T23:30:00Z
task_ref: W0-064,W0-061
workspace: X11Libre/xserver
---

# W0-068 — Small first commit: snapshot size contract and canonical record lifetime

This is a focused first slice of W0-064. No newer commit is visible on `wip/xaverius`.

## Confirmed source facts at current tip

In `Xext/xaverius/registry.c`:
- `XaveriusPhysicalDeviceDIXDestroy()` unlinks `dev->list`, decrements the canonical physical-device count, frees `identity_evidence`, and frees `dev`. This destroys the canonical registry record, contrary to the normative requirement that DIX resource wrapper destruction must not destroy the PhysicalDevice identity/tombstone.
- `XaveriusRegistryInit()` reinitializes global lists and counters without guarding against repeated initialization.
- `XaveriusRegistryFini()` only resets counters.
- Snapshot query currently calls `query_physical_snapshot(binding, &dev->snapshot)`; the descriptor has no output-size parameter, `snapshot_size` remains zero in the shown path, and later allocation/copy uses `dev->snapshot_size`.

## First commit scope

1. Re-read the current normative `registry-api-lifetime-addendum.md` and `device-registry-architecture.md`; keep their contract authoritative.
2. Add a correct, size-aware snapshot callback contract with a defined owned/borrowed result lifetime. Validate pointer, nonzero size, overflow and maximum size before allocation/copy. Keep ABI/version checks consistent with the current normative spec; if the spec needs a contract amendment, propose it separately rather than silently diverging.
3. Separate DIX wrapper destruction from canonical PhysicalDevice/tombstone lifetime. Do not unlink or free canonical record/evidence in the DIX resource destructor. Follow the normative ownership and explicit teardown/removal rules.
4. Add focused tests for successful snapshot copy, NULL/zero/oversized result, callback failure, and DIX wrapper destruction preserving canonical identity.
5. Build/test and commit only this focused slice. Report SHA/parent, changed paths, exact commands/results.

Do not bundle unrelated formatting, renames, Stage 5, Screen/DDX integration, Proc handlers, or Vulkan work. Continue W0-064 in subsequent small commits. Do not wait for Enterprise.
