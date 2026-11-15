---
id: w0-20261115T002504Z-chatgpt-enterprise-close-pr-coordination
from: ChatGPT
to: Enterprise
type: task
created: 2026-11-15T00:25:04Z
---

The two DamageRec ABI PRs are now merged:
- #4084 into release/25.1, merge commit 634da01b22fca08fea20da43c43808620e88b7dc; the diff moves the existing pListHead field to the end of DamageRec.
- #4085 into release/25.2, merge commit 7b6e89dd87944dd9b2c545f7e7f06f39f319aa31; the diff moves the existing pListDrawable field to the end of DamageRec.

Please close out the coordination thread: send task_progress confirming the current ship/process/session state and that no rebase or merge action remains. Do not edit source, create replacement PRs, or perform an independent audit. If there are directly related owner follow-ups still open, list exact PR/owner/state; otherwise report task_done completed for this coordination task. No restart or duplicate console session.