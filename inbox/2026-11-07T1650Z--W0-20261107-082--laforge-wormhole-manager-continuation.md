# W0-082 — Continue Wormhole Manager implementation to acceptance

To: LaForge
Priority: high
Date: 2026-11-07

This is a focused continuation of W0-076. Continue on the existing `work/wormhole-manager-w0-017` branch. Do not move work to master, force-push, rewrite history, or merge. Do not restart, relaunch, detach, replace, or otherwise disturb your console-attached session.

## Current observed state

Latest observed remote branch head: `63c3e263f5f245bd52b71660cebd2e40c43297f7` (parent `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`). No newer pushed commit was visible at the latest check.

## Continue, do not merely report

Complete W0-076 acceptance across lifecycle, configuration, Git sync, durable Comms delivery, CLI/service behavior, tests, and documentation. In particular:
- make start/stop/restart/status/autostart correct across processes, with duplicate-start prevention, stale PID safety, cleanup and subprocess tests;
- use one authoritative yaml.v3 config model, load configured repositories and worker settings, validate/default consistently;
- make Git fetch/rebase/branch validation and local-change preservation coherent, propagate failures, honor timeouts/retries, and test with local bare repositories;
- deliver through the existing `internal/comms.Bus.PostExternal()`; preserve its durable journal, idempotency, tombstones and fanout; acknowledge source messages only after durable post; retry safely and deduplicate after restart; preserve original payload/provenance;
- implement real `wormhole sync`, status, exit codes and shutdown/cancellation;
- add focused tests and update docs truthfully.

Do not add a second outbox or write directly into Comms inbox files. Do not expand scope into the future Comms addressing/routing proposal or SOP/trust policy; those are explicitly deferred until Wormhole is complete.

Work in coherent, narrowly scoped commits; push the same branch and verify remote HEAD and CI. Continue until all assigned acceptance criteria are met, not just until one partial commit exists. Report exact blockers or protected ask IDs/text; never claim completion without evidence.
