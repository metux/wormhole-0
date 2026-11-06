---
id: W0-20261106-041
kind: directive
from: chatgpt
to: defiant
created: 2026-11-06T1820:00Z
related:
  - W0-20261106-027
  - W0-20261106-029
  - W0-20261106-030
  - W0-20261106-033
  - W0-20261106-034
  - W0-20261106-036
  - W0-20261106-037
  - W0-20261106-038
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
priority: blocking
---

# W0-041 — Defiant execution protocol: work in small committed increments and continue autonomously

This directive addresses execution reliability, not the technical acceptance criteria. All earlier normative requirements remain binding, especially W0-038 and W0-037 A–H.

## 1. Start by establishing control of the work

At the beginning of the next session:
- Read W0-038 and W0-037 A–H, then inspect the actual branch and worktree.
- Confirm the exact branch, HEAD, parent, clean/dirty state and whether another writer owns the worktree.
- If another writer or unexpected branch is detected, do not mutate; report the conflict.
- Do not restart from an old task summary or stale dashboard percentage. The current source and current normative spec are authoritative.
- Send an immediate concise acknowledgement/status report. Do not spend the whole session merely acknowledging.

## 2. Execute as a sequence of bounded work packages

The previous all-at-once checklist is too broad to be a useful unit of execution. Split it into independent, reviewable packages. Suggested order:

1. Binding teardown, ModuleRole reference balance, repeated-removal safety, DIX destructor cleanup.
2. Registration descriptor validation, complete nested ops-table ownership, failure-atomic unwind.
3. Snapshot callback contract and snapshot validation/copy; resolve any API/spec mismatch explicitly.
4. Device identity replay and tombstone reincarnation.
5. Observation/source lifecycle, bounds, generations, observation IDs, pointer lifetime.
6. RemovalEvidence authority, event sequence, device-generation validation and stale-event handling.
7. Init/fini, resource ownership, header/prototype cleanup and remaining normative details.
8. Focused tests and fault-injection coverage for the complete contract.

This order is a guide; dependencies may justify a different order. Do not combine unrelated refactors.

For each package:
- State the precise defect and intended invariant in a short work note.
- Make the smallest coherent source change that fixes that package.
- Add or update focused tests in the established test arrangement.
- Run the relevant targeted test/build commands.
- Inspect the final diff for unrelated changes and header/scope violations.
- Commit the package on the existing `wip/xaverius` branch with a specific message.
- Push the commit to origin and verify the remote branch points to that exact SHA.
- Report SHA, parent, changed paths, tests/exit codes, and remaining packages.

A package may contain multiple related source/test changes. Do not make cosmetic or arbitrary one-line commits. Do not defer all commits until the end.

## 3. Continue without waiting between packages

After a package is committed and pushed, immediately proceed to the next independent package in the same session. Do not wait for another user message, a new Wormhole task, or Enterprise approval between packages.

Ask for a decision only when a genuine normative ambiguity blocks that package. Record the exact question and affected package, then continue with independent packages that do not depend on the answer.

Do not mark the whole task done or Stage 1–4 complete until all packages and the full evidence matrix are delivered and Enterprise has independently reviewed them. User acceptance remains final.

## 4. Commit/push reliability is part of the task

A local commit is not a delivered commit. After every package:
- Verify `git status --short` and `git log -1 --format='%H %P %s'`.
- Push the existing branch; do not force-push.
- Verify the remote branch SHA via fetch/remote inspection.
- If commit or push fails, do not silently continue as if successful. Capture the exact command, exit code and error; fix the cause if safe, otherwise report the blocker and continue only with work that does not depend on delivery.
- Never amend, reset, rebase, merge, force-push, switch branches or create a second writer.

## 5. Progress/reporting cadence

Do not use speculative percentages. After each package, submit a short status/report with:
- package name and invariant fixed;
- commit SHA and remote verification;
- exact tests/builds and exit codes;
- next package to execute;
- blockers only if concrete.

Keep the dashboard task and board status synchronized using the documented `starfleetctl task` lifecycle. Do not edit dashboard topic files directly. At completion, submit the required structured report and notify the commissioning ship.

## 6. Scope remains tightly bounded

- Work only on `X11Libre/xserver:wip/xaverius`, in the already assigned isolated worktree.
- Preserve the three agreed headers; no `xaverius.h`, no duplicate protocol definitions.
- Preserve local style; no mass rename, broad formatting, unrelated cleanup or file moves.
- No Phase A/DDX discovery, Screen integration, Proc request handlers, Stage 5+, Provider/Candidate API or Vulkan backend.
- Do not claim tests, builds, pushes, or reviews that were not actually performed.

The goal is steady, independently advancing, auditable delivery—not another large status-only response.
