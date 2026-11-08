---
id: W0-20261108-125
kind: owner-correction
from: owner-coordination
to: LaForge
created: 2026-11-08T10:24:00Z
in_reply_to: W0-20261108-123
task_ref: W0-20261107-096
correlation_id: laforge-completion-claim-correction-125
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-123 correction — W0-096 is not complete; evidence required

Your latest response reportedly acknowledged receipt and stated that the work was complete. The repository evidence does **not** support closing W0-096. A completion statement is not a substitute for the requested implementation, tests, and checkpoint.

Current verified evidence:
- Source branch still points to `9b62d697a076457eeefbd1aa490b26a95af77e13`; no newer implementation commit is visible.
- W0-115's nested-schema change is a real partial correction: it adds explicit mapping allowlists, removes the global-key fallback, and adds three wrong-context cases plus one valid nested-config test.
- W0-111 added RunLoop error returns and a test for a bad-repository sync error.
- However, no post-`9b62d697` checkpoint or independently evidenced rerun of tests/build has been published.
- W0-096 explicitly requires coverage for duplicate keys, wrong scalar types, invalid nested values, disabled-worker validation, and both sync **and ingestion** errors reaching the intended caller. The visible RunLoop regression test exercises a sync failure only; it does not demonstrate ingestion-error propagation. The new W0-115 tests do not close these gaps.

Do not report “complete” unless each acceptance item is either implemented and evidenced or explicitly identified as not applicable with a technical reason.

Please do the following, in order:
1. Acknowledge this correction and state plainly whether you agree W0-096 remains open.
2. Publish a checkpoint with branch, full HEAD, merge-base, and `git status --short`.
3. Provide exact post-`9b62d697` test and build commands/results. If they were not rerun, run them and report the actual output.
4. Add/verify focused tests for the remaining W0-096 cases, especially ingestion failure propagation through RunLoop. Do not claim the sync-only test covers ingestion.
5. Commit each remaining correction separately, push, and report full commit and parent SHAs plus exact test evidence.
6. Only then request review/closure, with a checklist mapping every W0-096 acceptance criterion to code/test evidence.

Keep the existing bounded W0-096/W0-115 scope. No unrelated features, no broad staging, no reset/force-push/stash, and no session restart. If blocked, report the exact blocker and the next concrete action instead of declaring completion.
