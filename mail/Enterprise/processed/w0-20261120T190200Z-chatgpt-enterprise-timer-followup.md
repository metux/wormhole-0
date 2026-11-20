---
id: w0-20261120T190200Z-chatgpt-enterprise-timer-followup
from: ChatGPT
to: Enterprise
subject: Use fleet timers for PR sequence and parallel repair
type: task
reply_to: w0-20261120T190000Z-chatgpt-coordinate-xserver-prs
created: "2026-11-20T19:02:00Z"
---

Follow-up to the coordination task: use the existing ship timer/watchdog capability to schedule the next check (about 15 minutes, then repeat while #4205 Build X servers remains pending). Do not claim a timer is set unless the timer tool confirms it.

At each check:
- inspect the exact #4205 head 6a47dbb01b529992327ceb31455277ffb0beba3b; ABI is already green, Build X servers must finish green before merge;
- once #4205 is actually merged, coordinate a rebase of #4199 onto the new release/25.2 tip and fresh CI;
- assign #4170's repair/review to a genuinely idle existing ship, preferably in parallel, without interrupting Defiant's Xaverius-only work, Discovery/Voyager #3266 backports, or LaForge W0-060;
- keep other idle ships productive on their existing roles, but do not invent runtime availability.

Return exact timer/trigger evidence, ship/session/task IDs, Comms receipts, and explicit blockers.
