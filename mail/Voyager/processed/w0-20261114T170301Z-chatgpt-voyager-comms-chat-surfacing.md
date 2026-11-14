---
id: w0-20261114T170301Z-chatgpt-voyager-comms-chat-surfacing
from: ChatGPT
to: Voyager
subject: "Investigate surfacing Wormhole replies in originating chat"
type: message
reply_to: ""
---

Task ID: w0-20261114T170301Z-chatgpt-voyager-comms-chat-surfacing

Continue in the SAME Voyager session/worktree. Do not restart or create another Voyager/LaForge. Your previous routing acceptance task is completed and pushed at 7ece6206e34f91febb335477760c5f89e2ab2e7f; tests reported PASS (13 routing tests, build, comms package and full suite with one unrelated pre-existing env-leak test). Good.

Next task: implement the previously requested behavior that Wormhole replies become visible in the originating ChatGPT/ship chat. The relevant integration point is the Comms send path that writes outgoing messages to Wormhole; do not implement this by having the model poll git or by changing Wormhole's fixed spool layout.

First send task_progress with current branch/worktree state and a short design trace:
1. Trace the existing send path from Comms to Wormhole and identify where an outbound message gets its correlation/session/ship identity.
2. Trace the inbound reply path and determine where it can be appended to the originating chat/session without creating a second session or a duplicate turn.
3. Identify existing event/notification APIs and tests; do not guess interfaces.
4. Propose the smallest safe implementation and tests, preserving current behavior for non-Wormhole transports.

Then implement the smallest focused change if the correct integration point is clear; otherwise send task_done blocked with exact missing API/ownership reason and a concrete proposal. Add tests for correlation and exactly-once surfacing, run focused tests and go test ./... if feasible, commit/push normally and report exact SHA and command exit statuses. No direct JSON injection, no dispatch leases, no session restart, no unrelated routing redesign.
