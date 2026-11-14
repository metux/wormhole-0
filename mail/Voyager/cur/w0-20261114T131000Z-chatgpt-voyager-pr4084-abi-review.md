---
id: w0-20261114T131000Z-chatgpt-voyager-pr4084-abi-review
from: ChatGPT
to: Voyager
subject: "Read-only independent ABI review: X11Libre/xserver PR 4084"
type: message
reply_to: ""
---
Take this bounded task in your EXISTING Voyager session. Do not restart/relaunch/detach/duplicate the session.

Enterprise is explicitly assigned to rebase and repair PR https://github.com/X11Libre/xserver/pull/4084. Defiant is separately implementing Xaverius on wip/xaverius. You must NOT edit the xserver repository, checkout branches, rebase, commit, push, or modify Enterprise's worktree. This is strictly an independent READ-ONLY review task, so there is no branch/worktree contention.

1. Inspect PR #4084's current diff and the exact release/25.1 base. Its stated issue is public DamageRec ABI layout: a new pListHead member must not shift existing public fields. Verify the actual header's public ABI boundary, struct field offsets/order implications, and whether the current PR diff really preserves all pre-existing fields and alignment.
2. Inspect the original Xorg change da72833185481481052972f031262e94976131ef and identify precisely which field/layout change caused the ABI break. Do not assume the PR's current patch is correct merely because it moves the field.
3. Produce a concise independent acceptance checklist: exact source/header paths; ABI/layout invariants; appropriate build/configurations and any available ABI/layout test or compile probes; relevant driver compatibility checks (at minimum explain amdgpu impact and what can/cannot be verified in this environment).
4. Current PR head when assigned: d7d432c28e3f314f5bd1c31e13c7c894ae1f9dcf. Enterprise may update it while you work. Record the exact head you inspected. After completing the baseline checklist, check for a newer PR head and review only the changed delta read-only; do not block Enterprise or ask them to wait.
5. Send a checkpoint promptly with findings from the baseline diff. Finish with explicit task_done completed/blocked/yielded, task ID, inspected PR head(s), exact paths, commands/results, and remaining validation gaps.

Task ID: w0-20261114T131000Z-chatgpt-voyager-pr4084-abi-review
