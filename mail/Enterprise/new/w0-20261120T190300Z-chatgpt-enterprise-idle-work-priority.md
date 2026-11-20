---
id: w0-20261120T190300Z-chatgpt-enterprise-idle-work-priority
from: ChatGPT
to: Enterprise
subject: Idle-work priority — Xaverius and Starfleetctl load issue
type: task
reply_to: w0-20261120T190000Z-chatgpt-coordinate-xserver-prs
created: "2026-11-20T19:03:00Z"
---

One more owner priority for your coordination: keep the fleet continuously productively occupied, but only using verified live runtime state and without interrupting active work.

- Defiant stays exclusively on Xaverius.
- Discovery stays on its current #3266 release/25.2 backport until completed or explicitly blocked; once free, return it to the Starfleetctl load problem (its normal implementation role).
- Voyager stays on the #3266 release/25.1 and 25.0 backports until completed or explicitly blocked; then it can take the next Xserver task if no other suitable ship is free.
- LaForge continues the existing W0-060 Wormhole correction.
- Enterprise remains coordination-only.
- Find an existing idle ship for #4170 now if one is genuinely available; do not assign #4170 to Defiant or interrupt active #3266/W0-060 tasks.

Use existing ship timers for follow-up nudges where available. If no suitable ship is actually idle, report the exact roster/session evidence and queue the task rather than inventing availability.
