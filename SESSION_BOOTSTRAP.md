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
6. Apply decision `decisions/2026-11-27--mandatory-wormhole-message-preflight.md`; validate every outbound message with the bundled communication-skill validator before publication.

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
- Before every outbound message commit, run `python3 .claude/skills/wormhole-communication/scripts/validate_message.py mail/<recipient>/new/<message-id>.md --recipient <recipient>`; do not publish unless it exits zero and prints `VALID`. All front-matter values must be JSON double-quoted strings.

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

**Remaining work:** issue https://github.com/mpbt-hq/starfleetctl/issues/24 tracks durable offered/submitted/turn-started/completed/failed lifecycle, restart recovery, and correlated end-to-end tests proving a specific event's model turn started and completed. The proposed correlation design is recorded in `decisions/2026-11-25--comms-event-lifecycle-correlation.md`. Additional OpenCode v1.19.3 source inspection confirmed `experimental.chat.system.transform` runs before `chat.params` in `packages/opencode/src/session/llm/request.ts`. Although `chat.params` carries `message: UserMessage` with `id` and `sessionID`, it is too late to correlate that same transform invocation. OpenCode v1.19.3 source confirms the user-message `message.updated` event is published before the request loop reaches system transformation, but does not prove the plugin's asynchronous event callback completes first. It remains the candidate; verify plugin event delivery ordering or query `session.messages` in the transform hook before implementation. Do not implement the `chat.params` approach. See `decisions/2026-11-25--opencode-user-message-event-order.md` and `decisions/2026-11-25--opencode-current-turn-correlation-ordering.md`. Branch `feat/comms-event-lifecycle` has been created from the PR #23 head for issue #24, but lifecycle code is not implemented yet. Neither `seen_mark` nor `promptAsync` returning is completion proof.

## Dedicated X11Libre HDR session

HDR/color-management work is handled in its own ChatGPT session, separate from Starfleetctl and XAverius. For that session, read `X11LIBRE_HDR_PROJECT_CONTEXT.md` before acting; it records the 2026-11-26 verified implementation state, PoC blockers, open design decisions and proposed milestones. Refresh current Xserver `master` and the workspace HDR tracker before relying on recorded SHAs. The HDR context is a handoff, not an implementation authorization; owner approval is required for unresolved design choices.

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


## Continuation checkpoint — 2026-11-25 19:42Z

### XAverius live review follow-up
- The workspace dashboard file `.starfleet-ai/dashboard/topics/task-physicaldevice-registry-stages-1-4-xaverius.md` on `X11Libre/mpbt-workspace:mtx/agent-config` still lists task `7f540f6aaa` as `in-progress`, assigned solely to Defiant. Its last appended progress entry is 70% from 2026-11-06; do not treat that percentage as current.
- Re-read current remote `X11Libre/xserver:wip/xaverius` source. `Xext/xaverius/registry.c` still contains the snapshot free-before-copy UAF, early free of an existing preferred binding, non-exact descriptor-size/version error handling, no-op DIX destructor/direct PhysicalDevice AddResource value, and RegistryFini freeing bindings before quiescence. The current source snapshot therefore still matches the five blockers in review message `w0-20261125T174209Z-defiant-registry-review-blockers`. The connector confirmed the branch is 64 commits ahead of xserver master, but did not expose the exact branch-tip SHA in this check; do not invent one.
- The review message `w0-20261125T174209Z-defiant-registry-review-blockers` is under `mail/Defiant/failed/`, not processed; Defiant has not thereby acknowledged it.
- Fresh task message `w0-20261125T193700Z-xaverius-review-followup` also entered `mail/Defiant/failed/`. A shorter retry without reply chaining, `w0-20261125T194200Z-xaverius-review-followup-retry`, also entered `failed/`. No Defiant acknowledgement or new source/test evidence is verified. Enterprise was asked in `w0-20261125T194201Z-defiant-failure-diagnostic` to inspect manager-owned diagnostics and report the exact import failure. That request subsequently appeared in `mail/Enterprise/processed/`, but no diagnostic reply or per-ship model acknowledgement was verified.
- Do not restart Defiant. Wait for Enterprise's failure diagnosis and use the existing session/worktree only.

### Fleet round-trip check
- Messages `w0-20261125T193701Z-pr23-review-evidence` (Discovery), `w0-20261125T193702Z-voyager-status-roundtrip` (Voyager), and `w0-20261125T193703Z-fleet-roundtrip-coordination` (Enterprise) are under their respective `mail/<ship>/processed/` directories. This proves Comms import only, not model acknowledgement or task progress.
- Enterprise was asked to gather per-ship publication/import/session-injection/model-acknowledgement/action evidence, with event IDs, plugin version, PID/session and task board. No reply was verified at this checkpoint.
- The failed Defiant messages are not evidence that the other ships have processed or replied.

### Starfleetctl lifecycle work
- PR #23 remains open/draft at exact head `d161b1b81292d11c5f375684db05df1d59c7f9e1`; exact-head Actions run `45671982146` completed successfully. GitHub review list and inline review threads were empty at this checkpoint.
- Branch `feat/comms-event-lifecycle` compares identical to `feat/comms-event-pipeline` (zero commits ahead/behind); issue #24 remains open and lifecycle implementation has not started.
- Re-fetched `fragments/opencode-plugins/starfleet-dispatch.ts` on that branch: the new-turn path calls `promptAsync` without a deterministic `messageID` and calls `seen_mark` immediately after successful submission; the current-turn transform path also calls `seen_mark` immediately after injection. Neither is correlated model-turn completion. Issue #24's durable lifecycle/correlation work remains necessary.
- Discovery was asked in `w0-20261125T193701Z-pr23-review-evidence` to provide the detailed exact-head review report and commands; the message is processed, but no model acknowledgement/report is yet verified.

### Operational caveats
- The XAverius skill in `X11Libre/mpbt-workspace:mtx/agent-config` still has a canonical API summary containing `XaPhysicalDevicePublish()`, while current Xserver source declares `XaveriusPhysicalDevicePublish()`; reconcile only against normative spec §7, not skill wording.
- For all new Wormhole messages, track publication, manager import, active-session delivery, explicit model acknowledgement, and action as separate states. Never infer the latter stages from `processed/` or `seen_mark`.


## Starfleetctl-only continuation checkpoint — 2026-11-25

This checkpoint records only the Starfleetctl workstream. The current ChatGPT session is explicitly scoped to Starfleetctl; do not inspect, message, assign, or modify XAverius/Xserver work from that session.

- Re-read the current Wormhole protocol and the workspace `wormhole-communication` skill. Active messages are immutable under `mail/<ship>/new/`; `processed/` proves import only. Actionable requests require explicit acknowledgement with interpretation, scope, exclusions, target, prerequisites and blockers; statuses are exactly `received|acknowledged|in_progress|blocked|completed|rejected|cancelled`.
- Refreshed PR #23: it remains open/Draft in the repository search. Its changed-file list contains 14 files. GitHub currently reports no submitted PR reviews and no inline review threads. The recorded exact-head Actions run `45671982146` for `d161b1b81292d11c5f375684db05df1d59c7f9e1` is completed/successful; this is CI evidence only, not a code review or local test run.
- Discovery's request `w0-20261125T193701Z-pr23-review-evidence` is in `mail/Discovery/processed/`. A repository search did not find a detailed review artifact with the requested six criterion-by-criterion evidence, personally run commands/results, limitations and findings. No model acknowledgement or detailed report is verified. Do not treat the earlier PASS receipt as a merge-ready review.
- Branch comparison confirms `feat/comms-event-lifecycle` is identical to `feat/comms-event-pipeline` (0 ahead/behind). The latter is 84 commits ahead of `master`; issue #24 remains open. Lifecycle implementation has not started.
- Direct PR diff inspection confirms the new-turn adapter calls `promptAsync` without a deterministic `messageID` (plugin lines 895–903) and then calls `seen_mark` after successful submission (906–909). The current-turn adapter pushes prompt text into `output.system` and then calls `seen_mark` (1004–1008). These are submission/injection acknowledgements, not proof that the associated model turn started or completed. This is the central reason issue #24 remains necessary; use correlated assistant completion, not session idle or `seen_mark`, as completion evidence.
- No local shell, worktree, ship process, or live Comms session was available in this check. No tests were run locally and no ship was restarted or relaunched.

Next Starfleetctl actions: obtain Discovery's actual immutable PR #23 review report (or explicitly mark it blocked); then implement issue #24 on the existing lifecycle branch with deterministic new-turn message IDs, current-turn message correlation, durable idempotent transitions/leases/recovery, and tests proving a specific event's correlated assistant turn completed. Re-check branch ownership before writing; do not merge PR #23 without the review artifact and owner decision.


## Starfleetctl continuation — 2026-11-26 08:22Z

- Rechecked PR #23 via the pull-request API: still open/Draft, base `master` at `bf6144e...`, head `feat/comms-event-pipeline` at `d161b1b81292d11c5f375684db05df1d59c7f9e1`, mergeable according to GitHub. No submitted reviews or inline review threads. Exact-head Actions run `45671982146` remains completed/successful. No merge performed.
- Issue #24 remains open and unassigned; its five owner comments include the verified OpenCode hook-order correction and the distinction between assistant-turn completion and task completion. No lifecycle code is present on `feat/comms-event-lifecycle`; comparison against master still shows the same 84-commit PR #23 diff.
- Published Discovery follow-up `w0-20261126T082200Z-pr23-review-followup` to `mail/Discovery/new/` requesting the explicit acknowledgement and detailed exact-head review artifact. Publication is verified by commit `66eb01d5894efa5d5e6ad4c69c8b8f859e91da4e`; no claim of import, acknowledgement or review completion yet.
- New source inspection of OpenCode tag `v1.19.3` found a stronger current-turn correlation candidate: the awaited `chat.message` hook receives `output.message.id/sessionID`; `SessionPrompt.createUserMessage()` awaits that hook before persisting the user message, and `SessionPrompt.prompt()` awaits message creation before entering the loop where `experimental.chat.system.transform` runs. This avoids relying on asynchronous `message.updated` callback completion and avoids the disproven `chat.params` ordering. Durable decision: `decisions/2026-11-26--opencode-chat-message-hook-correlation.md`; finding also added to issue #24 comment `7186545316`.
- This is upstream source-order evidence only, not a runtime test. Implementation must still test session matching, multiple pending messages, retries, regeneration, compaction/autocontinue, concurrency and restart recovery. Completion still requires a correlated assistant `message.updated` with matching `parentID` and `time.completed`; turn completion is not task completion.
- No local workspace or ship runtime was available. No local tests were run and no ship was restarted. Session remains Starfleetctl-only; no XAverius/Xserver work was touched.


## Starfleetctl PR #23 clean-history replacement — 2026-11-26 09:57Z

- Owner approved replacing PR #23's 84-commit history without force-pushing. Created `mpbt-hq/starfleetctl` branch `feat/comms-event-pipeline-clean-history` at `30b8b3dc2f3d4f3e1a2609a63bd11f1a6995056f` and draft PR [#25](https://github.com/mpbt-hq/starfleetctl/pull/25).
- Reconstructed the implementation as six semantic commits: event-pipeline design docs; Go event model; Go polling endpoint plus tests; shared plugin prompt-preparation helpers; plugin event coordinator/adapter integration plus tests and bootstrap packaging coverage; CI plugin-test integration.
- Verified the new head's Git tree is exactly identical to old PR #23: `eeb48dadfde77bb06ee80a7c3ef0f07cce27c293`. Both diffs contain the same 14 files, 1,488 additions and 197 deletions. No source content changed during reconstruction.
- Both exact-head Actions runs `45796376211` and `45796386407` passed all three jobs: Go build/vet/full tests on stable and Go 1.22.x, plugin bundle/typecheck and plugin unit tests.
- Closed PR #23 as superseded (not merged); added cross-links/comments to both PRs. PR #25 remains Draft and open. Published a fresh read-only request to Discovery for a detailed report on the new exact head: `w0-20261126T095710Z-pr25-review-detail` (publication commit `b45440a6d95887b1e6d14a0d43b941567bbdb322`). The manager moved it to `mail/Discovery/processed/`, proving import only; no explicit model acknowledgement or detailed report is yet visible. Do not merge #25 until the detailed review evidence is reconciled and the owner authorizes it.
- No local tests were run; all test evidence above is from GitHub Actions. No local worktree or ship runtime was accessed.


## Starfleetctl task-marker decision and PR #29 — 2026-11-26 10:51Z

- Owner decision is recorded in `decisions/2026-11-26--dashboard-topic-reference-not-acceptance.md`: a `Dashboard-Topic` marker in a commission message is a reference only. Delivery/import/injection does not accept or start the task and must not automatically associate it with the ship.
- Acceptance is explicit via `starfleetctl task begin <slug>`; task progress/completion use the existing lifecycle. Plugin startup, `comms init`, session recreation, model telemetry, and heartbeats must not infer or overwrite task association or board status. A new ship with no existing status defaults to idle.
- Ready-for-review PR [#29](https://github.com/mpbt-hq/starfleetctl/pull/29), branch `fix/task-topic-reference-explicit-acceptance`, head `a7945d5800ef245c2faf98bb41e0d661effb8003`, base `master` at `bf6144ebae79c41a166ce6695f9affcf67402bcd`. One semantic commit, ten changed files; executable mode of `scripts/check-opencode-plugin.sh` is preserved. GitHub reports mergeable; no review or inline threads yet. Not merged.
- PR #29 removes automatic marker parsing and plugin-local `currentTask`, removes inferred working/idle state from plugin startup/session/model/heartbeat telemetry, and preserves task status across restarts. It runs `comms init` before refreshing PID/heartbeat, so stale records are pruned before the new process refreshes liveness. Go `comms init` preserves an existing live status/task/note; new/offline records default idle.
- Generated commission messages label Dashboard-Topic as reference-only and instruct explicit `task begin`; flagship messages retain delegation. Node and Go regression tests cover marker handling, telemetry/startup ordering, commission wording, and init status/task/note preservation. Task skill and Comms SOP are updated.
- Exact-head Actions run `45804837499` completed successfully: plugin-typecheck / `make check-plugin`, Go build, vet, and full tests on stable and Go 1.22.x all passed. No local test run.
- PRs #26, #27, and #28 were closed as superseded, not merged. PR #25 remains Draft/open. After PR #29 is merged, refresh all Starfleetctl branch heads and rebase the active dependent branches (including `feat/comms-event-pipeline-clean-history` / PR #25 and the lifecycle branch) onto the new master. Preserve the no-force-push rule: if a published branch requires history rewriting, use a replacement branch/PR rather than force-pushing. Inventory obsolete/superseded branches before deciding whether they still need updating.
- No local worktree was used; no other work branch was rebased or modified. Session remains Starfleetctl-only.



## Starfleetctl runtime / model-health tracking task — 2026-11-26 11:07Z

- Owner requested a follow-up plan for dashboard-visible ship activity and model/provider failures. Canonical task: [Starfleetctl issue #30](https://github.com/mpbt-hq/starfleetctl/issues/30), `feat(status): track ship runtime and model health from turn events`.
- Immutable Wormhole handoff published to `mail/ChatGPT/new/w0-20261126T110700Z-starfleet-runtime-health-issue30.md` (publication commit `c4414e2077df5863e67b36ac442d040367c2e4c3`). This is a pointer/handoff only; issue #30 is the authoritative task record, consistent with COLLABORATION.md.
- Planned model: separate **runtime** (`starting|working|waiting|idle`) from **health** (`healthy|degraded|error|offline`) and from explicit **task lifecycle**. Plugin reports verified observations; Go owns validated transitions, persistence, stale/duplicate event rejection and offline expiry; CLI/dashboard expose the same projection.
- First prerequisite is to verify the exact deployed OpenCode/plugin SDK event contract (names, payloads, ordering, awaited hooks) before implementing turn tracking. Correlate a specific assistant turn; do not infer completion from heartbeat, `seen_mark`, submission return, or unrelated `session.idle`.
- Test matrix includes start/end, provider/model error, retry/degraded, recovery, process exit/heartbeat expiry, restart, duplicate/out-of-order/stale events, and regression that runtime telemetry never mutates task association.
- **Bootstrap/packaging invariant explicitly required by owner:** every new plugin source/helper/runtime file must be added to canonical bootstrap/install/packaging manifests and their tests in the same change; verify generated/deployed plugin contents. Never leave helper modules uninstalled.
- Sequencing: follow PR #29; reconcile PR #25 and issue #24 before choosing the implementation base/branch. Do not merge #29 or restart/relaunch ships without explicit owner approval. No force-push.
- No implementation has started; issue #30 is open and unassigned. The Wormhole message is published, but no import/acknowledgement by another agent is claimed.


### Starfleetctl issue #30 — bootstrap plugin-asset parity test refinement (2026-11-26)

- Owner requested an automatic regression test proving that every canonical plugin runtime file is embedded and installed by bootstrap.
- Added acceptance criteria to [Starfleetctl issue #30](https://github.com/mpbt-hq/starfleetctl/issues/30): recursively discover runtime assets under `fragments/opencode-plugins/`, compare source files with the Go-embedded filesystem, run the real installer in a temporary workspace, compare installed paths and bytes, and verify entry-point registration versus helper-module installation. CI must fail on missing/unexpected assets, omitted runtime extensions, or unregistered entry points.
- Current `master` implementation dynamically scans embedded `.ts`/`.mjs` assets in `verifyOpencodePlugins`; the current focused test `TestFixOpencodePluginsInstallsModuleAssets` still hard-codes helper names. The plugin entry point imports three helpers (`session-resolution.mjs`, `inbox-response.mjs`, `runtime-identity.mjs`), while that test currently lists only two. This is a concrete coverage gap; the discovery-driven parity test should replace the manually maintained helper-name slice.
- The source/embedded/installed comparison must distinguish runtime assets from build/test/config files intentionally present under the source directory. New runtime extensions must not silently fall through as non-runtime.
- This is a planned acceptance criterion only; no implementation or CI run has been performed for it yet.


## Starfleetctl continuation — PR #29 merged, plugin parity PR #31, event-pipeline replacement PR #32 — 2026-11-26 12:05Z

- **PR #29 merged:** [#29](https://github.com/mpbt-hq/starfleetctl/pull/29) merged at `6e693d1d14299763911783beb68b6a9d39a303f5` (2026-11-26T11:55:36Z). Repository only permits rebase merges; merge used rebase with expected head `a7945d5800ef245c2faf98bb41e0d661effb8003`. Exact-head CI `45804837499` passed. Master is now `6e693d1d14299763911783beb68b6a9d39a303f5`.
- **Plugin bootstrap parity:** [PR #31](https://github.com/mpbt-hq/starfleetctl/pull/31), branch `test/bootstrap-plugin-asset-parity`, head `2466562ebb72ccda5953c2b8dfc3af6993d5af8a`, open/ready for review/not merged. Exact-head Actions run `45816662501` completed successfully: plugin-typecheck and Go build/vet/full tests on Go 1.22.x and stable.
  - Recursively discovers runtime assets under `fragments/opencode-plugins/`, compares canonical source, embedded FS, and the complete fresh bootstrap installation; verifies byte identity and entry-point registration/helper non-registration.
  - The installed directory is compared as a complete file set, so non-runtime test/config/declaration files cannot leak into it.
  - Bootstrap detects and removes legacy `.test.ts` / `.test.mjs` files and `.test.ts` plugin registrations left by older versions, preserving unrelated user plugin registrations.
  - Negative tests cover missing installed helper, unregistered entry point, missing/unexpected set entries, unclassified extensions, and migration from a legacy install.
  - Replaced the manually maintained helper-name list with discovery.
  - Found/fixed a real bug: bootstrap previously treated colocated `.test.ts` and `.test.mjs` as runtime plugins, installing/registering tests. Runtime classification now excludes those.
  - Issue #30 has a comment linking PR #31 and its exact CI evidence. PR #31 has not been merged.
- **Event pipeline replacement:** [PR #32](https://github.com/mpbt-hq/starfleetctl/pull/32), branch `feat/comms-event-pipeline-rebased`, head `148f28e2e24d81b9528640217823c63bedda1723`, open/Draft. Based on master `6e693d1...`, ahead 7 / behind 0, 14 changed files. It preserves the six semantic commits from old PR #25 as the second parent of a replacement merge commit; old published branch was not force-pushed or rewritten.
  - Reconciled the two overlapping plugin files with PR #29: no Dashboard-Topic auto-adoption/currentTask cache, heartbeat is liveness-only, startup init precedes PID/liveness refresh, and inferred `working` writes were removed.
  - Removed obsolete regex-only tests expecting auto-adoption; carries forward PR #29's task-marker test and its invocation from `scripts/check-opencode-plugin.sh`; all other non-overlapping PR #29 changes are included from master.
  - Exact-head Actions run `45815962061` completed successfully: plugin-typecheck and Go build/vet/full tests on stable and Go 1.22.x all passed. No local tests were run. PR #32 remains Draft pending independent review.
- Old PR #25 was closed as superseded after PR #32's exact-head CI passed; the old branch/history remains preserved. PR #32 now documents the replacement and green CI. No merge of #32 authorized.
- Branch inventory after PR #29:
  - `feat/comms-event-pipeline-clean-history` remains old PR #25 head `30b8b3dc2f3d4f3e1a2609a63bd11f1a6995056f`; preserved, not force-pushed.
  - `feat/comms-event-lifecycle` and `feat/comms-event-pipeline` both remain at `d161b1b81292d11c5f375684db05df1d59c7f9e1`, same superseded PR #23 implementation tree, 84 commits ahead / 1 behind master, with no unique lifecycle implementation. Do not treat either as an independent lifecycle implementation or blindly rebase it. Issue #24 remains open; create the active lifecycle branch from the resulting master after PR #32 is merged.
  - `backup/master-2026-11-07` remains historical at `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`, 64 commits behind and zero ahead; do not rebase or modify.
- Issue #24 has been updated with the dependency replacement and branch inventory. Its durable lifecycle/correlated-turn implementation has not started.
- Owner's standing plugin invariant remains mandatory: every new plugin runtime/helper file must be discovered by bootstrap packaging and the source/embed/install parity tests in the same change.
- No local worktree, local test run, ship restart, or force-push. No XAverius/Xserver work was touched.


### Starfleetctl follow-up verification — 2026-11-26 12:06Z

- Rechecked [PR #29](https://github.com/mpbt-hq/starfleetctl/pull/29): merged true at `6e693d1d14299763911783beb68b6a9d39a303f5`.
- Rechecked [PR #31](https://github.com/mpbt-hq/starfleetctl/pull/31): open, ready for review, not merged; exact-head run `45815535581` completed success for plugin-typecheck and both Go test matrices.
- Rechecked [PR #32](https://github.com/mpbt-hq/starfleetctl/pull/32): open, Draft, not merged; head `148f28e2e24d81b9528640217823c63bedda1723`, based on master `6e693d1...`, ahead 7 / behind 0; exact-head run `45815962061` completed success for plugin-typecheck and Go build/vet/full tests on stable and Go 1.22.x.
- Closed old PR #25 as superseded (not merged), with a cross-link to PR #32. Its published branch `feat/comms-event-pipeline-clean-history` remains unchanged.
- PR #32 body now records full green CI and the closure of old PR #25. PR #32 remains Draft; no merge performed.
- Issue #30 links PR #31 and its test evidence. Issue #24 records that the two old lifecycle/event-pipeline refs are duplicate superseded PR #23 trees and that the active lifecycle branch should be created from master after PR #32 is merged.
- Remaining branch inventory unchanged: `feat/comms-event-lifecycle` and `feat/comms-event-pipeline` both at `d161b1b81292d11c5f375684db05df1d59c7f9e1` with no unique lifecycle work; historical backup `0765d18...` stays untouched. No force-push, no local tests, no ship restart.


### Starfleetctl PR #31 follow-up — legacy test asset migration — 2026-11-26 12:08Z

- PR #31 head advanced to `2466562ebb72ccda5953c2b8dfc3af6993d5af8a`; latest exact-head CI run `45816662501` is fully green (plugin-typecheck; Go build/vet/full tests on stable and Go 1.22.x).
- Additional migration regression test now seeds a temporary workspace with legacy-installed `.test.ts` / `.test.mjs` files and a registered test entry point, runs the real installer, and verifies those leftovers are removed while unrelated user plugin registrations survive.
- Production `verifyOpencodePlugins` now reports stale test files/registrations, and `fixOpencodePlugins` cleans them. This avoids leaving a test plugin loaded in existing workspaces after upgrading the bootstrap binary.
- PR #31 remains open/ready for review/not merged. PR #32 remains open/Draft/not merged with green exact-head CI run `45815962061`. Old PR #25 remains closed as superseded. No other branch refs were force-pushed.



### Starfleetctl PR #31 merge and PR #32 replacement — 2026-11-26 12:55Z

- PR #31 was merged after its targeted CI retry. Exact head `2466562ebb72ccda5953c2b8dfc3af6993d5af8a`; rerun attempt 2 of Actions run `45816648190` completed success. All three jobs passed: `build-test (stable)`, `build-test (1.22.x)`, and `plugin-typecheck`. Merge used GitHub's rebase method and produced master commit `c18352bb3269524294fda90d6d319ddef92098dd`.
- PR #32 could not be updated in place without rewriting its published merge-based branch. Its branch and history remain unchanged; PR #32 was closed as superseded (not merged).
- Created replacement branch `feat/comms-event-pipeline-rebased-2026-11-26` from the new master and replayed the event-pipeline tree as one commit `d23a8ca73f42a1634307222eb272b42f6bc3de35`. The resulting diff is 13 files. The old PR #32-only helper-list adjustment in `internal/bootstrap/checks_test.go` was intentionally not copied because PR #31's newer recursive discovery/parity tests supersede it. PR #29 explicit task-acceptance behavior and PR #31 bootstrap parity/migration tests remain in the base.
- Opened [PR #33](https://github.com/mpbt-hq/starfleetctl/pull/33) as Draft, based on master `c18352bb3269524294fda90d6d319ddef92098dd`. Exact-head CI is running (Actions runs `45823658776` and `45823617833`); no CI result is claimed yet. Independent review and exact-head green CI are still required.
- Current open PR inventory: only PR #33. PR #32 is closed as superseded; PR #31 and #29 are merged.
- Duplicate historical refs `feat/comms-event-lifecycle` and `feat/comms-event-pipeline` still point to the superseded PR #23 tree `d161b1b81292d11c5f375684db05df1d59c7f9e1`; neither has unique lifecycle code. They were deliberately not rebased. Create the active issue #24 lifecycle branch from the resulting master only after PR #33 is merged. Preserve the old refs until then.
- `feat/comms-event-pipeline-rebased` (old PR #32 head) and `feat/comms-event-pipeline-clean-history` (old PR #25 head) remain unchanged. Historical `backup/master-2026-11-07` remains untouched. No force-pushes, no local tests, and no ship restarts.


### Starfleetctl PR #33 — commit-message cleanup — 2026-11-26 13:37Z

- Owner requested a descriptive semantic commit message and explicitly authorized force-pushes where useful. Recreated the one-commit PR #33 head with the exact same tree `b4fee244963ad0c93fe9538c1e28c66eb97a89b1`, same parent/master `c18352bb3269524294fda90d6d319ddef92098dd`, and updated branch using force-with-lease (expected old head `d23a8ca73f42a1634307222eb272b42f6bc3de35`). New head: `5783f16af0ed45ebbcb0022f19c22b8f16f63daf`.
- New commit subject: `refactor(comms): unify polling and OpenCode event delivery`. Body explains Go-owned prompt formatting/event classification, shared adapter coordination/deduplication, legacy compatibility, packaged helper modules/plugin tests, and explicitly defers durable turn lifecycle to issue #24.
- PR #33 description updated to the new SHA and exact verification state. The diff remains one commit, 13 files, ahead 1 / behind 0; no source tree changes were introduced by the message rewrite. Old PR #32 branch/history remains untouched.
- Previous tree-equivalent SHA `d23a8ca73f42a1634307222eb272b42f6bc3de35` passed all three jobs in Actions run `45823658776` (plugin-typecheck, Go 1.22.x, stable). The rewritten SHA needs its own exact-head CI run; at the time of this update no run was yet returned for the new SHA. PR #33 remains Draft/open, not merged, awaiting exact-head CI and independent review.


### Starfleetctl PR #33 — exact-head CI and review status — 2026-11-26 13:55Z

- Exact current PR #33 head `5783f16af0ed45ebbcb0022f19c22b8f16f63daf` completed Actions run `45830846333` successfully. All three jobs passed: `plugin-typecheck` (including `make check-plugin` and `make test-plugin`), `build-test (stable)`, and `build-test (1.22.x)`; both Go jobs ran build, vet, and full `go test ./... -count=1 -timeout 300s`.
- Rechecked PR #33: open, Draft, mergeable, one commit, 13 changed files; base master remains `c18352bb3269524294fda90d6d319ddef92098dd`; head remains `5783f16af0ed45ebbcb0022f19c22b8f16f63daf`. No submitted reviews, no inline review threads, and no PR comments were present at this check. PR description now records exact-head green CI and accurately states that independent review and owner merge authorization remain outstanding.
- Focused source review checked Go event classification/formatting and non-destructive poll path, shared plugin coordinator and adapter claims/releases, legacy compatibility, helper modules, and CI integration. No immediate correctness blocker was identified in this pass. Durable submitted/turn-started/completed correlation remains explicitly out of scope and belongs to issue #24.
- Issue #24 was refreshed with the new PR #33 dependency/head/CI state. The old `feat/comms-event-lifecycle` branch still points to superseded PR #23 tree `d161b1b81292d11c5f375684db05df1d59c7f9e1`; do not implement from that base. Create the active issue #24 branch from the resulting master only after PR #33 is merged.
- PR #33 remains Draft and unmerged. No merge was authorized or performed. No ships were restarted and no XAverius/Xserver work was touched.
