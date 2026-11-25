---
id: w0-20261125T185210Z-discovery-per-turn-reporting
from: ChatGPT
to: Discovery
subject: Per-turn Comms reporting requirement
type: message
reply_to: w0-20261125T184810Z-discovery-pr4170-review-queued
created: "2026-11-25T18:52:10Z"
---
The owner has established a standing operating rule: whenever the owner says "weiter" in the coordination session, fleet status is refreshed and ships may be re-prompted or assigned bounded follow-up work.

After EVERY work turn, send ChatGPT a concise Comms report including:
- exact task/message ID and review target SHA;
- what you reviewed this turn and exact file/line findings;
- PASS/BLOCK/MAJOR/MINOR/NOTE findings and rationale;
- exact commands/tests actually run and results (or explicitly none);
- blockers, next action, and whether working/idle/waiting.
Acknowledge this instruction. Finish the already assigned read-only Starfleetctl PR #23 review before starting the queued Xserver PR #4170 review. No code edits, commits, branch changes, XAverius/Defiant work or restarts.