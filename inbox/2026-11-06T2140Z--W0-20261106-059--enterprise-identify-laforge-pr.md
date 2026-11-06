# W0-059 — Enterprise: identify Laforge's claimed pull request

**Owner:** Enterprise  
**Role:** local coordination and Comms routing only  
**Priority:** immediate

## Trigger

The owner reports that LaForge has already created a pull request for the Wormhole manager work. Current public GitHub checks do not show a corresponding PR in `mpbt-hq/starfleetctl`, and the current branch remains `work/wormhole-manager-w0-017` at `94af3ab07b5188e6a994dd98c40a93c2365355da`.

## Required actions

1. Contact LaForge through the established local Comms route and ask for the exact GitHub PR URL, repository, PR number, head repository/branch/SHA, and base repository/branch.
2. Verify locally that the reported PR URL opens and corresponds to the intended Wormhole manager implementation. Report the actual URL and metadata into the Wormhole outbox.
3. If LaForge says the PR is in `mpbt-hq/starfleetctl`, explain that the current GitHub PR listing for that repository shows only PRs #1–#6 (all old/closed), and ask whether the PR was opened against a different repository or fork.
4. Do not create, edit, close, or merge a PR. Do not perform a technical audit. Owner/ChatGPT will inspect the complete diff, commits, checks, and review state after receiving the exact URL.
5. Distinguish clearly between a local branch, a pushed branch, a draft PR, and an actually opened GitHub PR. Do not report a PR as created without its URL and GitHub metadata.

## Boundaries

Enterprise remains coordination-only. Do not guess the URL, repository, PR number, or destination.
