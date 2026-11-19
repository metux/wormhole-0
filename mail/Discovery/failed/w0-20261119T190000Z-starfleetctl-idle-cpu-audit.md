# Performance audit: idle starfleetctl CPU / Wormhole polling

Please investigate performance reports: multiple long-lived starfleetctl processes reportedly consume persistently high CPU/load even while the fleet is idle. Wormhole is a suspected contributor. This is an investigation first; do not change code or configuration yet.

Repo/branch: mpbt-hq/starfleetctl, work/wormhole-manager-w0-017 (current observed head 127b3ed35641f52eee21197a2e4171b848626535).

Initial code-review leads to verify, not assume:
1. internal/wormhole/wormhole.go RunLoop ticks every config.sync.interval; default is 5s. Each tick calls processPendingMessages.
2. processPendingMessages -> Spool.RunCycleForShips / runOnce -> GitSync.SyncLocked; every cycle does ensure clone, git status/recovery, git fetch, rebase, push even if no new messages. Assess actual idle cost and subprocess count.
3. internal/comms/monitor.go comms monitor-loop scans all message records every 2s; fleet-watch scans all status records every 2s. Could be separate persistent starfleetctl processes.
4. fragments/opencode-plugins/starfleet-dispatch.ts pollRetryStatus has a 2s interval and calls tickLog on every tick; tickLog synchronously spawns starfleetctl comms dispatch --stdin. Poll/inbox and log-monitor timers also spawn CLI subprocesses. Determine whether this is material per OpenCode ship, even when idle.
5. Verify daemon uniqueness across workspace roots, PID/start lock behavior, and whether more than one worker can run for one root. Check for loops that spin when config values are zero/invalid, timers that overlap, and retry storms on failed git/network commands.

Please provide evidence-based report:
- Identify long-lived process entrypoints and expected idle wake frequency.
- Quantify worst-case subprocesses / filesystem scans / git network calls per minute per ship and per workspace.
- Separate actual busy loops from expected periodic polling.
- Recommend smallest safe changes (e.g. slower/adaptive idle polling, avoid tick logging on every retry poll, skip Git fetch when no relevant state/remote changes, or explicit opt-in monitor loops) with tradeoffs.
- Suggest exact diagnostics the operator can run (ps -eo pid,ppid,%cpu,etime,args; pstree; strace -c/-p; perf) without killing/restarting anything.
- Do not make changes, restart/stop processes, or merge PRs. Publish report to outbox with commit/diff/line evidence.
