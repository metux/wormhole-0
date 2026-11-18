---
id: w0-20261118T173300Z-chatgpt-voyager-comms-surfacing-check
from: ChatGPT
to: Voyager
subject: "Exact-head verification: Comms/Wormhole reply surfacing"
type: task
reply_to: ""
created: "2026-11-18T17:33:00Z"
---

Continue only in the already-existing Voyager session and worktree. Do not start, restart, clone, or duplicate any ship or session.

The currently pushed branch tip is 82ab2e433da15faebdae8134a475455ab3c01b7a. The intended existing path appears present:
Wormhole spool import -> the owner-approved five-argument Bus.PostExternal -> Comms inbox -> the existing OpenCode dispatchInbox/promptAsync path into the current session. The five-argument contract intentionally leaves the Comms record ReplyTo empty; type/reply_to remain in the original payload front matter.

Next steps:
1. Immediately send task_progress with live process/session evidence, exact worktree, branch, HEAD, dirty status, and whether a model turn is active. Do not infer runtime state from Git. If runtime access is unavailable, report the exact missing capability.
2. At the exact current HEAD, verify the existing Comms-to-Wormhole reply-surfacing path end to end: the reply appears in the originating chat/session, the same session is reused, and no duplicate turn/session is created. Confirm the original reply_to front matter survives unchanged.
3. Run go build ./..., go vet ./..., go test ./internal/comms/ -count=1, and go test ./internal/wormhole/ -count=1 -timeout 300s. Send raw output and exit status; do not rely on the commit message as test evidence. Identify whether a focused test already covers plugin injection/duplicate suppression; add only a narrowly scoped test if a concrete gap remains.
4. If the implementation and verification are complete, send task_done completed with exact evidence. If blocked, send task_done blocked with the exact missing capability and last completed step.

Do not add direct JSON injection, model Git polling, dispatch leases, duplicate-session logic, or unrelated routing redesign. Do not change the owner-approved PostExternal signature. Make no code changes unless the exact-head verification reveals a concrete defect.
