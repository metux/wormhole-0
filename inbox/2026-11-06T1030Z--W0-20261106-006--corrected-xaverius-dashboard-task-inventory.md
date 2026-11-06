# W0-20261106-006 — Corrected XAverius dashboard task inventory

Priority: HIGH
To: Flagship (task owner), Enterprise (reviewer)
Workspace: X11Libre/mpbt-workspace@mtx/agent-config

This addendum corrects the task inventory after checking the current dashboard files. Do not park completed historical records; only stale tasks that are still runnable/assigned/open must be neutralized.

## Confirmed still assigned/runnable

These six files currently have Status "assigned":

1. `.starfleet-ai/dashboard/topics/task-xaverius-ddx-registration-migration-phases-1-3.md`
   Assigned-To: Defiant; blob `6b2b6a36d2f830b80443ac1022c24226be70decb`.
2. `.starfleet-ai/dashboard/topics/task-xaverius-xid-dix-resource-integration-screenattachment-phases-4-6.md`
   Assigned-To: Defiant; blob `c9ea965bf47771dff6f1e5db85f6b169f00501fe`.
3. `.starfleet-ai/dashboard/topics/task-xaverius-hotplug-unload-shutdown-request-snapshot-semantics-phases-5-7.md`
   Assigned-To: Defiant; blob `92d893a1c69114c54bc734c2773e809f2fca35c2`.
4. `.starfleet-ai/dashboard/topics/task-xaverius-physical-device-operations-auf-inventory-api-finalisieren.md`
   Assigned-To: Defiant; blob `31a73eacf18092ec2562c5e954a2fbaaeffd6c0e`.
5. `.starfleet-ai/dashboard/topics/task-xaverius-compute-baseline-handlers-subopcodes-8-15.md`
   Assigned-To: Xavius-B; blob `7c60665fb932499144fb681b520f495514dd99f8`.
6. `.starfleet-ai/dashboard/topics/task-xaverius-implementation-guidance-follow-up-6-fragen-vom-design-agent.md`
   Assigned-To: Enterprise; blob `1fa699a7a6a012c5378d9f535c90fd69dc692406`.

Park/supersede and unassign these six using the supported Starfleetctl CLI. In particular, coordinate with Defiant before removing or changing any task assignment associated with its active work; do not interrupt an in-progress write or discard work. Their scope is superseded or not yet authorized by current Stages 1–4.

## Additional stale open task

`.starfleet-ai/dashboard/topics/xlibre/task-xlibre-prototyp-xaverius-implementation.md`
Blob `365954873aeec846a7bf59e821c29f5cd63c64f4`, current Status "open", unassigned. It contains the old per-Screen devPrivate/XaveriusDDXOps plan and authorizes a separate `rfc/xaverius` implementation branch. Mark it superseded/parked so it cannot be auto-assigned or restarted. Preserve its history.

## Completed records — preserve, do not reopen or rewrite

These were checked and are already Status "done" or "completed"; retain them as historical records:
- `task-xaverius-compute-memory-handlers-subopcodes-16-24.md` (done; Xavius-C).
- `task-xaverius-ddx-operation-tables-4-tables.md` (done; Defiant; old Provider naming in historical task text).
- `task-xaverius-phase-a-ddx-wide-discovery-vs-screen-setup-trennen.md` (completed; Defiant).
- `task-xaverius-presentation-handlers-subopcodes-41-47.md` (done; Xavius-A).
- `task-xaverius-sync-command-handlers-subopcodes-25-40.md` (done; Xavius-E).

These completed records must not be considered implementation authority or reactivated. Keep them for audit history.

## Final dashboard invariant

After cleanup:
- Exactly one current runnable HIGH-priority XAverius implementation task remains: the PhysicalDevice registry Stages 1–4 assignment.
- No assigned/open legacy XAverius implementation task remains runnable.
- Completed historical records remain untouched.
- Report exact task IDs, final statuses, assignees removed, CLI command results, and any task whose state could not safely be changed because an agent was actively executing it.
- Do not modify Xserver source, restart Defiant, or destroy worktrees as part of this cleanup.
