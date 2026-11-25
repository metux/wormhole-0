# ChatGPT Project Instructions

These instructions are intentionally limited to project-independent guidance for locating and using the Wormhole coordination channel. Project-specific implementation knowledge, fleet assignments, and technical constraints belong in the Wormhole repository and the relevant skills.

## Session entry point

1. Start with the current `metux/wormhole-0:master` repository.
2. Read `COLLABORATION.md` for the active message format, mailbox paths, lifecycle ownership, and publication rules.
3. Read `SESSION_BOOTSTRAP.md` for the current project handoff and repository index. Refresh live state; historical notes and SHAs are not automatically current.
4. Read `CHATGPT_PROJECT_INSTRUCTIONS.md` in the Wormhole repository only as a short pointer/index; this ChatGPT Project file is not the place for project-specific technical policy.
5. Locate and read the applicable skills in the workspace repository before acting:
   - Cross-agent communication: `X11Libre/mpbt-workspace:mtx/agent-config/.claude/skills/wormhole-communication/SKILL.md`
   - XAverius implementation, when relevant: `X11Libre/mpbt-workspace:mtx/agent-config/.claude/skills/xaverius-implementation/SKILL.md`
   - For other work, locate the corresponding project skill rather than copying its contents here.

## Wormhole communication

- Use `metux/wormhole-0:master` as the transport spool unless the owner explicitly selects another branch.
- Before publishing a message, follow the current `COLLABORATION.md` exactly. Publish a new immutable message only under `mail/<recipient-ship>/new/<message-id>.md`.
- Never write to manager-owned `cur/`, `processed/`, or `failed/` paths, and never replay the frozen legacy `inbox/` or `outbox/` archives.
- Distinguish publication, claim/import, active-session injection, explicit acknowledgement, work started, and completion. A Git commit or `processed/` entry alone does not prove that an agent read, acknowledged, or acted on a message.
- Use the established skills and current repository documentation; do not invent paths, commands, identities, or message lifecycle transitions.

## General working discipline

- Refresh current task ownership, repository branches, HEADs, and relevant specifications before acting. Treat chat history and historical handoff notes as pointers, not live state.
- Do not claim access to local terminals, filesystems, processes, or Comms sessions unless an available tool actually provides it.
- Preserve existing ownership and avoid duplicate tasks or concurrent writes in a shared worktree.
- Make scoped, reviewable changes and report exactly what was inspected, changed, published, and verified. Do not claim tests or agent activity that were not observed.

## Project-specific context

Keep project-specific instructions out of this file:
- Fleet/task handoff and historical implementation context: `metux/wormhole-0:master/SESSION_BOOTSTRAP.md`.
- XAverius-specific ownership, repository/spec authority, and implementation boundaries: `metux/wormhole-0:master/XAVERius_PROJECT_CONTEXT.md`.
- Detailed communication and implementation procedures: the skills listed above.
