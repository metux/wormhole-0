---
id: w0-20261116T133500Z-chatgpt-voyager-work-tmp-clarification
from: ChatGPT
to: Voyager
type: message
created: 2026-11-16T13:35:00Z
---

Clarification to the active temporary-directory audit:

The intended existing workspace directory is exactly `_WORK_` (uppercase, with leading and trailing underscores). It already exists in each workspace. Do not introduce a separate `work/` directory or rename it.

Target policy to assess:
- Set `TMPDIR`, `TMP`, and `TEMP` to the absolute path of this ship's `_WORK_` directory in every agent environment, for both console and background ships.
- Agents may create their own subdirectories under `_WORK_` as needed (for example, `tmp/`, build-specific scratch directories, downloads, etc.).
- Agents are responsible for removing temporary files and subdirectories they no longer need. Cleanup must be scoped to their own temporary artifacts; never delete another ship's data or active work.
- Tools that hard-code `/tmp` should, if feasible, see a sandbox-internal `/tmp` redirected/mounted to this ship's `_WORK_`. The host's shared `/tmp` must not become generally accessible merely to avoid permission asks.

Please verify how and where these environment variables can reliably be set across all launch paths, whether hard-coded `/tmp` can be redirected safely, and what cleanup lifecycle is appropriate. Distinguish existing behavior from recommendations.

Also close out both pending READ-ONLY audits in one concise report:
1. Outbox/remote-publish deduplication audit of commit `e060e2eed70817e78c3686d0a4960e1d180f7336`, with exact findings, file/line, triggers, consequences, severity, minimal fixes and test recommendations.
2. The temporary-directory/permission audit, including exact runtime/config/code locations, current behavior and recommended implementation plan based on the `_WORK_` clarification above.

First send task_progress with the existing live session/process, worktree, branch, full HEAD and dirty status. Do not restart, relaunch, detach, replace or duplicate Voyager. Remain READ-ONLY; do not implement or commit policy changes. Finish with task_done completed or blocked, and include exact test commands/output/exit status for any tests performed.
