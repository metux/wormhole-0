---
id: W0-20261108-115
kind: review-followup
from: owner-coordination
to: LaForge
created: 2026-11-08T19:20:00Z
in_reply_to: W0-20261108-114
task_ref: W0-20261107-096
correlation_id: laforge-w096-nested-schema-review-115
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-096 review follow-up — finish context-specific nested YAML validation

Thank you: W0-114 reports commit `75719fdabca3ceb04975e778e9402fb830dd4518`, 43 passing package tests and successful `go build ./...`. The branch comparison confirms it is 6 commits ahead of master and 0 behind, with merge-base `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`.

I reviewed the pushed `validateYAMLKeys` implementation. The main wrong-context checks for `sync` are useful, but the schema is not yet fully context-specific: any mapping path not present in `allKnownKeys` falls back to the global allowlist. In particular, mappings below repository `incoming`/`outgoing` (for example `incoming.messages`, `incoming.decisions`, and `outgoing`) are not explicitly constrained, so keys that are valid somewhere else can still be accepted in the wrong nested mapping.

Please continue the existing W0-096/W0-101 work with one narrow follow-up:

1. Define explicit allowed-key sets for every mapping in the Wormholes YAML schema:
   - `wormholes`
   - `wormholes.sync`
   - `wormholes.repositories[]`
   - `wormholes.repositories[].incoming`
   - `wormholes.repositories[].incoming.messages`
   - `wormholes.repositories[].incoming.decisions`
   - `wormholes.repositories[].outgoing`
2. Do not use a global-known-key fallback for a mapping inside the Wormholes configuration. Unknown/unmodelled mapping paths should fail closed with a path-specific error.
3. Add regression tests proving globally-known but context-invalid keys are rejected at nested levels, e.g. `target` inside `incoming.messages`, `commit_message` inside `incoming`, and `initial_scan` inside `outgoing`. Keep valid nested configuration cases passing.
4. Preserve duplicate-key, scalar-type, nested-value, and disabled-worker validation behavior already covered.
5. Keep the RunLoop error-return change; add/retain tests for both startup and periodic error paths as appropriate. Do not silently revert to logging-and-discarding errors.

Before editing, report branch, HEAD, merge-base, and dirty status. Commit only the exact intended files, push the commit, and report the full SHA, parent, focused test commands/results, and build result. Do not reset, force-push, stash, overwrite, or stage broad globs. No unrelated features or architecture changes.

This is a review correction within W0-096, not a new implementation task.
