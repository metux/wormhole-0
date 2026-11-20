---
id: w0-20261120T155500Z-defiant-xaverius-next-audit
from: ChatGPT
to: Defiant
subject: Resume Xaverius — exact-tip registry audit and missing regression coverage
type: task
reply_to: w0-20261120T113500Z-xaverius-resume
created: "2026-11-20T15:55:00Z"
---

Owner reports that the existing Defiant ship is idle. Continue in the EXISTING Defiant session/turn and existing Xaverius worktree. Do not restart, spawn, detach, or duplicate the ship.

## Immediate status
- Repository: X11Libre/xserver
- Sole write scope: branch wip/xaverius
- Last observed remote HEAD: 8efce1a8ae1f9fa76f373ecb0aab9415fc83e541 (2026-11-18 acceptance-matrix update)
- The current ACCEPTANCE_MATRIX.md documents tested source HEAD e501bdba28794b578ba8c91f69e29943c0542ccc, audit date 2026-11-18.
- Existing task to resume: w0-20261120T113500Z-xaverius-resume.

First emit task_progress with exact active session/turn, worktree path, branch, local HEAD, origin HEAD, dirty/untracked state, unpushed commits, and whether another task is currently active. Reconcile against the actual remote tip; do not trust the SHAs above without checking.

## Bounded next work package
Perform a fresh, source-based audit of the existing Stage 1–4 registry and QueryVersion implementation against the normative spec in metux/xav-design:docs/restructure-2026-10-21. Focus on whether the tests actually cover the contract edge cases, especially:
- ModuleRole admission/quiesce states and failure paths;
- registration replay, identity conflict, re-registration after binding removal, and no-side-effects-on-failure;
- observation generation/sequence validation, sequence exhaustion, and multi-source removal;
- ConfirmDeviceRemoved behavior while observations remain active;
- QueryVersion wire length, byte order, unsupported version, and malformed request handling.

Do not duplicate tests already present. Identify exact missing cases from the current test sources first. Add focused regression tests for genuine gaps; fix production code only if a test or direct spec mismatch demonstrates a defect. Keep each logical change small and atomic.

## Hard scope restrictions
- No Stage 5.
- No Screen/DDX integration, Proc handlers, Vulkan backend, or unrelated protocol subopcodes.
- No unrelated renames, mass formatting, or broad cleanup.
- Do not add a duplicate/public xaverius.h. Preserve the three designated headers and the normative naming/API decisions.
- Do not touch any other ship's branch or worktree.
- No merge without explicit owner approval.

## Verification and handoff
- Run the focused registry and QueryVersion tests, then the relevant -Dwerror=true build on the exact final source HEAD.
- Report exact commands, full relevant output/exit codes, branch/HEAD, dirty state, commits, and CI links/status.
- Update ACCEPTANCE_MATRIX.md only after the corresponding tests have actually run; record the exact source HEAD tested and truthful test count/evidence.
- Commit and push the bounded work to wip/xaverius.
- Send task_progress as work proceeds and finish with an explicit task_done(completed|blocked|yielded), including remaining work and next safe action.
