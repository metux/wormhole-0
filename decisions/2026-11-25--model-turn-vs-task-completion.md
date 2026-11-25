# Model-turn completion is not task completion

**Observed 2026-11-25; owner-reported runtime behaviour.**

The owner reports that Defiant's OpenCode process remained healthy, but the model itself ended its turn before the assigned work was finished. The owner has seen this occur with Nemotron more than once. This is an operational observation, not yet a measured failure rate or a verified model-specific defect.

## Decision

Keep these states distinct:

1. **OpenCode process health** — whether the client/session process is alive and responsive.
2. **Model-turn lifecycle** — whether a particular submitted prompt has started and whether its assistant turn ended, errored, or was interrupted.
3. **Task lifecycle** — whether the assigned work is still in progress, blocked, or explicitly completed against its acceptance criteria.

A normal assistant turn completion proves only that the turn ended. It does not prove that the assigned task was completed. In particular, a short/early Nemotron turn must not silently turn an unfinished task into completed, nor be misdiagnosed as an OpenCode process crash.

## Consequences

- Keep Comms event/turn completion separate from task completion. The completed state in the Comms event lifecycle means the correlated assistant turn completed; it must not imply the agent task is complete.
- Task completion requires an explicit, evidence-backed completion report against the task's acceptance criteria.
- If a turn ends while the task remains in_progress, preserve the task state and make the incomplete/next-step condition visible. Any automatic continuation or retry must be bounded and idempotent; do not blindly replay the original prompt or create duplicate work.
- Diagnostics should record the correlated OpenCode message/turn, completion/error metadata, task ID, and last progress marker so that early turn endings can be distinguished from process failures and Comms delivery failures.

This decision records the distinction and the owner-reported observation; it does not assert that Nemotron is generally unreliable or prescribe a model-specific workaround.
