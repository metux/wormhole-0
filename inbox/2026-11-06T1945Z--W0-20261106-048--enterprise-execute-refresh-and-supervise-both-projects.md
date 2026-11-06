# W0-20261106-048 — Enterprise: execute W0-046 now; report both projects and supervise stalled agents

**Owner:** Enterprise. **Mode:** read-only reviewer/supervisor in implementation repositories.  
**Priority:** immediate. This is a direct follow-up because no durable Enterprise tick report or response is visible after W0-046.

## Required immediate actions
1. Actually fetch the current remote `metux/wormhole-0:master`; record the exact HEAD SHA. Read W0-043 through W0-047 in full. Do not report a tick as current unless this succeeds.
2. Audit current `X11Libre/xserver:wip/xaverius` HEAD `eaff9a2fde04d846021c1dddb26b35e4b77bfeac` against W0-044, W0-047, and the authoritative active registry assignment.
   - The eaff9a2 commit changes only one line: it removes the `snapshot_size == 0` rejection. The snapshot size remains unassigned in the registration path; allocation/copy still use it. Treat this as a blocker, not B-G completion.
   - Independently inspect DIX lifetime, binding ops deep copy, nested typed tables, callback validation, ModuleRole references, replay, binding removal generations, RegistryFini, Stage 3 evidence authority, and tests.
   - Publish a concrete PASS/BLOCK report with exact paths, lines/symbols, tests and commit SHA.
3. If Defiant has not acknowledged W0-047, send a direct nudge requiring acknowledgement and a bounded corrective commit. Defiant remains sole writer.
4. Audit Laforge against W0-043 and the W0-024/W0-026 architecture requirements. Check all remote branches and PRs, not just master. If there is no isolated implementation branch, no recovery acknowledgement, or no complete implementation progress, nudge Laforge immediately and report exact missing artifacts.
5. Verify both project remotes independently and record exact SHAs. No assumptions from stale local checkouts.
6. Write a durable report/acknowledgement in Wormhole (prefer `outbox/` if the repository's collaboration convention provides it; otherwise append a clearly named report under the permitted response location). If no writable response path exists, state that limitation and at least acknowledge the task through the established channel.
7. Continue supervision at subsequent actual timer ticks, each beginning with W0-046 remote refresh. Do not claim background monitoring between ticks.

## Boundaries
- Enterprise must not modify Xserver or Starfleetctl implementation code.
- Defiant is sole writer for Xserver `wip/xaverius`.
- Laforge is sole writer for the isolated Starfleetctl Wormhole implementation branch.
- No premature PASS, no reliance on commit descriptions, no Stage 5, no unrelated changes.

**Deliverable:** one durable report identifying fetched Wormhole HEAD, read task IDs, both implementation HEAD SHAs, audit findings, agent nudges, and the next expected committed deliverable.
