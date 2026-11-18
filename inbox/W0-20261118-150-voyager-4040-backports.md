# W0-20261118-150 — Voyager: prepare PR #4040 release backports

The user reports Voyager is idle and explicitly wants backports of https://github.com/X11Libre/xserver/pull/4040.

Scope: release/25.0, release/25.1, release/25.2 only. Do not change master.

1. Inspect exact current HEAD and fb line-drawing implementation in each release branch. Determine independently whether the undefined signed-left-shift expression fixed by #4040 is still present. Do not infer from master or branch ancestry.
2. If a branch already contains an equivalent fix, do not create a duplicate PR; document the exact source/commit evidence.
3. For each affected branch, create a separate minimal backport branch and PR. Preserve the original narrow scope; no unrelated refactoring or formatting.
4. Add or adapt a focused regression test if the existing test infrastructure can exercise the case without broad changes. Explain if a test cannot reasonably be added.
5. Run focused tests and the relevant CI for each backport. Record exact branch HEAD, patch commit, PR URL, test commands and actual results.
6. Publish an outbox report with a per-branch table: vulnerable expression present? backport commit/PR? test/CI status? remaining blocker?
7. Do not merge the PRs. Wait for user review.

Keep the separate future repository-wide undefined-behavior scan on the TODO list; this task is only the #4040 backports. Do not claim Git inbox publication proves Comms import or task acceptance.
