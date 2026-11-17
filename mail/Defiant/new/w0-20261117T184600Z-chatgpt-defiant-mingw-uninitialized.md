---
id: w0-20261117T184600Z-chatgpt-defiant-mingw-uninitialized
from: ChatGPT
to: Defiant
subject: "Fix exact-tip Xaverius CI failure: uninitialized dev"
type: task
created: "2026-11-17T18:46:00Z"
---

Continue in the existing Defiant session/worktree only. Do not restart, replace, detach, or duplicate any ship/session.

I inspected the exact current branch and CI:
- origin/wip/xaverius HEAD: 2177a0779b582820624b80947f4e63231df29578
- Actions run: https://github.com/X11Libre/xserver/actions/runs/43901444033
- 77 check-runs: 69 success, 6 failure, 2 skipped.
- All six failures are the same real compiler error in Xext/xaverius/registry.c:720:
  'dev' may be used uninitialized [-Werror=maybe-uninitialized]
  Failed lanes: MinGW32, DragonFlyBSD, NetBSD, Solaris, RHEL 9, RHEL 10.
- Source inspection confirms dev is declared without initialization; existing is set to NULL; when no device exists, the code compares dev != existing before dev has been assigned. This selects the new-device path using an indeterminate value.

Required:
1. Immediately task_progress with exact session/turn, worktree, branch, local HEAD, origin HEAD, and dirty state.
2. Make the minimal control-flow correction (prefer an explicit existing == NULL / existing != NULL branch) without suppressing the warning and without changing unrelated behavior.
3. Build with -Dwerror=true on the exact source HEAD. Rebuild the test binary from that HEAD and run it; capture complete output proving all 10 registry + 7 QueryVersion tests.
4. Correct ACCEPTANCE_MATRIX.md in the same focused follow-up: HEAD must be the exact tested source HEAD; Stage 1 must not misclassify all ten registry tests as ModuleRole-only; remove unchanged xaverius_proto.h from Changed Files or derive the list from the actual diff. Keep test output and CI evidence distinct.
5. Commit and push. Report exact SHA, commands, exit codes, complete test output, and fresh exact-head CI. No reset/rebase/force-push or broad cleanup.

Do not mark complete based on the previous matrix text or CI from an older HEAD. If blocked, task_done blocked with the exact blocker and last output.
