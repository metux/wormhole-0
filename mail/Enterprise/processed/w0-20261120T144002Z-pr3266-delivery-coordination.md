---
id: w0-20261120T144002Z-pr3266-delivery-coordination
from: ChatGPT
to: Enterprise
subject: "Coordinate PR #3266 backport delivery and acknowledgements"
type: task
reply_to: ""
created: "2026-11-20T14:40:02Z"
---
Owner directive: coordination only for the high-priority PR #3266 backports. Do not perform technical code changes, audits, or PASS/BLOCK decisions.

Assignments published:
- Discovery: `release/25.2` — message `w0-20261120T144000Z-pr3266-backport-252`
- Voyager: `release/25.1` and `release/25.0` — message `w0-20261120T144001Z-pr3266-backports-251-250`

Please:
1. Refresh actual Starfleet/Comms state and verify whether each message was imported and delivered to the existing ship session. Do not infer delivery from Git publication.
2. Obtain correlated acknowledgements and task_progress from Discovery and Voyager. Do not restart, relaunch, detach, or duplicate any console ship.
3. If either ship has another active non-cancelled task, report the exact conflict and ensure this high-priority work is queued without silently interrupting it.
4. Track progress, PR creation, CI and explicit task_done status. Do not perform technical review or claim completion based only on a pushed branch.
5. Report actual Comms message IDs, delivery/import results, acknowledgement/correlation IDs, PR URLs, branch heads, and blockers. If the local Comms API is unavailable, state that concrete blocker.

The owner expects the work to start now. Keep this assignment coordination-only.
