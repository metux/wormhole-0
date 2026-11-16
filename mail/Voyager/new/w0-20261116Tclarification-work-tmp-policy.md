# Clarification: temporary files belong under _WORK_

This clarifies the temporary-directory policy audit requested in the previous message.

The intended existing workspace directory is exactly `_WORK_` (uppercase, with leading and trailing underscores). It already exists in the workspace; do not introduce a separate `work/` directory or rename it.

Please assess this as the target policy:

- Set `TMPDIR`, `TMP`, and `TEMP` to the absolute path of this ship's `_WORK_` directory in every agent environment, for both console and background ships.
- Agents may create their own subdirectories under `_WORK_` as needed (for example, `tmp/`, build-specific scratch directories, downloads, etc.).
- Agents are responsible for removing temporary files and subdirectories they no longer need. Cleanup must be scoped to their own temporary artifacts; never delete another ship's data or active work.
- Tools that hard-code `/tmp` should, if feasible, see a sandbox-internal `/tmp` redirected/mounted to this ship's `_WORK_`. The host's shared `/tmp` must not become generally accessible merely to avoid permission asks.

In the audit, verify how and where these environment variables can reliably be set across all launch paths, whether hard-coded `/tmp` can be redirected safely, and what cleanup lifecycle is appropriate. Distinguish existing behavior from recommendations. Remain READ-ONLY; do not implement policy changes. Continue in the same existing session/worktree; do not restart or duplicate the ship. Send task_progress and task_done when appropriate.
