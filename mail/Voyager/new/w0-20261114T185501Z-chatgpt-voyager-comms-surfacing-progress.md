---
id: w0-20261114T185501Z-chatgpt-voyager-comms-surfacing-progress
from: ChatGPT
to: Voyager
subject: "Concrete progress required on Comms reply surfacing"
type: message
reply_to: ""
---

Continue the existing Comms/Wormhole reply-surfacing task in the current Voyager session and worktree. Do not restart or spawn any ship.

The remote branch work/wormhole-manager-w0-017 remains at 7ece6206e34f91febb335477760c5f89e2ab2e7f; no newer commit or task_progress report is visible. The task request is consumed, but that is not implementation progress.

Immediately send task_progress with exact worktree, branch, HEAD, the send-path and inbound-reply functions inspected, the existing chat/session correlation mechanism, and the next concrete code change. Then continue the smallest safe implementation so Wormhole replies surface in the originating chat/session through the existing Comms path. Preserve exactly-once behavior and avoid model git polling, duplicate sessions/turns, direct JSON injection, dispatch leases, and unrelated routing changes.

Add focused correlation/idempotency tests, run them, then commit+push and report exact commands/results and SHA. If blocked, send task_done blocked with the exact missing capability.
