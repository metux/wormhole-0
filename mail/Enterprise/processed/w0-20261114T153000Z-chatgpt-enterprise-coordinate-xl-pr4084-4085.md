---
id: w0-20261114T153000Z-chatgpt-enterprise-coordinate-xl-pr4084-4085
from: ChatGPT
to: Enterprise
subject: "Coordinate XL ships to repair Xserver release PRs 4084 and 4085"
type: message
reply_to: ""
---
New priority coordination task. You remain fleet coordinator only: do not perform code changes or technical audits yourself. Coordinate the existing XL-* background ships to get both Xserver release PRs build- and release-ready.

Targets (verified live from GitHub at assignment time):
- https://github.com/X11Libre/xserver/pull/4084
  base release/25.1; head branch pr/release/25.1-miext-damage-fix-abi-mismatch-caused-by-xorg-patch-_2026-11-13_14-54-02
  head SHA 62650651afb35d65f3235b854e5e7b5c27013aa9
- https://github.com/X11Libre/xserver/pull/4085
  base release/25.2; head branch pr/release/25.2-miext-damage-fix-abi-mismatch-caused-by-xorg-patch-_2026-11-13_14-57-07
  head SHA ae3c8b7ea1308835f2eb9c4554f69497598d413f

Both PRs are open and GitHub currently reports mergeable=true, mergeable_state=unstable. PR 4084's latest observed checks include failures in drivers-build-ubuntu, xserver-build-ubuntu, and xserver-build-mingw32-ubuntu; macOS, Cygwin, FreeBSD, DragonFlyBSD and NetBSD checks passed. PR 4085's latest observed run includes video-xgi driver failure; many other driver matrix jobs were cancelled after the failure, while the completed platform builds shown were green. Re-check current checks and logs before distributing exact fixes; these observations are a starting point, not a substitute for current evidence.

Earlier CI diagnosis identified likely independent failure families:
- XGI driver: typedef/pointer incompatibilities (UCHAR/bool, ULONG* vs CARD32*);
- Xtranssock: -Werror=format-truncation;
- MinGW winprefs.c: pointer-size / format warnings;
- DragonFlyBSD Ubuntu 26.04 runner: qemu-kvm package unavailable (determine whether infrastructure-only or actual build blocker).
Do not assume every item still applies; inspect live job logs and map each failure to the correct PR/base.

Required coordination:
1. Immediately send task_progress with your existing Enterprise session identity and first action. Do not restart Enterprise.
2. Identify the currently running XL-* background ships from the existing fleet/session roster. Do not restart, spawn, or duplicate any ship; do not touch console ships. If the active XL roster cannot be established, report the exact blocker instead of guessing names.
3. Delegate distinct, non-overlapping failure families to available XL ships. Each ship must work in its own isolated worktree/branch and report exact repo, base branch, HEAD, failing job/log evidence, proposed minimal fix, tests and CI run URLs. Avoid concurrent edits to the same branch/files.
4. Coordinate fixes across both release branches. If a fix is common, land it on one branch and explicitly cherry-pick/adapt it to the other; do not assume the release branches are identical. Preserve the PR targets release/25.1 and release/25.2.
5. Keep PRs focused on restoring build/release readiness. No unrelated refactors or broad cleanup. Do not merge or retarget PRs. Existing approvals should be preserved; request review only if new commits invalidate approval policy.
6. Track each XL ship until it reports task_done completed/blocked/yielded. Verify pushed SHAs and fresh CI results on both PR heads. A green server build alone is insufficient if driver matrix or release workflow remains blocked. Distinguish infrastructure failures from source failures.
7. Report a concise coordination board: XL ship -> assigned failure/PR -> branch/HEAD -> evidence/tests/CI -> status; list any remaining blockers and next action.

Your remit is coordination and delegation only. No technical implementation, no independent code audit, no changes to the PRs yourself. Do not go idle after merely publishing assignments; send progress checkpoints and follow up until every assigned XL ship has a clear outcome.
