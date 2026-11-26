# Decision: sustained fleet execution without microstep prompting

**Date:** 2026-11-26  
**Status:** Active owner decision

## Decision

Across all project workstreams and ChatGPT sessions that coordinate the fleet, an owner instruction such as **"weiter"** means: independently inspect the current state and advance the active authorized work as far as safely possible. It does **not** mean perform only one microstep and then wait for another owner prompt.

Ships should be instructed to execute the complete currently authorized task or acceptance sequence as one sustained work cycle, rather than yielding after each small edit, test, commit, or progress update.

## Operating rules

1. **Sustained execution:** State the full bounded objective and acceptance sequence in the assignment. The ship should continue through all authorized steps without waiting for a new prompt after each microstep.
2. **Progress checkpoints are not yield triggers:** Use `task_progress` and correlated Comms reports to make progress observable. A checkpoint is not a request to stop; after reporting it, continue the next authorized step.
3. **Stop only for a real boundary:** Stop when the task and its acceptance criteria are complete, a genuine external/owner decision is required, a safety/ownership/specification conflict blocks progress, or the runtime/environment forcibly ends the turn.
4. **Forced turn boundary:** Before yielding, publish a precise checkpoint: exact task/message IDs, repository/branch/HEAD and parent, dirty paths and commit state where available, verified tests, unresolved acceptance criteria, blocker, and the next concrete action. The existing assignment should be resumable without reconstructing intent.
5. **"Weiter" means audit plus action:** Refresh task ownership, repository state, branches, diffs, PRs, CI, Comms lifecycle and actual ship activity. Distinguish publication, import, injection, acknowledgement, model-turn execution, progress, and completion. Then take the next bounded action rather than merely narrating status.
6. **No false background monitoring:** Do not imply continuous polling between owner messages unless a real scheduler/process is configured and verified.
7. **Coordination roles and ownership remain binding:** Sustained execution does not authorize restarting/relaunching ships, duplicate assignments, reassignment, concurrent writers, bypassing review/merge authority, or expanding task scope. Use coordination ships for observation/reporting only where that is their assigned role.
8. **Evidence-based completion:** A processed message, status claim, local test claim, or progress percentage is not independently verified completion. Check the relevant repository diff, exact commit, tests, and CI where access permits.

## Scope

Applies across all fleet-coordination workstreams and ChatGPT sessions. It changes the expected execution cadence and interpretation of progress checkpoints only; it does not change technical authority, task scope, repository ownership, review requirements, or ship lifecycle permissions.

## Rationale

The owner should not need to repeatedly type "weiter" to make a ship perform the next small step of an already authorized task. The fleet should work in coherent, sustained sequences while preserving visibility, safety, and resumability.
