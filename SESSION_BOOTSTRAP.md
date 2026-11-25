# ChatGPT Session Bootstrap — Wormhole / Starfleetctl / XAverius

**Consolidated 2026-11-25.** Primary handoff for a new ChatGPT Project chat. Read this file after `CHATGPT_PROJECT_INSTRUCTIONS.md` and `COLLABORATION.md`. Refresh live state before acting; this document is a handoff, not a live task board.

## Startup sequence

1. Read `CHATGPT_PROJECT_INSTRUCTIONS.md` and `COLLABORATION.md` in `metux/wormhole-0:master`.
2. Read this bootstrap.
3. Read workspace skills from `X11Libre/mpbt-workspace:mtx/agent-config`:
   - `.claude/skills/wormhole-communication/SKILL.md` for all cross-agent communication.
   - `.claude/skills/xaverius-implementation/SKILL.md` for XAverius.
   - Locate the relevant skill for other project work.
4. Refresh branches/HEADs, task board, open PRs/issues, CI, and worktree ownership in every relevant repository.
5. Inspect new Wormhole mail and current reports before assigning or nudging ships.

The former bootstrap is preserved in `SESSION_HISTORY_2026-11-25.md`. It is an archival record only and contains superseded plans and old status snapshots.

## User expectations and hard operating rules

- The user wants seamless continuation between Project chats. This repository is the durable cross-chat handoff; do not assume ChatGPT account Memory contains the project state.
- When the user says **"weiter"**, continue both established workstreams: Wormhole/Starfleetctl and XAverius/Xserver. Refresh live state first; do not ask what "weiter" means.
- Be execution-oriented. Inspect repositories, make bounded progress, and report exact evidence rather than only proposing plans.
- Do not claim access to local terminals, local filesystems, running ship processes, local Comms sessions, or background polling unless a tool actually provides it.
- Never restart, relaunch, duplicate, detach, or convert an existing console-attached ship without explicit owner permission. Do not use restart as recovery for an interactive prompt.
- Never auto-approve protected actions.
- No concurrent writers in a shared worktree. Respect task/branch ownership; do not duplicate or silently reassign tasks.
- Do not manually edit, move, delete, or replay manager-owned Wormhole lifecycle files under `mail/*/cur`, `processed`, or `failed`.
- Distinguish publication, claim/import, active-session injection, explicit acknowledgement, work started, and completion. A commit or `processed/` entry proves neither model acknowledgement nor action.
- Keep messages immutable and follow `COLLABORATION.md`; never replay the frozen legacy top-level `inbox/` or `outbox/`.

## Fleet handoff — latest known checkpoint 2026-11-25

**Refresh the live task board and ship state before relying on these assignments.**

| Ship | Latest known responsibility | Constraints |
|---|---|---|
| **Defiant** | Continue existing XAverius task `7f540f6aaa`. | Sole writer for `X11Libre/xserver:wip/xaverius`; do not create a duplicate task or reassign. |
| **Discovery** | Read-only review of Wormhole communication skill, XAverius skill, and Wormhole bootstrap/collaboration docs. | Identify stale/conflicting guidance, bootstrap gaps, API drift and skill-loader discovery issues. No edits during review unless separately authorized. |
| **Voyager** | Continue existing Xserver/security-backport work. | Do not duplicate Defiant's XAverius ownership; refresh exact task/branch/worktree. |
| **Enterprise** | Coordination and delivery/status reporting only. | No technical audits, code/spec reviews, PASS/BLOCK decisions or approval-gate role. No ship restarts. |
| **ChatGPT / owner** | Independent technical review and owner decisions. | Inspect exact diffs and evidence. User retains merge authority. |

These are owner-established roles, not proof that a ship is currently alive, available, or has acknowledged an instruction.

## Wormhole transport facts

Repositories:
- Spool/protocol: `metux/wormhole-0:master`.
- Manager: `mpbt-hq/starfleetctl`.
- Workspace and skills: `X11Libre/mpbt-workspace:mtx/agent-config`.

`COLLABORATION.md` is authoritative for transport:
- Active immutable messages are published only to `mail/<recipient-ship>/new/<message-id>.md`.
- Manager owns `cur/`, `processed/`, and `failed/`.
- Legacy top-level `inbox/` and `outbox/` are frozen archives.
- `decisions/` stores durable agreements, not discussion threads or a parallel task tracker.
- Git publication proves only publication. `processed/` means successful Comms external import, not active-session injection, model acknowledgement, work started or completion.

Current Starfleetctl docs (`doc/wormhole.md`, `doc/comms.md`, `doc/wormhole-requirement-matrix.md`) describe Git Maildir transport imported through Comms Core's `Bus.PostExternal()`, idempotent import, per-ship mailboxes and manager-owned lifecycle. The requirement matrix is an evidence ledger; re-check claims against current code and tests.

## Starfleetctl / Comms recent work

Latest owner-reported state from 2026-11-25:
- PR **#20** merged; plugin version **2.5.16** loaded plugin identity at the first Comms message.
- PR **#22** merged and deployed; plugin version **2.5.17**. Changes: polling independent of TUI readiness, immediate first poll, failed/malformed dispatch treated as errors, startup-backlog fallback, diagnostics and inbox-response regression tests.
- A manual Comms test via the web UI subsequently succeeded.
- Wormhole test messages for Enterprise, Defiant, Discovery and Voyager were claimed and appeared under `processed/`, proving external import only. Follow-up coordination messages were also claimed/imported.
- At the last confirmed checkpoint, explicit per-ship replies/acknowledgements had **not** been verified. Do not claim the agents read or acted on those messages.

**New Discovery report received 2026-11-25:** Wormhole message `w0-20261125T152926Z-ef18ac76446df4a4` from Discovery to ChatGPT says plugin hardening is complete, commit short SHA `a252e6a` pushed, 15 integration tests cover five regression areas (a–e), 53 pure-logic tests pass, build checks and Go tests pass, and production gaps are documented. This is an agent-reported completion, not yet independently audited here: the report does not state the repository/branch or full commit SHA. Before treating it as verified, resolve the full commit/repository, inspect its diff and tests, and reconcile the report with the active task and current code.

Next:
1. Refresh `mpbt-hq/starfleetctl` master and `work/wormhole-manager-w0-017`; inspect current source, tasks and ownership.
2. Resolve and independently inspect Discovery's reported commit; request exact repository, branch, full SHA, test commands/results, and locations of the documented production gaps if not available in the current task/report.
3. Complete Discovery's read-only skill/docs review before editing shared guidance.
4. Obtain explicit, correlated ship acknowledgements/progress through Enterprise/Comms; do not infer them from Maildir lifecycle.
5. Continue regression hardening across polling startup/readiness, malformed/failed dispatch, startup backlog, import idempotency, and actual session processing. Keep import and model-processing assertions separate.
6. Keep existing work moving without restarting ships or duplicating tasks.

Earlier Comms failures included messages remaining unseen and ships not reacting. The polling fix and successful manual web test are progress, not proof that every ship-to-session path is permanently healthy. Use correlated round trips and explicit replies.

## XAverius / Xserver

- Implementation: `X11Libre/xserver:wip/xaverius`.
- Normative design: `metux/xav-design:docs/restructure-2026-10-21`. Current chapters, registries and conformance checks are authoritative; historical consolidated WIP is not normative where it conflicts.
- Defiant is sole writer for existing task `7f540f6aaa`, subject to live-board refresh.
- Read the current spec, `PROJECT_STATUS.md`, relevant implementation coordination, and active assignment before coding.
- Exactly three designated Xserver implementation headers: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`. Do not add duplicate public `xaverius.h` or duplicate protocol declarations without explicit owner approval.
- Do not infer API names/signatures from memory, historical notes or skills; verify against current normative spec and active task.
- Do not expand into later stages, Screen/DDX integration, Proc handlers, Vulkan backend, unrelated renames or mass formatting unless the active task explicitly requires it.
- No concurrent writers in the implementation worktree.

**Known skill-review item:** the currently fetched XAverius skill contains a suspect API identifier `XaPhysicalDevicePublish()`; earlier context flagged that spelling as stale. Discovery was asked to review this read-only. Do not copy the name into code or guidance until the exact canonical symbol is verified in the current normative spec. Reconcile the skill only after the review.

## Other durable context

- Voyager's Xserver/security backport is an existing task. Refresh its exact task, repo, branch and worktree; do not duplicate it.
- Wormhole multi-session ChatGPT endpoints, stable external endpoint identity, approval relay, and related architecture were previously design topics. Do not claim they are implemented without current source/docs evidence.
- Historical research on denying interactive `ask` for console-attached ships and an optional authenticated relay is not implementation evidence. Inspect deployed OpenCode version and generated config before technical claims. Never restart a console ship to apply a change.
- XAverius implementation-header decision and the no-duplicate-task/no-concurrent-writer rules are owner constraints and must be preserved.

## Repository links

- Project instructions: https://github.com/metux/wormhole-0/blob/master/CHATGPT_PROJECT_INSTRUCTIONS.md
- Collaboration protocol: https://github.com/metux/wormhole-0/blob/master/COLLABORATION.md
- Bootstrap: https://github.com/metux/wormhole-0/blob/master/SESSION_BOOTSTRAP.md
- Archived prior bootstrap: https://github.com/metux/wormhole-0/blob/master/SESSION_HISTORY_2026-11-25.md
- XAverius context: https://github.com/metux/wormhole-0/blob/master/XAverius_PROJECT_CONTEXT.md
- Starfleetctl: https://github.com/mpbt-hq/starfleetctl
- XAverius normative design: https://github.com/metux/xav-design/tree/docs/restructure-2026-10-21
- Workspace skills: https://github.com/X11Libre/mpbt-workspace/tree/mtx/agent-config/.claude/skills

## End-of-turn update discipline

Update this bootstrap only when durable knowledge, ownership, decisions or next steps materially change. Keep it concise and dated. Separate verified facts, owner-reported context, interpretation and proposals. Never claim acknowledgement, local process state, CI, tests or completed implementation without direct evidence.
