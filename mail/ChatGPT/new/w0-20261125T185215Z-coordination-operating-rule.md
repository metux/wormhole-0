---
id: w0-20261125T185215Z-coordination-operating-rule
from: ChatGPT
to: ChatGPT
subject: Standing operating rule for coordination session
type: message
created: "2026-11-25T18:52:15Z"
---
Owner instruction:
Whenever the owner says "weiter" in this ChatGPT coordination session:
1. Refresh current fleet/Wormhole state and active task ownership.
2. Inspect the progress reports received since the previous turn.
3. Re-prompt ships that have not acknowledged, are stuck, or have no recent report, checking first whether the existing message is imported and whether a reply already exists.
4. Assign suitable bounded tasks to genuinely idle Enterprise/Discovery where useful; never duplicate active work.
5. Continue Voyager's existing Xserver PR #4170 task and verify exact-head code/CI/review evidence.
6. Do not contact or inspect Defiant; its XAverius ChatGPT session owns it.
7. Require every ship to send a Comms report to ChatGPT after EVERY work turn. Reports must correlate exact task/message IDs, branch/head where relevant, changes, tests, blockers and next action. Imported/processed alone is not proof of work.
8. Report to owner what was verified, what was merely imported, who is working/idle/waiting, and any new tasks/messages issued.
This is a coordination-turn protocol, not a claim of autonomous background monitoring.