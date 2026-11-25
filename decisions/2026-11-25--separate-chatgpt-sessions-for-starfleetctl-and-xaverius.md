# Decision: separate ChatGPT sessions for Starfleetctl and XAverius

**Date:** 2026-11-25  
**Status:** Active owner decision

## Decision

Starfleetctl and XAverius/Xserver are separate workstreams and must be handled in separate ChatGPT sessions.

- A session explicitly scoped to **starfleetctl** works only on the Starfleetctl repository, its issues/PRs, Comms/event-pipeline implementation, and directly relevant Wormhole coordination.
- A session explicitly scoped to **XAverius** works only on XAverius/Xserver implementation, specifications, tests, and its directly relevant coordination.
- Do not silently switch workstreams, mix their backlogs, or continue both merely because the user says "weiter".
- Interpret "weiter" within the scope of the current session. Refresh that workstream's live state and continue its next concrete task.
- If the user explicitly changes the session scope, follow the new instruction. Otherwise, keep the existing scope.
- Cross-workstream dependencies or handoffs may be recorded in Wormhole, but do not perform the other workstream's implementation/review in the current session.

## Rationale

The owner deliberately handles Starfleetctl and XAverius in separate ChatGPT sessions to keep context, task ownership, technical review, and execution boundaries clear. Combining the strands creates ambiguity about which backlog "weiter" refers to and risks cross-session interference.

## Consequences

- New ChatGPT sessions must read this decision through the project instructions/bootstrap before acting.
- The previous `SESSION_BOOTSTRAP.md` instruction to continue both strands on every "weiter" is superseded.
- The current session scope is established by the owner's explicit session instruction/context, not inferred from unrelated open tasks in the fleet.
- Keep shared durable facts and cross-workstream dependencies in Wormhole, while preserving separate session execution.
