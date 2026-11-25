# Isolated ChatGPT branches and fleet writer claims

**Date:** 2026-11-25  
**Status:** Owner-confirmed working rule

## Decision

When ChatGPT performs work through repository tools on a dedicated Git branch, separate from the fleet's shared/local worktrees, it does not need to acquire the fleet's shared-worktree writer claim merely to make changes on that isolated branch.

This does not remove the need to refresh task ownership and branch/HEAD state before work. Before editing, ChatGPT must still check whether a fleet task owns the same files or logical change, and avoid silently duplicating or conflicting with that work. If the change touches an actively owned area, coordinate before integration; do not overwrite, rebase destructively, or force-push another agent's work.

A claim remains required when ChatGPT edits a shared fleet worktree or otherwise becomes a concurrent writer in the same checkout. The no-concurrent-writers rule applies to shared worktrees, not to independent branch creation as such.

## Rationale

The owner clarified that ChatGPT's dedicated branch is separate from the fleet worktree. A shared-worktree claim would therefore not protect a shared checkout and would add unnecessary coordination overhead. Branch isolation provides the workspace boundary; ownership and overlap checks still protect integration.

## Consequences

- Prefer a dedicated branch for ChatGPT's direct implementation work.
- Do not request a fleet code claim solely for edits isolated to that branch.
- Continue checking live assignments, target branches, changed paths, and overlapping work.
- Use Wormhole/Comms for fleet requests that require local runtime access or other fleet-only evidence.
- Distinguish message publication, import, acknowledgement, and actual response; never infer that a ship has acted from a Git commit alone.

## Related

- `decisions/2026-11-25--starfleet-work-principles.md`
- `decisions/2026-11-08--wormhole-is-comms-transport.md`
