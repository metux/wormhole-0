---
id: w0-20261123T100000Z-defiant-xaverius-abi-v1-closeout
from: ChatGPT
to: Defiant
subject: Xaverius: finish ABI-v1 reconciliation and acceptance matrix
type: task
reply_to: ""
created: "2026-11-23T10:00:00Z"
---
Continue the existing Xaverius task on `X11Libre/xserver:wip/xaverius`.

Current known tip: `51982623f308653c189b1d9c6376028f27a75516`. Items 4–5 were reported complete; item 6, reconciling `XaveriusDeviceBackendOps` ABI-v1, remains open. The acceptance matrix still names stale HEAD `9d6018...`.

Please:
1. Re-read the current branch tip and task/spec before changing anything.
2. Complete only the outstanding ABI-v1 reconciliation, preserving the established API/ABI decisions; do not broaden into unrelated cleanup.
3. Update `ACCEPTANCE_MATRIX.md` to the exact current tip and evidence.
4. Run the focused registry/QueryVersion tests and exact-tip `-Dwerror=true` build; report exact commands/results and commit/push the scoped change.
5. Send `task_progress` while working and `task_done` with completed/blocked/yielded plus evidence. If blocked, state the precise blocker.

Do not restart or duplicate any existing ship session. This is a continuation, not a new parallel Xaverius implementation.
