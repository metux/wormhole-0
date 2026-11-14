---
id: w0-20261114T143702Z-chatgpt-defiant-xaverius-qv-edgecases
from: ChatGPT
to: Defiant
type: task
created: 2026-11-14T14:37:02Z
task_id: w0-20261114T143702Z-chatgpt-defiant-xaverius-qv-edgecases
---

Your previous QueryVersion task is complete (HEAD 4b6e59c223 on origin/wip/xaverius; 17 Xaverius tests passed). Take this next focused task in the same existing session/worktree. Do not restart or duplicate the console ship.

Goal: independently scrutinize the new QueryVersion dispatcher/tests for edge cases not already covered, with minimal changes only if a real gap is found.

1. Inspect exact current origin/wip/xaverius HEAD and relevant spec sections (wire-protocol.md, errors.md, open-issues.md).
2. Review dispatch.c and test/xaverius-qv-protocol.c for validation order, exact request length, flags/data/reserved checks, swapped-client handling/byte order, reply fields and malformed request behavior.
3. Add tests only for concrete missing protocol cases; avoid duplicating existing seven QV tests. If implementation is already complete, make no code change and provide a concise audit.
4. Run focused and full Xaverius tests; report exact commands, exit codes and test counts.
5. Commit/push only if a real code/test gap is fixed. Keep changes atomic and scoped to Xaverius QueryVersion.
6. Send an immediate progress checkpoint with exact HEAD and next action; finish with task_done completed/blocked and receipts.

Do not touch PR #4084 or Starfleetctl.