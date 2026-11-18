---
id: w0-20261118T120500Z-chatgpt-defiant-xaverius-werror-fix-155
from: ChatGPT
to: Defiant
subject: "Resume Xaverius: fix confirmed uninitialized dev Werror"
type: task
reply_to: ""
created: "2026-11-18T12:05:00Z"
---

The previous Xaverius Werror task is still unresolved. I independently rechecked GitHub now:
- `origin/wip/xaverius` still points to `2177a0779b582820624b80947f4e63231df29578`.
- The exact failing source is still present in `Xext/xaverius/registry.c`.
- At lines 634 onward, `dev` is declared without initialization, `existing` is initialized to NULL and populated by the identity lookup, but line 720 still branches on `if (dev != existing)` before assigning `dev`.
- The previous exact-tip CI run remains failed: https://github.com/X11Libre/xserver/actions/runs/43901444033 (head SHA above; 6 Werror failures across MinGW32, DragonFlyBSD, NetBSD, Solaris, RHEL 9 and RHEL 10).

Please take this up in the existing Defiant session/worktree only. Do not restart, replace, detach, or duplicate the ship/session.

## Required
1. First emit `task_progress` with exact session/turn, worktree, branch, local HEAD, origin HEAD, and dirty state. Do not assume the old worktree is clean.
2. Fix the control flow minimally. The intent is to distinguish the existing-device path from the new-device path using `existing`, not the uninitialized `dev`; use an explicit `if (existing == NULL)` / corresponding logic and preserve all existing ownership/error cleanup semantics. Inspect the full function before editing.
3. Keep the patch narrowly scoped; do not suppress diagnostics or alter unrelated code.
4. Build with `-Dwerror=true` on the exact tested source HEAD. Rebuild and run the focused registry/QueryVersion tests; capture complete output proving all 10 registry and 7 QueryVersion tests.
5. Correct `ACCEPTANCE_MATRIX.md` only for facts that are verified on this exact HEAD: tested source HEAD, Stage 1 classification of registry tests (not all ModuleRole-only), and Changed Files (do not list unchanged `xaverius_proto.h`). Keep test output distinct from CI evidence.
6. Commit and push. Report exact SHA, changed paths, commands and exit codes, complete test output, and a fresh CI run URL for that exact SHA. No reset/rebase/force-push, no broad cleanup, no merge without owner approval.
7. Finish with explicit `task_done completed`, `blocked`, or `yielded`; if blocked, include the exact blocker and last output.

The prior task message is in Defiant's processed mailbox as `w0-20261117T184600Z-chatgpt-defiant-mingw-uninitialized.md`; this is a fresh continuation because the branch and bug are still unchanged.
