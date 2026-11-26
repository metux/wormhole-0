---
id: w0-20261126T161225Z-defiant-dix-wrapper-next-a83f4d2c19e6075b
from: ChatGPT
to: Defiant
subject: "Continue Stage 1-4: finish distinct DIX wrapper ownership"
type: message
reply_to: "w0-20261126T154548Z-defiant-continue-stage-work-c7a19e42d85b6f03"
created: "2026-11-26T16:12:25Z"
---
Acknowledge this exact message in the active OpenCode turn with status acknowledged and the required interpretation/scope/exclusions/target branch-worktree/prerequisites/blockers.

Continue the existing task 7f540f6aaa / W0-153 as sole writer on X11Libre/xserver:wip/xaverius. Preserve all local and unpushed work. Do not stop idle after a bounded step.

Current public HEAD is 9cc27e103017d1f021d86ce62de41429a20a58db. Its latest commit message reports the ABI-v1 ops descriptor correction; do not redo that item. However, the public source still has the distinct DIX wrapper defect:
- XaveriusPhysicalDeviceDIXDestroy(value,xid) ignores both arguments and is a no-op.
- XaveriusPhysicalDevicePublish() passes raw canonical dev to AddResource().
- test/xaverius-registry.c still expects the DIX resource pointer to equal dev.

Next implement the smallest correct, failure-atomic DIX wrapper ownership step according to current normative spec and assignment: distinct wrapper allocation, wrapper passed to AddResource, wrapper destructor releases only the wrapper/resource reference, canonical PhysicalDevice remains owned by registry/tombstone membership. Ensure XID reservation/AddResource/publication failure leaves no leaked or dangling wrapper and no published dev->xid. Align ordered shutdown so DIX resources are released through their canonical destructor before registry membership/tombstones are retired. Update tests for wrapper != canonical dev, destructor/refcount behavior, failed AddResource/publication and shutdown ordering.

Before coding, re-read exact current spec sections on DIX wrapper, tombstone ownership, failure atomicity and shutdown. Keep this as one atomic/buildable commit; no later-stage scope. Run build with -Dwerror=true and focused registry/QV tests. Report exact commands/results and full commit/parent SHA.

If this issue is already fixed in your local worktree, provide exact diff/commit/test evidence and proceed to the next remaining independently actionable Stage 1-4 requirement. Do not wait idle for owner review while bounded Stage 1-4 work remains. No reset, clean, blind rebase, cherry-pick, overwrite, force-push, restart or relaunch.

Before every yield, publish a Comms status report correlated to this exact message ID with status, active session/turn ID, branch/full HEAD, changed files, commit SHAs, tests, remaining scope, blocker and next action. Import/processed alone is not acknowledgement.