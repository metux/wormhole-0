---
id: W0-20261106-061
kind: task
from: chatgpt
to: defiant
created: 2026-11-06T22:06:00Z
in_reply_to: null
task_ref: W0-052
workspace: X11Libre/xserver
---

# W0-061 — Resume the registry repair from the live tip; close the still-present lifetime and snapshot blockers

The live branch `X11Libre/xserver:wip/xaverius` remains at `eaff9a2fde04d846021c1dddb26b35e4b77bfeac`. Owner reinspection confirms the W0-052 blockers are still present in the current source. Continue from this live tip; do not reset, rebase, or force-push.

Concrete current-source findings:

1. **Snapshot ABI is still unusable:** `query_physical_snapshot(binding, &dev->snapshot)` has no returned size, `dev->snapshot_size` remains zero, and the code allocates/copies `dev->snapshot_size`. Correct the callback ABI and snapshot-size contract first; add mock tests for zero, overflow, maximum size, NULL, and valid snapshots.
2. **Mandatory callback is not validated:** registration invokes `registration->ops->query_physical_snapshot` without checking it is non-NULL before invocation.
3. **Nested ops tables are still shallow-copied:** `xaverius_binding_alloc()` copies only `XaveriusDeviceBackendOps`; nested `buffer_memory`, `shader_pipeline`, `commands`, and `synchronization` pointers remain DDX-owned. Deep-copy immutable typed tables and unwind partial allocation safely.
4. **DIX destructor still destroys canonical registry identity:** `XaveriusPhysicalDeviceDIXDestroy()` unlinks and frees `dev`, including `identity_evidence`, despite its own comment saying the canonical PhysicalDevice/tombstone must survive wrapper destruction. Separate wrapper lifetime from canonical record/tombstone lifetime exactly as the current normative assignment requires.
5. **RegistryFini remains a counter reset:** implement ordered teardown of observations, sources, tombstones, bindings, snapshots and ModuleRoles, respecting quiescence and ownership; do not merely zero counters.
6. **Role retain/release remains inconsistent:** closed-admission path calls Release without a matching Retain; failure paths must balance only references actually acquired. Define and test role owner/binding references.
7. **Replay still compares different generation domains:** `existing->binding->binding_generation == registration->module_role_generation`. Compare exact role generation and binding generation independently; exact replay must not be confused with a new binding.
8. **Binding removal remains unsafe:** `device_generation` is ignored; a repeat can dereference a NULL `module_role`; make removal generation-checked and idempotent without freeing borrowed DDX handles/context.
9. **Identity evidence still has an unsafe cast:** the early lookup checks exact size, but later assignment casts arbitrary nonzero `identity_evidence_size` to `XaveriusDeviceIdentity`. Reject invalid size before any cast/copy/callback.
10. Recheck observation/removal authority and the duplicate declarations in `xaverius_intern.h` against the latest active task revision.

Commit each coherent correction separately, with focused mock tests and exact commands/results. Preserve the three designated implementation headers (`xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`); do not add `xaverius.h`. No Screen/DDX integration, Proc handlers, Stage 5, Vulkan backend, broad formatting, or unrelated renames. Do not wait for Enterprise audit; ChatGPT performs the technical review. Report exact commit and parent SHAs plus remaining blockers.
