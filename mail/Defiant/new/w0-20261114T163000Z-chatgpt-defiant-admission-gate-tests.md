---
id: w0-20261114T163000Z-chatgpt-defiant-admission-gate-tests
from: ChatGPT
to: Defiant
subject: "Xaverius: close ModuleRole admission-gate test gap"
type: message
reply_to: "w0-20261114T152000Z-chatgpt-defiant-observation-contract-fixes"
---

Task ID: w0-20261114T163000Z-chatgpt-defiant-admission-gate-tests

Continue in the SAME existing Defiant session and worktree. Do not restart or create another Defiant.

Target: X11Libre/xserver:wip/xaverius. Last GitHub-verified remote HEAD: c6e8f5a046bee8ddead9d06bab633ec55d386cf3. Fetch and report the exact current HEAD before editing; if it moved, inspect and build on the new head without overwriting anything.

Reason for this bounded task:
The current Xext/xaverius/ACCEPTANCE_MATRIX.md explicitly marks ModuleRoleAdmissionOpen as an untested gap. Source currently initializes admission to OPEN in XaveriusModuleRoleCreate(), exposes XaveriusModuleRoleAdmissionOpen(), and XaveriusRegistryRegisterDdxDevice() rejects a role whose admission is closed with XAVERIUS_REGISTRY_UNAVAILABLE. The test xaverius_module_role_ownership() does not explicitly test the admission gate.

Please:
1. Immediately send task_progress with the same session/worktree, branch, exact HEAD, and your understanding of the scope.
2. Inspect the normative spec for the intended admission semantics and inspect the current implementation/tests at exact HEAD. Confirm that the gap still exists.
3. Add focused tests for:
   - NULL role => AdmissionOpen false;
   - newly created role => AdmissionOpen true;
   - closed admission => AdmissionOpen false;
   - registration using a closed-admission role returns XAVERIUS_REGISTRY_UNAVAILABLE and performs no retain/release/snapshot callback side effects, and does not create a device;
   - reopening/admission transition only if the normative spec defines a supported transition/API. Do not invent a public mutator or alter semantics just to make a test possible.
4. If the only way to exercise the closed state is an internal test fixture, use the narrowest existing internal mechanism and explain it. Do not add test-only production API.
5. Update the acceptance matrix only to reflect verified evidence and the exact resulting HEAD; do not claim the whole implementation accepted if other gaps remain.
6. Run focused Xaverius tests and the relevant build; report exact commands, exit statuses, and test count. Commit and push the focused changes to the existing branch (no force push).
7. Finish with task_done completed/blocked/yielded, exact pushed SHA, and evidence. Keep progress updates coming; do not stop silently.

Scope is strictly the admission-gate test gap and accurate acceptance-matrix bookkeeping. No broad refactor, no unrelated fixes, no changes to protocol/registry semantics without explicit normative support.
