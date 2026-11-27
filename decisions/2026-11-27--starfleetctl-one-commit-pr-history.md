# Starfleetctl pull-request history: one coherent commit by default

Date: 2026-11-27

## Decision

Starfleetctl pull requests should normally land as exactly one self-contained commit on `master`. Before merge, consolidate the PR's implementation and follow-up commits into one coherent commit with a clean Conventional Commit subject, a useful body describing the motivation and important behavior, test evidence, and the required `Signed-off-by` trailer.

Use GitHub's native squash-merge method when the repository allows it. If native squash merge is disabled but rebase merge is allowed, rewrite the owner-controlled PR branch to one commit based on the current target branch, verify the rewritten branch contains exactly one commit and the same reviewed tree, wait for CI on the new exact head, then use rebase merge. Do not silently fall back to a merge commit that creates noisy history.

## Exceptions

Retain multiple commits only when there is a concrete benefit, such as independently useful/revertible changes, a deliberately staged migration, or a meaningful dependency/order boundary. Explain the reason in the PR description or review notes. Do not split commits merely to expose implementation micro-steps.

## Safety and verification

- Never rewrite another agent's branch or a shared/protected branch without its owner's explicit authorization.
- For an owner-authorized rewrite, use force-with-lease against the exact observed old head and verify the resulting ref.
- Re-run and inspect CI on the final rewritten head; green checks from the pre-squash SHA are not a substitute.
- Preserve the PR and its discussion; do not close/recreate it just to clean history.
- Follow the repository's `AGENTS.md` requirements, including signed-off commits and documentation consistency.

## Consequences

PR review and merge history become easier to scan and revert. Fine-grained development history remains available in the PR discussion and archived branch refs when preserved. A repository that disables native squash merge can still maintain one-commit-per-PR history by consolidating the branch and using rebase merge.
