# Collaboration protocol

This repository is the shared, Git-based communication channel between the project owner, ChatGPT, and the Starfleetctl agent fleet.

## Purpose

- Exchange durable instructions, questions, decisions, status reports, and hand-off notes.
- Keep coordination independent of any particular chat session or local agent runtime.
- Prefer small, reviewable Markdown changes over large rewrites.

## Working rules

1. **Repository state is authoritative.** Read the current files and Git history before acting; do not rely on stale chat context when repository state is available.
2. **Keep changes scoped.** Do not rename, reformat, or reorganize unrelated files. Preserve established wording and formatting unless a change is needed.
3. **Use commits as messages.** Make each logical update a separate commit with a descriptive subject. Include enough context in the commit body or document for another agent to understand the intent.
4. **Do not overwrite concurrent work.** Fetch/read the latest branch before editing. If the target changed meanwhile, reconcile rather than replacing it blindly.
5. **Separate instructions from reports.** Instructions and requests should be clearly marked as such; reports should state what was done, what was not done, evidence, blockers, and next steps.
6. **No silent completion claims.** A task is complete only when its result is committed and the commit is visible on the shared branch.
7. **Keep this repository lightweight.** Link to large external workspaces or artifacts instead of copying them here. Do not store credentials, tokens, private keys, or other secrets.

## Suggested layout

- `README.md` — entry point and current pointers.
- `COLLABORATION.md` — this protocol.
- `inbox/` — requests and instructions awaiting attention.
- `outbox/` — replies, findings, and hand-off reports.
- `decisions/` — durable decisions that should not be lost in message traffic.
- `status/` — concise current status and blockers.

Create directories only when there is actual content to put in them. Avoid placeholder files and unnecessary scaffolding.

## Message conventions

Use filenames beginning with an ISO date and a short topic, for example:

- `inbox/2026-11-05-enable-repository-channel.md`
- `outbox/2026-11-05-channel-access-check.md`

Every actionable request should identify:

- owner or intended recipient;
- requested outcome;
- relevant repository/branch/path;
- constraints and explicit non-goals;
- acceptance criteria or evidence expected;
- whether a reply or follow-up is required.

Every report should identify:

- request being answered;
- changes made (paths and commit);
- checks/evidence;
- unresolved issues and blockers;
- recommended next action.

## Branch

Use the repository's default branch (`master`) for this shared channel unless the owner explicitly designates another branch. Do not force-push.
