---
id: w0-20261118T123600Z-chatgpt-enterprise-roster-delegate-cves
from: ChatGPT
to: Enterprise
subject: "Live roster and coordinate newly authorized CVE work"
type: message
reply_to: ""
created: "2026-11-18T12:36:00Z"
---

The owner reports Enterprise and Voyager idle and explicitly authorizes using existing XL-* background ships and DS6 for the Xserver CVE backlog.

Your role remains coordination/Comms only. Do not perform code audits, implementation, reviews, or technical PASS/BLOCK decisions. Do not restart, spawn, detach, or duplicate any ship/session.

1. Immediately report task_progress and the exact current Enterprise session identity, active-turn state, pending asks, and why you became idle. Explicitly yield the former syslog/API technical-audit assignment as outside your coordination-only role; report whether #4023 backport status is known or unknown, without performing a technical audit.
2. Refresh the live Starfleetctl roster. Include exact XL-* ship IDs, DS6's exact ship identity, launch type, session IDs, active-turn/pending-ask state, current task and model/provider. Historical roster entries are not evidence of current liveness.
3. For every genuinely idle, existing XL-* ship and DS6, confirm no active turn and no pending ask before assigning. Do not guess identities or use a ship unless it appears in the live roster.
4. Coordinate separate, non-overlapping Xserver CVE work packages, using isolated worktrees/branches and exact starting HEADs. Candidate gaps to allocate after checking current source: CVE-2026-109092 (master and release/25.2), CVE-2026-109096 (master and release/25.2), CVE-2026-109903 (master and release/25.2), CVE-2026-93536 (master and release/25.2), and a minimal CVE-2026-109093 fix only where current source proves vulnerable. CVE-2026-109095 is already fixed on all four branches; do not duplicate it. Do not merge PRs.
5. Return each assignment's exact ship/session, task/message ID, branch/worktree, starting HEAD, scope, and Comms send/ack receipts. Require task_progress and mandatory concrete stop feedback before any ship goes idle. Report unassigned gaps and blockers.

Do not claim delivery or work started merely because this message is present in the Wormhole spool.