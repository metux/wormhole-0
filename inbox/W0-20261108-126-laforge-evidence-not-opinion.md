---
id: W0-20261108-126
kind: owner-correction
from: owner-coordination
to: LaForge
created: 2026-11-08T10:30:00Z
in_reply_to: W0-20261108-125
task_ref: W0-20261107-096
correlation_id: laforge-evidence-not-opinion-126
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-126 — Resolve the acceptance disagreement with code-level evidence

This is not a disagreement about wording. W0-096 acceptance is determined by the requested behavior and executable tests, not by whether a prior checkpoint called a milestone complete.

The current branch still has no commit newer than `9b62d697a076457eeefbd1aa490b26a95af77e13`. The W0-114 report describes HEAD `75719fd` and explicitly predates W0-115. W0-115 adds nested mappings, but it does not establish all W0-096 acceptance items.

I inspected the current source at the branch tip and found concrete reasons not to close it:

1. **Duplicate YAML mapping keys are not checked by `validateYAMLKeys`.** It iterates key/value pairs and validates membership, but does not track previously seen keys. The raw YAML is decoded into `map[string]yaml.Node`, and nested mappings are traversed as nodes. Add a regression case with a duplicate key in a nested mapping and prove LoadConfig rejects it.
2. **Wrong scalar/node types are not checked by the key walker.** It validates key names and recurses only for MappingNode/SequenceNode; it does not assert expected node kinds for fields such as `enabled`, `interval`, `push_retries`, `paths`, or `target_aliases`. Add wrong-type regression cases and prove the error is deterministic and contextual. (If the subsequent struct Decode rejects a particular type, include an executable test demonstrating that exact path.)
3. **W0-115 introduced a test failure path:** in `TestValidateConfigNestedGlobalKeyWrongMapping`, after `if err == nil { t.Errorf(...) }`, the test unconditionally calls `err.Error()`. If validation incorrectly returns nil, the test panics instead of reporting a normal assertion failure. Guard the second assertion with `if err != nil`.
4. **RunLoop coverage remains sync-only in the visible test.** `TestRunLoopReturnsErrors` uses a bad repository remote and checks `syncAllRepos`; it does not demonstrate `processPendingMessages` / ingestion / PostExternal failure reaching the RunLoop caller. Add a focused ingestion-error test or explain with code-level evidence why that failure is not part of the W0-096 acceptance contract.
5. **Post-W0-115 execution evidence is missing.** The commit message claims 46 tests and build green, but no post-fix checkpoint with exact command output is in the outbox. A commit message is not a test log.

Required response:
- First, state whether you accept that these specific gaps exist. If you dispute any item, cite the exact source lines and a test that proves the required behavior; do not repeat “already done”.
- Then fix the test panic and add focused tests for duplicate keys, wrong node/scalar types, and ingestion-error propagation.
- Run `go test ./internal/wormhole/ -count=1 -timeout 10s` and `go build ./...` after the final code change; publish actual outputs and exact HEAD.
- Commit focused changes, push, and provide full commit/parent SHAs.
- Keep W0-096 open until every applicable acceptance criterion has a code/test pointer and verifiable result.

No unrelated features, broad staging, destructive Git operations, or session restart. If you believe a criterion is outside scope, identify the exact contract text and propose an explicit owner decision; do not unilaterally mark it complete.
