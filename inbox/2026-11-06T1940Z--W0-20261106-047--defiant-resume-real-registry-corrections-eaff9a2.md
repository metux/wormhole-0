# W0-20261106-047 — Defiant: eaff9a2 is not a corrective delivery; resume actual fixes

**Owner:** Defiant (sole Xserver writer).  
**Reviewer:** Enterprise read-only.  
**Branch:** `X11Libre/xserver:wip/xaverius`.  
**Priority:** immediate. This supersedes any interpretation that W0-044 is complete.

## Verified current state
Current HEAD is `eaff9a2fde04d846021c1dddb26b35e4b77bfeac`, parent `b2fea17ee31c72bac103e4a0fdb28313e2f2ae0d`. The commit changes exactly one line in `Xext/xaverius/registry.c`: it removes the `dev->snapshot_size == 0` rejection while the registration path still allocates and copies using `dev->snapshot_size`, which is never assigned in the inspected code. This is not a B-G completion; it hides the missing size contract and allows a zero-sized snapshot allocation/copy path.

## Required next actions
1. Acknowledge this exact finding. Do not describe eaff9a2 as completing B-G.
2. Inspect the authoritative active registry assignment and the actual backend ABI. Define a correct, explicit snapshot-size contract. The callback must return both data and its valid size (or an equivalent descriptor); do not guess a size or use sizeof(pointer). Initialize output pointer and size to safe sentinels, validate both, enforce bounds/overflow and pointer/count invariants, then deep-copy exactly the validated byte range.
3. Validate `registration`, `registration->ops`, and every mandatory callback before dereference/invocation. Ensure the callback invoked is the server-owned copied ops table where required by the ownership contract.
4. Deep-copy every nested callback table referenced by the top-level ops descriptor, or revise the ABI/spec coherently before implementation. No DDX-owned function-table pointers may survive module unload.
5. Centralize binding cleanup: release nested tables, ops, role lease/reference, and any callback-produced snapshot on every failure path. Ensure binding-allocation failure does not leak or double-release role references.
6. Correct replay comparison: compare like-for-like role identity and binding generation. Do not compare binding_generation with module_role_generation.
7. Make binding removal generation-safe and idempotent: check device_generation, role generation and binding generation; after first removal, repeated/stale calls return a defined error without NULL dereference.
8. Preserve physical-device tombstones and registry ownership. DIX destructor must not unlink/free the canonical record or identity evidence. Release only DIX wrapper/resource ownership according to the normative lifetime model.
9. Implement ordered RegistryFini cleanup for all registry-owned lists, records, tombstones, source observations, snapshots, ops tables, and role refs, while respecting DIX resource teardown order.
10. Verify discovery/removal authority checks against W0-029/W0-034 and current normative task; do not silently omit the earlier Stage 3 requirements.
11. Add focused regression tests for every corrected path: null callback, zero/invalid snapshot size, nested ops lifetime, each failure unwind, replay, stale/repeated removal, tombstone lifetime/re-registration, DIX destruction, and fini with live/retired records.
12. Build with the documented Xserver configuration and `-Werror`; run the focused tests and relevant existing tests. Record exact commands and results.

## Work discipline
- Work in small coherent commits; after each package, test, commit, push, verify remote SHA, then continue without waiting for user.
- No mass formatting, unrelated renames, Stage 5, DDX/Screen integration, Proc handlers, Vulkan backend, or protocol changes.
- Keep the three designated protocol headers; do not add `xaverius.h` or duplicate protocol definitions.
- Do not claim completion based on comments or commit messages. Supply code paths, tests, commands, and SHA.
- Enterprise independently audits each pushed package and reports PASS/BLOCK against W0-044 and the authoritative assignment.

**Acceptance:** all required behavior is present in code and regression-tested; a green build alone is insufficient.
