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
- Current head: `d161b1b81292d11c5f375684db05df1d59c7f9e1`
- Draft PR: https://github.com/mpbt-hq/starfleetctl/pull/23
- Plugin version in the canonical fragment: `2.5.19`.
- The canonical plugin source is `fragments/opencode-plugins/starfleet-dispatch.ts`; deployed `.opencode/plugins/` files are generated. The misleading fragment header was corrected. See decision `decisions/2026-11-25--starfleetctl-plugin-source-of-truth.md`.

**Implemented on the branch:**
- Generic Go event polling protocol v1 via `events.poll`, with stable event IDs, source metadata, payload, and generic actions.
- Event types: `prompt.inject` (Go-formatted complete prompt string, `start_turn` intent), `command.execute` (transitional preservation of existing command/model shorthand behavior), and `unsupported` (unknown types remain pending).
- `init` returns versioned `events` alongside legacy `messages`; the old `inbox` command remains for older plugins.
- Go owns Comms prompt formatting, sender/context prefix and separators. The plugin validates and concatenates supplied prompt parts without adding Comms formatting.
- Both V1 OpenCode adapters share one event coordinator, single-flight poller, queue, ordering/deduplication, leases, and dispatcher.
- Startup events and live polls are deduplicated; failed handoffs release leases; unsupported event types are not acknowledged.
- The new plugin uses `events.poll` and falls back to legacy `inbox` only when paired with an older CLI.
- Runtime helper modules and build-only `.d.mts` declarations are packaged by bootstrap.

**Runtime compatibility evidence:**
- Discovery read-only probe: OpenCode CLI `1.19.3`, resolved `@opencode-ai/plugin` SDK `1.19.3`, timestamp `2026-11-25T17:22:47Z`.
- Upstream tag `v1.19.3` confirms the existing V1 hooks and `promptAsync` API.
- `session.idle` is session-scoped and carries no Comms event ID; it is not sufficient proof that a particular directive completed.
- Durable decisions: `decisions/2026-11-25--opencode-1.19.3-compatibility.md`, `decisions/2026-11-25--opencode-plugin-helper-packaging.md`, `decisions/2026-11-25--comms-event-poll-protocol-v1.md`, and `decisions/2026-11-25--starfleetctl-plugin-source-of-truth.md`.

**Exact-head CI evidence:**
- Code head `542298bf77ee3de68518121c4a0059f4e79f8001`: all three GitHub Actions jobs passed.
- Go 1.22.x and stable: build, vet, and full test suites passed.
- Plugin bundle/typecheck passed; 70 plugin tests passed.
- Final head `d161b1b81292d11c5f375684db05df1d59c7f9e1` (including documentation updates): all three GitHub Actions jobs passed.
- No local execution is claimed.

Discovery reported the exact-head read-only review complete for `d161b1b81292d11c5f375684db05df1d59c7f9e1`, with all six requested criteria PASS, in receipt `w0-20261125T181934Z-5a19744b8c45d8fc` (published to `mail/ChatGPT/new/`). The detailed report with file/line evidence and personally run test commands was not present in the reply; a follow-up request for that immutable report was published as `w0-20261125T182900Z-pr23-review-report-detail`. Treat the PASS receipt as a positive status only, not as a substitute for the detailed report. PR #23 remains Draft/open pending review artifact reconciliation and owner decision.

**Remaining work:** issue https://github.com/mpbt-hq/starfleetctl/issues/24 tracks durable offered/submitted/turn-started/completed/failed lifecycle, restart recovery, and correlated end-to-end tests proving a specific event's model turn started and completed. The proposed correlation design is recorded in `decisions/2026-11-25--comms-event-lifecycle-correlation.md`. Additional OpenCode v1.19.3 source inspection confirmed `experimental.chat.system.transform` runs before `chat.params` in `packages/opencode/src/session/llm/request.ts`. Although `chat.params` carries `message: UserMessage` with `id` and `sessionID`, it is too late to correlate that same transform invocation. `message.updated` remains an unverified candidate for pre-transform correlation; do not implement the `chat.params` approach. See `decisions/2026-11-25--opencode-current-turn-correlation-ordering.md` (which supersedes the earlier candidate note). Branch `feat/comms-event-lifecycle` has been created from the PR #23 head for issue #24, but lifecycle code is not implemented yet. Neither `seen_mark` nor `promptAsync` returning is completion proof.

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
