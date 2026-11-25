# Decision: “weiter” means independent progress audit

**Date:** 2026-11-25  
**Status:** Active

## Decision

When the owner says **“weiter”** in a project workstream, ChatGPT must continue execution by independently checking what has actually changed and what remains to be done. It must not treat ship status messages, acknowledgements, or claimed completion as reliable evidence.

## Required procedure

1. Refresh relevant task ownership, repositories, branches, HEADs, PRs, diffs, and CI.
2. Inspect new Wormhole messages and lifecycle state. A message in `processed/` proves import only, not session injection, model acknowledgement, work, or completion.
3. Verify implementation directly against the active task, normative specification, and acceptance criteria.
4. Check tests and CI against the exact relevant commit; do not repeat unverified agent claims as facts.
5. Identify the next concrete bounded action. If a ship needs direction, send an actionable instruction with scope, exclusions, target branch/worktree, prerequisites, and evidence-based deliverables.
6. Verify publication and transport: exact committed message content, then manager-owned lifecycle transition to `processed/` or inspect `.reason` under `failed/`. Never edit manager-owned lifecycle paths.
7. Report verified facts separately from agent-reported claims and unresolved uncertainty.

## Rationale

The owner cannot rely on consistently reliable ship status reports or session processing. Progress must therefore be assessed from repository and transport evidence, with new instructions issued as needed rather than waiting passively for reports.

## Scope

Applies to all project workstreams and ships whenever the owner says “weiter”. It does not authorize restarting ships, changing task ownership, concurrent writes, or bypassing existing review/merge authority.
