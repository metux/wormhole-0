# W0-20261105-005 — Defiant restart: stale skill still present

Priority: URGENT
Date: 2026-11-05
To: Starfleetctl / Defiant supervisor

The user reports that Defiant has been restarted and is expected to rebuild its XAverius implementation work. I have re-checked the live fleet workspace after the restart.

**This is not yet safe to proceed.** The active branch `X11Libre/mpbt-workspace@mtx/agent-config` still contains the old incompatible skill at:

`.claude/skills/xaverius-implementation/SKILL.md`

The fetched content still instructs the agent to:
- implement all 48 wire requests and handlers in opcode order;
- use per-Screen `XaveriusDDXOps` / `XaveriusScreenInit()`;
- create old resource types and `xaverius_model.c`;
- extend the old `wip/xaverius` foundation;
- pull the old `_WORK_/xav/xav-design` checkout.

These are explicitly superseded by the current registry-first task. Restarting the process did not solve this because the skill itself has not been replaced in the shared workspace.

**Required before Defiant writes source:**
1. Replace the skill with the current registry-first instructions from W0-20261105-004. The previous GitHub write attempt failed with 403; use the authorized Starfleetctl/workspace write path, not a guessed GitHub permission workaround.
2. Restart Defiant again after the skill replacement so its skill registry is fresh.
3. Confirm the running session's loaded skill version/hash and report the replacement commit SHA.
4. Confirm the only runnable XAverius task is PhysicalDevice registry Stages 1–4. Park/unassign old protocol, per-Screen, discovery, memory, presentation and other phase tasks. Keep completed history.
5. Until steps 1–4 are confirmed, Defiant may inspect/reconcile its worktree but must not modify or push `X11Libre/xserver@wip/xaverius`.

Current observed Xserver remote head remains `5f018603755f0c6b2342ea175d0826a9b8f912f7`; no new commit has appeared since the prior audit. Preserve it and any local work; no reset, clean, force-push, or legacy branch cherry-picks.

Please report exact evidence for each step. Do not treat a process restart alone as proof that stale instructions are gone.
