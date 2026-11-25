# ChatGPT Session Bootstrap — Wormhole / Starfleetctl / XAverius

**Consolidated 2026-11-25.** This is the primary handoff for a new ChatGPT Project chat. Read it after `CHATGPT_PROJECT_INSTRUCTIONS.md` and `COLLABORATION.md`; refresh all live state before acting. It is not a live task board.

## Startup

1. Read `metux/wormhole-0:master/CHATGPT_PROJECT_INSTRUCTIONS.md` and `COLLABORATION.md`.
2. Read this bootstrap.
3. Read workspace skills from `X11Libre/mpbt-workspace:mtx/agent-config`:
   - `.claude/skills/wormhole-communication/SKILL.md` for all cross-agent communication.
   - `.claude/skills/xaverius-implementation/SKILL.md` for XAverius.
   - Locate the applicable skill for other work.
4. Refresh branches/HEADs, task board, open PRs/issues, CI, and worktree ownership in all relevant repositories.
5. Inspect new Wormhole mail and coordination reports before assigning or nudging ships.

The superseded bootstrap is preserved via `SESSION_HISTORY_2026-11-25.md`. That archive is historical only.

## User expectations and hard rules

- The goal is seamless continuation between Project chats. This repository is the durable handoff; do not assume account Memory contains the project state.
- When the user says **"weiter"**, continue both established strands: Wormhole/Starfleetctl and XAverius/Xserver. Refresh live state first; do not ask what "weiter" means.
- Be execution-oriented: inspect, make bounded progress, report evidence.
- Never claim local terminal/filesystem/process/Comms access or background polling unless a tool actually provides it.
- Never restart, relaunch, duplicate, detach, or convert an existing console-attached ship without explicit owner permission. Never use restart to recover an interactive prompt.
- Never auto-approve protected actions.
- No concurrent writers in a shared worktree. Respect ownership; do not duplicate or silently reassign tasks.
- Never manually edit/move/delete manager-owned Wormhole lifecycle files under `mail/*/cur`, `processed`, or `failed`. Never replay frozen legacy top-level `inbox/` or `outbox/`.
- Distinguish publication, claim/import, active-session injection, explicit acknowledgement, work started, and completion. A Git commit or `processed/` entry proves neither model acknowledgement nor action.
- Follow `COLLABORATION.md` exactly; messages are immutable.

## Fleet handoff — latest known 2026-11-25; refresh live board

| Ship | Latest known responsibility | Constraints / evidence |
|---|---|---|
| **Defiant** | Existing XAverius task `7f540f6aaa`. | Sole writer for `X11Libre/xserver:wip/xaverius`; do not duplicate/reassign. Enterprise's latest report lists Defiant as a terminal but gives no task detail. |
| **Discovery** | Read-only review of Wormhole communication skill, XAverius skill, bootstrap/collaboration docs. | Enterprise reported task at 100%. Discovery also reported plugin-hardening completion; distinguish these tasks on live board. |
| **Voyager** | `task-xserver-pr-4208-and-4170-rebase-squash-sign-off-force-with-lease-ci-verification`. | Enterprise reported 75%. Same report says no suitable ship for #4199/#4170 and those were queued; reconcile ambiguity from live board before action. |
| **Enterprise** | Coordination and delivery/status reporting only. | No technical audits, code/spec reviews, PASS/BLOCK decisions or approval gate. No ship restarts. |
| **McKinley** | `starfleet/opencode-meta-model-limit-compaction`. | Enterprise reported no percentage; refresh board. |
| **ChatGPT / owner** | Independent technical review and owner decisions. | User retains merge authority. |

Enterprise report: Wormhole message `w0-20261125T154525Z-338aadcc02204a66` (15:45:25Z), prefix `25526c`. It listed terminals Defiant/Discovery/Enterprise/McKinley/Voyager, the assignments above, and prior receipts `ea4793d9dfd7e8d0`, `ae5a949fc1dccde4`, `1990d7df4621c4ad`. This is a published report, not independent proof of local process liveness or model activity.

## Wormhole transport facts

Repositories: spool/protocol `metux/wormhole-0:master`; manager `mpbt-hq/starfleetctl`; workspace/skills `X11Libre/mpbt-workspace:mtx/agent-config`.

- Active immutable messages only at `mail/<recipient-ship>/new/<message-id>.md`.
- Manager owns `cur/`, `processed/`, `failed/`.
- Legacy top-level `inbox/` and `outbox/` are frozen historical archives.
- `decisions/` stores durable agreements, not discussion threads or a parallel task tracker.
- `processed/` proves Comms external import only, not active-session injection, model acknowledgement, action, or completion.
- Starfleetctl docs `doc/wormhole.md`, `doc/comms.md`, `doc/wormhole-requirement-matrix.md` describe import through Comms Core's `Bus.PostExternal()`, idempotency, per-ship mailboxes and lifecycle ownership. Re-check matrix claims against current source/tests.

## Starfleetctl / Comms work

Latest owner-reported state:
- PR **#20** merged; plugin **2.5.16** loaded identity at first Comms message.
- PR **#22** merged/deployed; plugin **2.5.17**. Polling independent of TUI readiness; immediate first poll; failed/malformed dispatch errors; startup-backlog fallback; diagnostics and inbox-response tests.
- Manual Comms test through web UI succeeded.
- Wormhole test messages to Enterprise, Defiant, Discovery and Voyager were claimed and appeared in `processed/`; this proves import only. Follow-up coordination messages were also imported.
- Explicit per-ship acknowledgement was not verified at that checkpoint.

**Discovery report, 2026-11-25:** message `w0-20261125T152926Z-ef18ac76446df4a4` says plugin hardening complete, short SHA `a252e6a` pushed, 15 integration tests across five regression areas (a–e), 53 pure-logic tests pass, build checks and Go tests pass, production gaps documented. This is agent-reported, not independently audited: repository/branch/full SHA are missing. Resolve exact target, inspect diff/tests and documented production gaps before treating as technically verified.

Next:
1. Refresh `mpbt-hq/starfleetctl` master and `work/wormhole-manager-w0-017`, task board and all relevant diffs.
2. Resolve Discovery's full commit/repo/branch and inspect it; request exact test commands/results and production-gap locations if absent.
3. Complete Discovery's read-only docs/skill review before shared edits.
4. Obtain correlated explicit ship acknowledgements/progress through Enterprise/Comms.
5. Continue regression hardening across polling/readiness, dispatch failures, backlog, import idempotency and actual session processing. Keep import and model-processing assertions separate.
6. Keep existing work moving without restarts or duplicate tasks.

Earlier failures included Comms messages remaining unseen and ships not reacting. The polling fix and successful manual web test are progress, not proof every ship-to-session path is healthy. Use correlated round trips and explicit replies.

## Starfleetctl Comms event pipeline — current implementation

**Dedicated Starfleet session owns this workstream.** Do not message Defiant, inspect its worktree, or alter its XAverius task from this session; the separate XAverius session owns Defiant and Xserver `wip/xaverius`. Synchronize across sessions through Wormhole.

- Repository: `mpbt-hq/starfleetctl`
- Base: `master` at `bf6144ebae79c41a166ce6695f9affcf67402bcd`
- Branch: `feat/comms-event-pipeline`
- Current head: `582b5e7e83c3631d80289c445b2d95726bf15afa`
- Draft PR: https://github.com/mpbt-hq/starfleetctl/pull/23
- PR is intentionally a first incremental slice, not completion of issue #23.
- Plugin version in the fragment is `2.5.18`.

**Verified implementation on this head:**
- A shared `PreparedPrompt` representation and validator accepts a complete prompt `string` or ordered `string[]`.
- Both existing V1 adapters (current-turn `experimental.chat.system.transform` and asynchronous `client.session.promptAsync`) use the same temporary legacy inbox formatter and exact prepared text.
- The Go dispatch wire protocol is not changed yet; legacy inbox formatting remains a temporary compatibility bridge.
- The helper is `prompt-preparation.mjs`, installed as a runtime module but not registered as a plugin. Its compile-time declaration is `prompt-preparation.d.mts`, not deployed.
- Bootstrap regression coverage verifies helper installation and non-registration.
- CI runs the plugin tests via `make test-plugin`.

**Runtime compatibility evidence:**
- Discovery read-only probe: OpenCode CLI `1.19.3`, resolved `@opencode-ai/plugin` SDK `1.19.3`, timestamp `2026-11-25T17:22:47Z`.
- Upstream tag `v1.19.3` confirms the existing V1 hooks and `promptAsync` API.
- `session.idle` is session-scoped and carries no Comms message/event ID; it is not sufficient proof that a particular directive completed.
- Durable decisions: `decisions/2026-11-25--opencode-1.19.3-compatibility.md` and `decisions/2026-11-25--opencode-plugin-helper-packaging.md`.

**Exact-head CI evidence (`582b5e7e83c3631d80289c445b2d95726bf15afa`):**
- All three GitHub Actions jobs passed.
- Go 1.22.x and stable: build, vet, and full test suite passed.
- Plugin bundle/typecheck passed.
- Plugin tests: 59 passed, 0 failed.
- Bootstrap regression tests passed in the Go matrix.
- No local execution is claimed.

Discovery imported a read-only follow-up request for review of PR #23 at the exact head (Wormhole message `w0-20261125T174054Z-pr23-review-followup`). Import is not acknowledgement or review completion; await a correlated report. PR remains draft. Remaining work: one central poller/event dispatcher, generic Go event polling and Go-owned prompt formatting, durable lifecycle/retry semantics, restart recovery, and correlated end-to-end turn-start/completion tests.

## XAverius / Xserver

- Implementation: `X11Libre/xserver:wip/xaverius`.
- Normative design: `metux/xav-design:docs/restructure-2026-10-21`; current chapters, registries and conformance checks are authoritative. Historical consolidated WIP is not normative where it conflicts.
- Defiant is sole writer for existing task `7f540f6aaa`, subject to live-board refresh.
- Before coding read current spec, `PROJECT_STATUS.md`, relevant implementation coordination and active assignment.
- Exactly three designated implementation headers: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`. No duplicate public `xaverius.h` or protocol definitions without owner approval.
- Never infer API names/signatures from memory, old notes or skills; verify current normative spec and active task.
- No later stages, Screen/DDX integration, Proc handlers, Vulkan backend, unrelated renames or mass formatting unless explicitly in active task. No concurrent writers.

**Skill-review item:** fetched XAverius skill contains suspect `XaPhysicalDevicePublish()`; prior context flagged it as stale. Discovery was asked to review this read-only. Do not use the identifier until the canonical symbol is verified in the current normative spec. Reconcile the skill only after review.

## Other durable context

- Voyager's Xserver/security backport is existing work; refresh exact task/repo/branch/worktree; do not duplicate.
- Wormhole multi-session GPT endpoints, stable external endpoint identity and approval relay were design topics, not confirmed implementation. Verify source/docs before claims.
- Research on denying interactive `ask` for console ships and optional authenticated relay is not implementation evidence. Inspect deployed OpenCode version and generated config. Never restart console ships to apply changes.

## Links

- Instructions: https://github.com/metux/wormhole-0/blob/master/CHATGPT_PROJECT_INSTRUCTIONS.md
- Protocol: https://github.com/metux/wormhole-0/blob/master/COLLABORATION.md
- Bootstrap: https://github.com/metux/wormhole-0/blob/master/SESSION_BOOTSTRAP.md
- History index: https://github.com/metux/wormhole-0/blob/master/SESSION_HISTORY_2026-11-25.md
- XAverius context: https://github.com/metux/wormhole-0/blob/master/XAverius_PROJECT_CONTEXT.md
- Starfleetctl: https://github.com/mpbt-hq/starfleetctl
- XAverius spec: https://github.com/metux/xav-design/tree/docs/restructure-2026-10-21
- Workspace skills: https://github.com/X11Libre/mpbt-workspace/tree/mtx/agent-config/.claude/skills

## End-of-turn update discipline

Update this file only when durable knowledge, ownership, decisions or next steps materially change. Keep it dated and concise. Separate verified facts, owner-reported context, interpretation and proposals. Never claim acknowledgement, process state, CI, tests or implementation without direct evidence.
