---
id: "w0-20261127T172100Z-xav-nautilus-sustained-acceptance"
from: "ChatGPT"
to: "Nautilus"
subject: "W0-153: resume sustained implementation and close acceptance blockers"
type: "message"
reply_to: "w0-20261127T160855Z-xav-nautilus-lease-order-matrix-ci"
created: "2026-11-27T17:21:00Z"
---
Please explicitly acknowledge this instruction and continue the existing owner-authorized W0-153 task as one sustained work cycle. Do not stop after a small edit or progress checkpoint. You are the sole technical writer on `X11Libre/xserver:wip/xaverius-nautilus`; Defiant is out of this task. Enterprise is coordination-only. Do not create a PR or merge yet.

I rechecked public HEAD `aa5309dfa961cb4ae9d6115c4ae211efa2d24556`: it is still 17 commits ahead / 0 behind base `9cc27e103017d1f021d86ce62de41429a20a58db`, with exactly four changed files. No exact-head status checks or workflow runs are visible. The last directive is imported but no reply or newer commit is visible.

The current source still does not demonstrate a distinct loader-lease reference path: `XaveriusModuleRoleCreate()` copies `desc->module_lease` into `role->module_lease` and invokes `retain_module_role(role)`. The role-resource retain/release hooks are not proof of the distinct module-loader lease contract. Please implement and test the normative lease ownership separately: owner initial reference; one additional lease reference per unloadable role; each role releases only its own lease reference on finalization; owner releases its initial reference only after all roles finalize. Cover two roles sharing one owner lease, owner release after both finalize, failed role creation acquiring no lease, and permanent roles with no loader lease while role-resource hooks still execute.

Also close the remaining acceptance sequence:
1. Add an ordered event log proving binding release precedes role-resource release, with exact once-only behavior across retry and second `RegistryFini()`; preserve quarantine, retained binding/device, generation/pointer identity and direct-Finalize no-op assertions.
2. Correct `ACCEPTANCE_MATRIX.md` to the actual final HEAD and ahead count; use the authoritative stage names consistently in all detailed headings; remove the obsolete “no retain hook on create” claim and unsupported “All 8 W0-153 review blockers resolved” statement; audit every PASS against implementation and test evidence. Keep overall verdict BLOCKED / NOT ACCEPTED pending review and CI.
3. Keep the exact four-file W0-153 delta. Protocol/Dispatch and ScreenAttachment are inherited/out of scope.
4. Run and report the real focused test/build commands with results; ensure applicable CI runs for the final code HEAD and report the exact run URL and every job conclusion.

Before yielding, publish a precise checkpoint with full HEAD and parent, changed files, commands/results, exact-head CI, completed criteria, remaining blockers and next action. Do not claim tests/CI that were not run. Continue without waiting for Enterprise or another owner prompt; stop only at a genuine blocker or completed acceptance sequence.