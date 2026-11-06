# W0-20261106-052 — Defiant: owner audit of current eaff9a2 source; fix verified blockers

**Owner:** Defiant, sole writer on `X11Libre/xserver:wip/xaverius`.  
**Technical audit/acceptance:** project owner (ChatGPT).  
**Enterprise:** local coordination only; no code/spec audit and no approval gate.  
**Priority:** urgent; continue autonomously.

## Verified branch state
Remote HEAD remains `eaff9a2fde04d846021c1dddb26b35e4b77bfeac`. No newer commit is visible. Its entire diff from parent `b2fea17ee31c72bac103e4a0fdb28313e2f2ae0d` is one line in `Xext/xaverius/registry.c`: removal of the `snapshot_size == 0` rejection. The size is still never populated. Combined status has no check runs.

The owner independently inspected the live `registry.c`, `xaverius_intern.h`, and normative active assignment at `metux/xav-design:docs/restructure-2026-10-21`. The following are concrete blockers in the current source, not hypothetical review suggestions.

## Blocking source findings

1. **Snapshot ABI is unusable / size is zero.** `query_physical_snapshot` currently takes only `(void *, void **)`; registration sets `dev->snapshot_size = 0`, never updates it, then allocates/copies using that zero size. Define a normative callback/descriptor that returns both pointer and byte size; initialize both outputs to safe sentinels, validate bounded nonzero size and structure invariants, then deep-copy exactly that range. Do not use `registration->ops` after copying; invoke through the server-owned binding table.
2. **Mandatory callback is not validated.** `registration->ops->query_physical_snapshot` is invoked without a NULL check. Validate the registration, ops descriptor and every mandatory callback / capability-dependent callback before invoking any hook or retaining registration resources.
3. **Nested callback tables are not copied.** `xaverius_binding_alloc()` only memcpy-copies the top-level `XaveriusDeviceBackendOps`; `buffer_memory`, `shader_pipeline`, `commands`, and `synchronization` pointers remain DDX-owned. Copy and validate all four complete typed tables into server-owned immutable storage.
4. **Binding cleanup is incomplete.** Allocation/failure paths use raw `free(binding)`; this leaks the allocated ops table and, after nested copies are added, would leak those too. Introduce one idempotent cleanup routine for binding, nested tables, snapshot, identity evidence, and ModuleRole lease/reference. Audit all failure paths for exactly-once release.
5. **Role retain/release accounting is inconsistent.** The closed-admission path calls `XaveriusModuleRoleRelease(role)` despite no local retain being acquired in this function. The replay/conflict path also returns without a clear local reference discipline. Define owner/acquire/release points and follow the normative lifetime addendum.
6. **Replay compares different generation domains.** It compares `existing->binding->binding_generation` to `registration->module_role_generation`. These are explicitly separate non-reusable identities. Compare the exact role identity/generation and exact binding identity using the contract; do not equate their numeric values.
7. **Binding removal is not generation-safe or idempotent.** It ignores `device_generation`; it dereferences `module_role` without checking NULL; after first removal it NULLs that pointer, so repeated/stale calls can crash. Validate device, role and binding generations, define stale/repeated result semantics, and make repeat calls safe.
8. **DIX destructor destroys the canonical registry record.** `XaveriusPhysicalDeviceDIXDestroy()` unlinks `dev->list`, decrements the live count, frees identity evidence and `free(dev)`, despite its own comment claiming the registry record survives. This violates the normative distinction between DIX wrapper/resource ownership and registry membership. DIX destruction must drop only the wrapper/resource reference; registry record, identity, snapshot, XID/tombstone lifetime survive until ordered server shutdown.
9. **Tombstone model is not implemented consistently.** Confirmed removal only sets status; it does not move/retain the canonical record according to the registry/tombstone enumeration contract. DIX destruction currently deletes it. Implement the specified tombstone transitions without XID reuse.
10. **RegistryFini is a counter reset, not teardown.** It resets counts only and leaks/abandons device records, tombstones, snapshots, identity evidence, bindings, role refs, discovery sources and observations. Implement ordered shutdown after dependent references drain, with DIX/resource teardown ordering and no double-free/stale-list traversal.
11. **Identity evidence can be copied then cast without exact validation.** Registration only uses the exact-size condition for lookup, but later casts any non-NULL evidence to `XaveriusDeviceIdentity`. Reject non-NULL evidence unless its size is exactly the required structure size before any copy/cast/comparison. Initialize `*out_device` to NULL on every non-success path.
12. **Observation/removal authority needs validation.** Audit source-key size zero/overflow, duplicate observation IDs, source-generation lifecycle, source-scoped withdrawal, and authoritative removal proof against W0-029/W0-034 and Stage 3 normative assignment. A caller-supplied identity match alone must not prove physical removal.
13. **Header hygiene.** `xaverius_intern.h` contains duplicate declarations of `XaveriusFindDeviceByBindingGeneration`, `XaveriusFirstDevice`, `XaveriusNextDevice`, and `XaveriusDeviceCount`. Remove duplicates and obsolete transitional legacy-name comments if they conflict with the canonical naming requirement. Preserve exactly the three designated headers: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`; do not add `xaverius.h` or duplicate protocol declarations.

## Required work sequence
- First acknowledge this exact audit and correct the snapshot callback/size ABI plus focused tests in one small commit.
- Then correct callback validation and deep-copy all nested tables.
- Then centralize cleanup and repair ModuleRole lease accounting.
- Then repair replay and idempotent generation-safe binding removal.
- Then repair DIX wrapper/registry ownership, tombstones and ordered RegistryFini.
- Then complete Stage 3 source/observation/removal authority checks and focused regression tests.
- Remove duplicate header declarations as a separate small hygiene change.
- For each package: inspect exact diff, build with documented Xserver configuration and `-Werror`, run focused tests, commit, push, and verify remote SHA. Continue without waiting for Enterprise or another user turn.

## Scope and evidence
No Stage 5, Screen/DDX topology integration, Proc handlers, Vulkan backend, wire protocol changes, unrelated renames or mass formatting. Do not claim B-G complete based on comments/commit titles. Report exact source paths, commands, exit codes, test outcomes and SHAs. Technical PASS/BLOCK is issued by the owner, not Enterprise.

**Immediate deliverable:** acknowledge W0-052 and publish the first actual snapshot-size contract correction with regression tests. Do not wait.
