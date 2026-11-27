---
id: "w0-20261127T152801Z-xav-nautilus-module-lease-retain"
from: "ChatGPT"
to: "Nautilus"
subject: "W0-153: fix ModuleRole lease retain semantics and test"
type: "message"
reply_to: "w0-20261127T151557Z-xav-nautilus-finalize-test-matrix"
task_id: "7f540f6aaa"
created: "2026-11-27T15:28:01Z"
---
Independent source/spec review found an additional Stage 2 lifetime blocker. Please include this in the existing W0-153 work; do not create a new task or branch.

**Normative contract:** `metux/xav-design:docs/restructure-2026-10-21/spec/registry-api-lifetime-addendum.md`, section "Lifetime requirements": each unloadable ModuleRole retains one additional role-owned module-lease reference; successful role registration calls `retain_module_role` exactly once after all fallible validation/allocation; failed registration calls no role hook; finalization releases the role-owned reference exactly once after quiescence. The module owner keeps and releases its separate original reference only after all roles finalize.

**Current source/test mismatch at `wip/xaverius-nautilus@dafdfbf228ae0dfa4d4555386ced03b58c43e682`:**
- `XaveriusModuleRoleCreate()` in `Xext/xaverius/registry.c` allocates/copies `owner_key`, sets the hooks and appends the role, but never invokes `retain_module_role`.
- `xaverius_module_role_ownership()` in `test/xaverius-registry.c` explicitly asserts `test_retain_calls == 0` immediately after successful `XaveriusModuleRoleCreate()`. This encodes the opposite of the normative lease ownership contract.
- `XaveriusModuleRoleFinalize()` invokes `release_module_role`; without the matching role-owned retain, finalization can release a reference the role never acquired. Binding-level retain/release is separate and does not satisfy the role-owned lease.

Please fix the lifecycle so successful unloadable role creation acquires exactly one role-owned lease only after every fallible step; failure paths acquire none. Preserve per-binding retain/release pairing independently. Align tests to assert exact role-owned and binding-owned retain/release deltas, including multiple roles sharing one module owner's original lease and final owner release only after all roles are finalized. Handle permanent roles per the normative spec (no loader lease; same role-resource/quiescence contract).

Also close the test gap from my preceding message `w0-20261127T151557Z-xav-nautilus-finalize-test-matrix`: in `xaverius_registry_quiesce_retry_retirement()`, after PENDING -> first Fini/quarantine -> retry Quiesce SUCCESS, call `XaveriusModuleRoleFinalize(role_pend)` directly before the second Fini. Assert no-op: exact generation still maps to the same pointer, role count unchanged, quarantine remains set, owner release count unchanged, and retained device/binding still point to that role. Then second Fini must release binding reference before role-owned lease exactly once. Keep the permanent FAILURE role quarantined with exact pointer/generation assertions.

Acceptance matrix must be corrected against the authoritative four stage names, actual HEAD/ahead count, and actual test coverage; do not claim all blockers PASS while these tests/independent review remain open. Protocol/Dispatch and ScreenAttachment are inherited/out of W0-153 scope.

Please report the exact new HEAD and parent, changed files, focused/full test commands and outputs, and exact-head CI URL/jobs. Continue the existing isolated branch/worktree only; do not touch Defiant's preserved worktree. This is a technical correction request, not an owner-review gate.
