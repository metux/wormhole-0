---
id: w0-20261116T130000Z-chatgpt-voyager-tmp-policy-audit
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T13:00:00Z
---

New bounded READ-ONLY audit, in the SAME existing Voyager session/worktree. Do not restart, relaunch, detach, replace or duplicate the session. Do not edit, commit or push.

First close out the previous Outbox audit with its promised report: exact reviewed commit, current session/worktree/branch/HEAD, findings with file/line, concrete trigger/consequence, severity, minimal fix and focused test recommendations; clearly distinguish confirmed defects from theoretical risks. If the audit is genuinely incomplete, say so and finish it before switching tasks.

Then audit Starfleetctl's temporary-directory and permission policy, especially why console ships still hit permission-ask when agents access /tmp. Inspect the actual runtime permission/sandbox configuration, console vs background ship launch paths, environment propagation (TMPDIR/TMP/TEMP), SOP/prompt sources, and any tool/build subprocess behavior. Do not infer behavior from docs alone.

Compare these options:
1. hard-deny global /tmp and route all temp use to a per-ship private scratch directory;
2. hard-allow /tmp globally;
3. expose an isolated per-ship /tmp mount/namespace while keeping host /tmp inaccessible.

Recommend the narrowest safe policy that eliminates interactive asks for routine temp use. Identify whether per-ship scratch can be outside shared project workspaces, how path canonicalization/symlink traversal is handled, and what happens when tools hard-code /tmp. Provide exact config/code locations, current effective behavior, attack/collision risks, minimal implementation plan and tests (console and background ships, parallel ships, hard-coded /tmp, symlinks, cleanup/restart). READ-ONLY: no changes until the user chooses a policy. Send task_progress before investigation and task_done completed/blocked with evidence.
