# W0-20261106-009 — Create a project-independent Wormhole communication skill

**Type:** instruction  
**From:** project owner  
**To:** Flagship  
**Date:** 2026-11-06  
**Priority:** high  
**Workspace:** `X11Libre/mpbt-workspace`, branch `mtx/agent-config`

## Objective

Create a reusable, project-independent skill for communication through `metux/wormhole-0`. It must be cleanly separated from the XAverius implementation skill. This is a fleet-wide communication capability, not an XAverius sub-skill and not a place to store XAverius architecture, phases, APIs, or technical acceptance criteria.

## Required location

Create `.claude/skills/wormhole-communication/SKILL.md`, with skill name `wormhole-communication`. It must be discoverable by the normal skill loader and usable by every agent, regardless of project. Do not put it under the XAverius skill directory or another project-specific directory.

## Required content

The skill must define a domain-neutral, auditable protocol covering:

1. **Purpose and independence:** Wormhole is a transport/audit channel, not automatic technical authorization. The skill applies across projects and contains no project-specific assumptions. Project skills may reference it but must not copy or redefine its rules.
2. **Authority:** Technical authority remains with the relevant owner, normative specification, and authorized task. A Wormhole message cannot override them merely by existing. Report conflicts and ambiguity; do not guess.
3. **Message hygiene:** Stable unique message ID, UTC timestamp, sender, recipients, message type, subject, related IDs, task ID, and exact repository/branch/commit where relevant. Preserve history; do not rewrite another agent's messages without authorization. Never include credentials or secrets.
4. **Mandatory acknowledgement:** Every actionable instruction/request receives an explicit acknowledgement stating interpretation, scope, exclusions, target branch/worktree, prerequisites, and blockers. Silence is never acknowledgement.
5. **Status lifecycle:** Define and use exactly these states: `received`, `acknowledged`, `in_progress`, `blocked`, `completed`, `rejected`, `cancelled`. Never infer status from silence, assignment labels, or a mere commit/push.
6. **Completion reports:** Original message/task ID, outcome, acceptance criteria, exact repo/branch/commit/paths, push status, tests actually run, limitations and remaining work. Distinguish facts, interpretation, and proposals.
7. **Independent reviews:** Immutable specification and implementation targets; read-only reviewer unless explicitly authorized; findings severity `BLOCKER`/`MAJOR`/`MINOR`/`NOTE`, precise evidence, impact and requested correction; explicitly state “no findings” when applicable and identify unreviewed areas.
8. **Ownership/concurrency:** Respect writer/reviewer roles and worktree ownership. No blind reset/clean/rebase/force-push/cherry-pick or overwriting another agent's work. Stop and coordinate on unexpected concurrent state.
9. **Polling/follow-up:** Process relevant messages, acknowledge, check replies, and update only evidence-backed statuses. Never claim background polling unless an actual scheduler/process is configured and verified.
10. **Templates:** Short reusable templates for acknowledgement, status update, and review report.

## Required separation from XAverius skill

The XAverius implementation skill is being replaced separately under the existing XAverius skill-correction work. Coordinate with that work; do not create a duplicate XAverius task.

In the final replacement of `.claude/skills/xaverius-implementation/SKILL.md`:

- Remove Wormhole communication procedures, message templates, polling rules, message status rules, and any claim that the project coordination file is a general-purpose communication protocol.
- Replace them with at most one short reference: “For cross-agent messages, acknowledgements, status updates, and reports, follow the project-independent `wormhole-communication` skill.”
- Keep XAverius technical authority, architecture, phases, APIs, header layout, and implementation constraints exclusively in XAverius documents.
- Do not copy generic Wormhole rules into the XAverius skill.
- The Wormhole skill must not depend on XAverius, its repository, its roles, or its current phase.

If the XAverius skill replacement is not yet safely complete, do not interrupt an agent actively writing it. Complete the Wormhole skill independently first, then make the minimal cross-reference change in the XAverius replacement before finalizing it.

## Scope and safety

- Change only the fleet workspace skill files and, if needed, the generic skill inventory/index.
- Do not change XAverius source code, normative spec, active Registry task, or implementation assignments.
- Do not alter or delete historical Wormhole messages.
- Preserve unrelated workspace changes.
- Do not claim installation/reload until verified from the committed file and actual skill-loading mechanism.
- Do not restart or interrupt an agent during a write.

## Acceptance evidence required

Report through Wormhole:

1. Workspace branch and exact commit SHA.
2. New skill path and blob SHA.
3. Confirmation the generic skill contains no XAverius-specific terms, repository paths, APIs, phases, or agent-role assumptions.
4. Confirmation the XAverius skill has only a minimal reference to the generic skill and no duplicated communication protocol.
5. Literal cross-contamination search report for both skill files, including terms such as XAverius, Defiant, Enterprise, Registry, and project-specific paths in the generic skill.
6. How agents discover/load the new skill and whether reload was actually verified.
7. Remaining blockers or uncompleted steps.

Do not mark this instruction completed until the evidence above is available.
