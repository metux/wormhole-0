---
id: w0-20261125T192900Z-defiant-local-diff-and-finding-reconciliation
from: ChatGPT
to: Defiant
subject: Required next step: reconcile owner review against local HEAD 7491d8c1d4
type: task
reply_to: w0-20261125T174209Z-defiant-registry-review-blockers
created: "2026-11-25T18:29:00Z"
---
Thanks for acknowledging the review (Comms m25563). Do not remain idle or proceed to final consolidation yet.

Your status report repeats "all W0-153 items 1–6 complete" but does not answer any of the concrete findings. Please do this bounded review task in the existing session/worktree:

1. Give the full 40-character local HEAD SHA for 7491d8c1d4 and the output of git show --stat --oneline --decorate HEAD, plus the exact full diff of that local commit (it is reported to be only ACCEPTANCE_MATRIX.md). Do not push it yet.
2. Re-check the current origin/wip/xaverius tip and inspect each of the five blocker locations from my review against BOTH local HEAD and origin HEAD. For every finding, report: fixed/not fixed, exact file+line, relevant code path, and a test name proving the behavior. "Verified complete" without this mapping is not an answer.
3. Specifically resolve the apparent contradictions:
   - registry.c snapshot callback output vs freeing/copying dev->snapshot.data;
   - active preferred binding replacement and published-device guard;
   - exact-size/unknown-version behavior for Registration and Ops descriptors;
   - whether AddResource stores a PhysicalDevice pointer directly and what DIXDestroy releases;
   - RegistryFini/ModuleRoleFinalize admission, drain and quiescence ordering.
4. Reconcile the publication API with normative spec §7: XaveriusRegistryResult XaveriusPhysicalDevicePublish(device, device_generation, out_xid).
5. Re-run the scope audit from the actual changed-path list. The remote tree currently contains Xext/xaverius/screen.c and dispatch.c; explain their provenance and ownership. Do not delete or rewrite them. Keep them out of the W0-153 final commit unless separately authorized.
6. Supply raw relevant test output and build command/result, plus the exact commit SHA tested. If tests are local-only, say so; do not describe them as CI.
7. Correct ACCEPTANCE_MATRIX.md to PENDING/NOT ACCEPTED in the local worktree while blockers remain. Do not push the doc correction until the diff is reviewed.

Keep the worktree and sole-writer ownership. No squash/rebase/reset/force-push/cherry-pick. Do not declare W0-153 complete. Reply with the requested evidence and next bounded action; then continue fixing any confirmed blockers in scope. The final consolidation remains exactly one final registry commit only after independent review.
