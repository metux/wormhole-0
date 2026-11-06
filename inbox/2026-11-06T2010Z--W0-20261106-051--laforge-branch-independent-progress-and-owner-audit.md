# W0-20261106-051 — Laforge: implementation progress must be committed on isolated branch; owner audits every branch

**Owner:** Laforge (sole Starfleetctl implementation writer).  
**Coordination:** Enterprise local coordination only.  
**Technical reviewer/acceptance:** project owner (assisted by ChatGPT).  
**Priority:** urgent; continue without waiting for another user prompt.

## Current verified repository state
At task creation, `mpbt-hq/starfleetctl` exposes these branches:
- `master` at `500f254f8ee5676cea720c67750c0b71adddc38f`
- `starfleetctl-api-mode` at `67754734e373b704dcc4a71f8268bb21c653f6df`
- `wip/capture-first` at `532a78e4600b3ff32ad354af5a9aae2164de0851`
- `wip/modelproxy-provider-refactor` at `e901e2c59e7ba109a70b13ed2fb6c34e85cb7586`
- `doc/web-ui` at `00a5be4b371a2df0015b1e85c3f2fdf267228ffd`
- `fix/zen-session-header` at `eec3db208ba218875c8e94ccb58a7f073b416e42`
- `submit/agent-bus-health` at `0f900531742624a5704629f82bdf368dc1c8f0f6`
- `submit/session-run-attach` at `0570ee5c40f304fb5f889d225465e9728c9ea4b4`

No dedicated Laforge Wormhole implementation branch or corresponding PR was visible in the current remote listing. Recent master commits were unrelated to the assigned Wormhole manager implementation. This is a status observation, not proof that no local unpushed work exists.

## Required actions
1. Acknowledge this task and W0-043/W0-042. Report whether any implementation or complete implementation plan exists locally but is not pushed. If so, identify exact worktree, branch, HEAD and status; do not conceal or discard it.
2. Create/use one dedicated isolated branch/worktree for the Wormhole manager. Do not write to `master`; preserve existing master history. Laforge remains the sole writer for this implementation.
3. Continue the full W0-043 implementation scope, honoring W0-017/018/019/024/026/028/031/035/039/042 and all later clarifications:
   - config loader, defaults, schema and safe reload;
   - `.starfleet-ai/var/wormholes/<id>/` runtime paths and config/ignore integration;
   - safe managed clone/fetch/rebase/push without destructive reset/clean, broad add, or unsafe staging;
   - real `Bus.PostExternal()` integration through Comms API with durable journal, idempotency, tombstones and fanout;
   - W0-026 sequence recovery, identity and recipient validation;
   - crash-safe incoming/outgoing message semantics;
   - worker daemon singleton/PID/start-time/readiness/signals/stop/backoff;
   - real CLI/service/autostart integration, documentation and extensive tests/fault injection/race tests.
4. Work in small coherent packages. For each package: implement, test, inspect exact staged diff, commit, push, verify remote SHA, then continue. Do not wait for Enterprise audit or another user turn.
5. Open a PR against master only when the implementation is complete enough for review. Do not merge; merge authority remains with the owner.
6. Provide exact commands and results, branch/worktree paths, commit SHAs, test evidence, and remaining blockers. Do not report completion based only on a plan, comments, or commit message.

## Audit model
The owner/ChatGPT independently audits every new Laforge commit on every branch/worktree that becomes remotely visible, not just master or PR branches. Enterprise is not a technical auditor and is not an approval gate; it only coordinates locally, refreshes Wormhole before coordination ticks, distributes instructions and nudges stalled agents.

## Scope discipline
No unrelated refactors, mass formatting, history rewriting, destructive cleanup, or master writes. Do not weaken W0-024/W0-026 crash-safety, identity, recipient-validation, sequencing or durability requirements.

**Immediate expected deliverable:** acknowledgement plus a dedicated branch/worktree and the first small, tested, pushed implementation package (or a factual report explaining any existing local work and why it is not yet pushed). Then continue autonomously.
