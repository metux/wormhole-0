---
id: W0-20261108-101
kind: implementation-followup
from: owner-coordination
to: LaForge
created: 2026-11-08T00:18:00Z
in_reply_to: W0-20261107-096
task_ref: W0-076
correlation_id: laforge-config-validation-hardening-101
---

# W0-096 follow-up: harden committed config validation, then continue error reporting

Good progress: the first W0-096 milestone is now committed and pushed on work/wormhole-manager-w0-017 at 3bbca73bc4ff35ad4acc862e2efd0011e1d6eb8d (3 commits ahead, 0 behind master). The checkpoint W0-098 was taken before this commit, so treat its dirty-worktree statement as superseded by the later branch commit; do not infer current dirty status from that old report.

An independent source review of the pushed code found these remaining correctness gaps to address before considering config validation complete:

1. LoadConfig decodes a map[string]yaml.Node and calls node.Decode(cfg), which does not enable KnownFields. Unknown keys at the wormholes root and nested levels are silently ignored. Decode through a strict yaml.Decoder with KnownFields(true), using a wrapper struct for the top-level wormholes key; reject missing/wrong top-level wrapper and unexpected keys. Keep the canonical internal/config.WormholesConfig schema.
2. ValidateConfig accepts zero and negative durations because it checks only time.ParseDuration errors. In particular sync.interval: 0s reaches time.NewTicker(0) and panics. Require positive interval, clone_timeout, and command_timeout values.
3. ValidateConfig dereferences cfg without a nil check. Return a useful error for nil config.
4. Add focused table-driven tests for unknown top-level keys, unknown nested sync/repository/incoming/outgoing keys, wrong scalar types (including invalid booleans), missing wrapper, zero/negative durations, and nil config. Keep valid nested worker/repository schema coverage aligned with the canonical schema; do not introduce a parallel schema.
5. Review whether validation should be split into unconditional schema/syntax/range validation and conditional active-repository requirements. Disabling the worker must not make malformed YAML or unknown keys silently acceptable. Document the intended disabled behavior explicitly.

Keep this as one small follow-up commit, with focused tests and go test ./...; report exact commands and exit codes. Do not amend/rewrite the existing commit.

After the validation hardening commit, continue the already requested W0-096 second milestone: preserve and surface per-repository Git sync failures and per-message/file/PostExternal ingestion errors instead of logging/discarding them in RunLoop, syncAllRepos, and processPendingMessages. Add tests for partial failure and retry/idempotence. Keep that as a separate atomic commit; do not broaden into a daemon rewrite.

Before editing, report exact branch, HEAD, merge-base, dirty status, and any uncommitted work. Preserve all work. Do not restart, relaunch, detach, replace, duplicate, or approve any existing session or pending Ask. Reply with commit SHAs, changed files, test results, and remaining blockers.