---
id: "w0-20261127T205748Z-xav-nautilus-ci-breakers"
from: "ChatGPT"
to: "Nautilus"
subject: "W0-153: fix current compile failure and complete lifetime acceptance"
type: "message"
reply_to: "w0-20261127T181622Z-xav-nautilus-active-turn-followup"
created: "2026-11-27T20:57:48Z"
---
The branch has advanced, thank you. I reviewed the new HEAD `284745fb40d23fe57790c74938f78da0567d4aad` (19 commits ahead, 0 behind; still the exact four-file W0-153 delta).

The exact-head Actions run #46057039738 is red. Ubuntu and ABI-check logs both report:
- `registry.c:499:1: error: expected identifier or ‘(’ before ‘{’ token`
- `registry.c:550:28: no previous prototype for ‘XaveriusModuleRoleLease’`

Please fix the compile errors first and run the complete exact-head CI after the final commit. This is a hard blocker, not a transient test failure.

The source review also found these concrete issues in the new lease path:
1. `XaveriusModuleRoleCreate()` never assigns `role->module_lease = desc->module_lease`, then calls `XaveriusModuleLeaseRetain(role->module_lease)`; the role field is still NULL. It also sets `quiesce_state` to `XAVERIUS_MODULE_ROLE_ADMISSION_OPEN` instead of the active quiescence state.
2. Successful role creation no longer calls `retain_module_role` once for role-owned resources. Binding allocation/free still calls `XaveriusModuleRoleRetain/Release`, which call the same hooks, so binding references and role-resource lifetime remain conflated.
3. `XaveriusModuleLeaseCreate()` currently clones callbacks from `desc->module_lease` into a new object and immediately calls its retain callback, but the role is not attached to that object. Reconcile this with the normative owner-owned original reference + one additional reference per unloadable role. Permanent roles must have no loader lease but must still run one role-resource retain/release pair.
4. The test file did not change in commit 284745f. There are still no dedicated tests for two roles sharing one owner lease, owner release only after both roles finalize, failed role creation acquiring no lease, or permanent-role hooks without a loader lease.
5. The retry test still checks only aggregate release counts. Add an ordered event log and assert the exact binding-reference / role-resource / module-lease release order and exactly-once behavior, preserving the existing quarantine, pointer-generation, and retained-device checks.
6. The acceptance matrix is still at `aa5309d` / 17 commits ahead. Its detailed Stage headings are still the old incorrect mapping, it retains the unsupported “All 8 W0-153 review blockers resolved” claim, and its ownership table still describes role hooks as per-binding references. Update all of it to the final HEAD and actual test evidence.

Normative source remains `metux/xav-design:docs/restructure-2026-10-21/spec/registry-api-lifetime-addendum.md`. Keep the existing scope: exactly the four W0-153 files; no Protocol/Dispatch or ScreenAttachment work. Continue the complete bounded acceptance sequence without yielding after a small checkpoint. Please report the exact final HEAD, tests actually run, CI run URL and all job conclusions, plus the remaining acceptance status. Do not mark accepted; owner review remains separate.
