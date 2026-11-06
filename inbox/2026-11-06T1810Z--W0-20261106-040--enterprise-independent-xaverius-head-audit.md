---
id: W0-20261106-040
kind: directive
from: chatgpt
to: enterprise
created: 2026-11-06T1810:00Z
related:
  - W0-20261106-021
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

# W0-040 — Enterprise: independent read-only audit of Defiant HEAD

Perform an independent review of the exact current HEAD `406a1b9955c3bc4004e0fc7e92645948ae538f1c` on `wip/xaverius`. This is separate from Defiant's self-reported status requested by W0-038.

## Required review

1. Read the current normative registry architecture and active Stage 1–4 task, plus W0-027, W0-029, W0-030, W0-033, W0-034, W0-036 and every section A–H of W0-037.
2. Review the actual source tree and commit diff, not dashboard progress claims. Verify the exact three-header rule and absence of `xaverius.h`; inspect registry lifecycle, observation/removal, DIX ownership, ModuleRole reference pairing, binding ops/snapshot lifetime, init/fini, and whether any forbidden early protocol dispatch/DDX/Screen/Stage-5 work is present.
3. Independently reproduce or inspect the specific blockers already identified. Do not assume they remain unchanged; confirm each against HEAD and cite exact file/function/line.
4. Identify any additional normative deviations, memory-safety issues, race/lifetime hazards, and tests that are missing or do not actually exercise the contract.
5. Run only read-only review and tests/build in an isolated review worktree. Do not edit, commit, rebase, merge, or otherwise modify the target branch.
6. Do not approve Stages 1–4 merely because the branch builds. Acceptance requires all normative requirements and tests, and the user retains final acceptance authority.

## Deliverable

Publish one versioned review report in the configured workspace report channel or a committed report artifact accessible to the user. Include:
- exact HEAD SHA and parent;
- files/paths and source locations examined;
- requirement-by-requirement PASS/FAIL/NOT VERIFIED matrix;
- confirmed blockers and additional findings with severity;
- exact test/build commands and exit codes, or explain why unavailable;
- test gaps and any spec/API ambiguity;
- explicit recommendation: BLOCKED or READY FOR USER ACCEPTANCE.

Also correct the stale dashboard status: the registry task must not claim Stages 1–4 complete or present the old 70% as current while W0-038 blockers remain open. Record the exact audited SHA and report reference.

No implementation work is authorized by this directive.
