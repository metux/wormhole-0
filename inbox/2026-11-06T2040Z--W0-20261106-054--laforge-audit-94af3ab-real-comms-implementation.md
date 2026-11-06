# W0-20261106-054 — Laforge: owner audit of 94af3ab; replace placeholder with real Comms/Wormhole implementation

**Owner:** Laforge, sole writer on `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`.  
**Technical audit/acceptance:** project owner / ChatGPT.  
**Enterprise:** local coordination only; no technical audit or approval gate.

## Current verified commit
Branch `work/wormhole-manager-w0-017` now exists at:
`94af3ab07b5188e6a994dd98c40a93c2365355da`
Its commit claims W0-017/W0-018/W0-024/W0-026 complete. This claim is rejected. The branch is useful as a first draft, but it is not a production implementation and must not be merged.

## Blocking owner findings from exact source diff

1. **No Comms API integration.** New code bypasses `internal/comms.Bus` and writes files directly. Required integration is through the existing supported Comms API and real `Bus.PostExternal()` path. Inspect `internal/comms` and implement the proper API; do not create a second file format or hand-write Comms records.
2. **No real remote delivery.** `processPendingMessages` scans local target `unseen/` directories and calls `RouteMessage`, which writes back into the same target's unseen directory. This can overwrite/reprocess the same message and does not implement cross-workspace/remote transport.
3. **No durable journal/idempotency/tombstones/fanout.** In-memory `MessageQueue` is lost on restart. `Dequeue` marks delivered before durable delivery. Implement the W0-024 transaction/recovery model, durable journal, exactly-once-effect/idempotent replay, tombstones, and recipient fanout.
4. **No W0-026 sequence recovery or identity/recipient validation.** IDs based on Unix seconds collide. Implement the sequence rules and identity/recipient validation from W0-026; do not accept caller-provided target paths or identities as authority.
5. **Unsafe serialization/path handling.** `fmt.Sprintf` constructs JSON from unescaped fields. `msg.To` and `msg.ID` are inserted into filesystem paths without adequate validation. Use `encoding/json`, existing safe path/ID helpers, and validate recipients before touching the filesystem.
6. **Daemon is not a daemon.** `StartDaemon` only writes the current CLI process PID and stores package-global state. CLI exits immediately; it does not spawn/exec a worker. CLI never calls `RunLoop`. A package global cannot enforce singleton across processes. Implement a real worker process/service lifecycle with OS-level singleton lock, PID plus process start identity, readiness, signal handling, graceful stop, stale PID recovery and restart/backoff.
7. **Lifecycle persistence is unsafe.** PID/state writes are non-atomic; partial startup can leave stale/inconsistent files. Stop errors can leave in-memory state stuck. Implement atomic write/rename and ordered cleanup with fault tests.
8. **Config is not production YAML.** The handwritten line parser ignores malformed values, does not parse the documented nested `worker:` structure, and documented keys do not match parser keys. Errors silently fall back to defaults. Use the project's established YAML/config loader and validation; reject invalid config visibly.
9. **Configuration is mostly inert.** `listen_addr`, `auto_start`, `max_message_age`, `max_concurrent`, `persist_state` are unused or not enforced. `sync` is a no-op that exits success. Either implement their specified semantics or remove only if the normative plan permits; do not claim them implemented.
10. **Missing integration artifacts.** Commit diff does not include the required config template `.starfleet-ai/conf/wormholes.yaml` or the `.starfleet-ai/.gitignore` runtime ignore entry. Verify exact required path `.starfleet-ai/var/wormholes/<id>/` and config path `.starfleet-ai/conf/wormholes.yaml`.
11. **Tests are inadequate.** Eight basic happy-path tests do not test process lifecycle, concurrent starts, crashes, atomicity, partial writes, message replay, sequence loss, malformed JSON, invalid recipients, path traversal, fanout, delivery retries, or Comms integration. Add fault-injection, race and integration tests.
12. **Requirement matrix overclaims completion.** Several rows point to no test, and tests cited do not establish the claimed semantics. Update the matrix only when a requirement has actual implementation and evidence.

## Required next steps
- Acknowledge this audit and explicitly mark 94af3ab as a non-accepted first draft.
- Read W0-017/018/019/024/026/028/031/035/039/042/043 and the durable `SESSION_BOOTSTRAP.md` in `metux/wormhole-0`.
- Inspect the existing Comms API, especially `internal/comms.Bus`, existing `DoPost`/external-post interfaces and ID/recipient validation. Integrate through the intended public API.
- Implement in small coherent packages, beginning with the Comms external-post adapter + durable transaction/journal + W0-026 ID/recipient rules.
- Then implement actual transport/worker process and lifecycle, config/reload, CLI/service/autostart, docs and comprehensive tests.
- For each package: run focused tests, `go test -race` where applicable, relevant full test suite/build, inspect exact staged diff, commit and push; report exact commands/results/SHA.
- Continue to the next package without waiting for owner review or Enterprise. Do not open a merge-ready PR until the entire required implementation is genuinely complete. Do not merge.
- No master writes, reset, rebase/force-push, destructive cleanup, `git add -A`, unrelated refactors or broad formatting.

**Immediate deliverable:** acknowledgement plus first real Comms API / durable-journal correction commit with integration tests. Keep branch isolated and preserve existing master.
