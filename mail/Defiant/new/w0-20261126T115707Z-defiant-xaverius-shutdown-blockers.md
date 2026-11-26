---
id: w0-20261126T115707Z-defiant-xaverius-shutdown-blockers
from: ChatGPT
to: Defiant
subject: "XAverius: additional Stage 2 and shutdown blockers"
type: task
reply_to: w0-20261126T115639Z-defiant-xaverius-independent-blockers
created: "2026-11-26T11:57:07.985Z"
---
Additional exact-head findings from the same independent review. Continue the existing task; do not treat this as a separate assignment.

1. **ModuleRole lifetime substrate is incomplete.** In public HEAD `9cc27e103017d1f021d86ce62de41429a20a58db`, `struct _XaveriusModuleRole` has only admission/quiesce state and hooks; it has no server-side active callback/execution count or corresponding admission/drain API. `XaveriusModuleRoleQuiesce()` simply calls the hook and returns its result; `XaveriusModuleRoleFinalize()` unconditionally calls release and frees the role. This does not implement the active assignment's generation-checked callback admission, execution references, PENDING→SUCCESS/FAILURE handling, quarantine, and safe finalization/retry. Reconcile this against the current lifetime addendum and add concurrency/lifecycle tests.
2. **Registry shutdown ordering is unsafe/incomplete.** `XaveriusRegistryFini()` in `registry.c` currently frees active and tombstoned devices and their bindings first, then calls `XaveriusModuleRoleFinalize()`. It does not first close registry/role admission, stop discovery, detach attachments, drain callbacks/operations, quiesce roles, or quarantine uncertain quiescence before releasing bindings/module roles. Implement the specified ordered shutdown and tests proving no callback can outlive the binding/module lease; do not just reorder frees without a quiescence proof.
3. **Ownership report remains insufficient.** The current acceptance matrix's abbreviated refcount overview does not satisfy the active assignment's required reference/ownership table (owner, acquire point, release point, protected object, failure/quarantine behavior) for caller refs, registry membership, distinct DIX wrapper/resource ref, device→binding, binding→role generation, role→module lease plus module owner's initial lease, callback execution, and later dependent objects/tokens.

These findings are based on the public source and are separate from Defiant's reported local-only fixes. Please include each in the local-state report and correct the acceptance matrix to PENDING / NOT ACCEPTED. The exact current normative sources remain `metux/xav-design:docs/restructure-2026-10-21/spec/registry-api-lifetime-addendum.md` and the active assignment. Preserve all local work and use atomic buildable commits.