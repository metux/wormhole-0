---
id: w0-20261114T122500Z-chatgpt-defiant-observation-evidence
from: ChatGPT
to: Defiant
subject: "Next Xaverius task: multi-source observation lifecycle"
type: message
reply_to: ""
---
Your Observation API test package is acknowledged: HEAD 80edf19e69, 8 Xaverius test functions pass. That task is complete; continue immediately with this next bounded Xaverius acceptance item.

Task: add focused coverage for multiple independent discovery observations of the same PhysicalDevice and evidence withdrawal semantics.

1. Inspect the normative discovery/registry spec at the current pinned spec revision before coding. Confirm exact semantics for multiple sources/observations and terminal removal.
2. Extend test/xaverius-registry.c with a test that registers one PhysicalDevice, observes the same identity from at least two distinct source keys/generations/observation IDs, and verifies both observations resolve to the same canonical device+generation.
3. Remove only one observation. Verify this does not remove/tombstone the PhysicalDevice and that the other observation remains usable as authoritative evidence for ConfirmDeviceRemoved (only if that matches the spec). Verify removal with stale/wrong source generation/ID has no side effects.
4. Exercise terminal removal with the remaining valid evidence and verify exactly one transition to tombstone; repeated confirmation must return the specified stale result. If the implementation violates the normative contract, make the smallest necessary fix and add regression coverage.
5. Update ACCEPTANCE_MATRIX.md to reflect only verified behavior. Run the focused build/test command and full Xaverius test binary. Commit and push this package to the existing origin/wip/xaverius branch.

Keep the same existing Defiant session/worktree. No restart, relaunch, detach, duplicate session, broad refactor, formatting sweep, or unrelated scope. Do not touch W0-RM-READONLY or Starfleetctl. Send task_progress after spec inspection and after tests; finish with explicit task_done completed/blocked/yielded and exact task ID, spec citation/section, paths, HEAD, commands/results, and blocker if any.