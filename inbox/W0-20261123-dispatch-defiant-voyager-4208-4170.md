---
id: W0-20261123-dispatch-defiant-voyager-4208-4170
kind: task
from: ChatGPT
to: Enterprise
created: "2026-11-23"
priority: high
task_ref: xserver-pr-cve-dispatch
---

# Dispatch two idle ships to concrete Xserver work

The owner explicitly authorizes a temporary reassignment of Defiant from its Xaverius-only work so both idle ships can advance open Xserver PRs/CVE work. Do not restart, relaunch, duplicate, or replace either existing ship session.

## Defiant — CVE-2026-50264, release/25.2 (#4208)

Send this task to Defiant through the established Comms API and ask for an immediate task_progress receipt.

- Work only on X11Libre/xserver PR #4208, base release/25.2, head branch backport/cve-2026-50264-252, current head 96ddd56337b77cc9dcf120a0818a4da4f82d3174.
- Independently verify the off-by-one heap-overflow fix against master PR #3266 and sibling backports #4200 (25.1) and #4202 (25.0). Confirm the allocation bound and synthesized-front-buffer path from source, not only the PR description.
- Verify Signed-off-by and exact-head CI. At dispatch time the head commit has Signed-off-by and the ABI check and Build X servers runs both report success.
- If any correctness or branch issue remains, make only the minimal scoped correction on the existing PR branch, push it, and report the new exact SHA and evidence. Do not merge.
- Report a clear content verdict: PASS or BLOCK, with source reasoning and test/CI evidence.

This is an explicit temporary owner-approved exception to Defiant's previous Xaverius-only assignment. Do not touch wip/xaverius as part of this task; resume its existing assignment after this bounded task is completed or blocked.

## Voyager — finish existing CVE backports, then repair #4170

Send this task to Voyager through the established Comms API and ask for an immediate task_progress receipt.

1. First inspect the existing Voyager session/task state for the already-assigned master #3266 CVE-2026-50264 backports to release/25.1 and release/25.0 (expected PRs #4200 and #4202). Do not duplicate work or interrupt an actually active turn. Continue the existing assignment and finish both isolated backports with exact PR URLs, SHAs, and verification evidence; do not merge.
2. Once those backports are completed or explicitly blocked/yielded, take PR #4170, "test: cover XTestFakeInput valuator stack boundary". Its latest reported head is fef654bb0c797cf40c2f08e1eeeb5e6828b2a8f8; Build X servers run #21413 failed with five build failures. Diagnose the exact failing jobs, fix only the relevant test/build issue, and rerun CI on the exact new head.
3. Recheck the test's substantive validity: demonstrate that it reaches the vulnerable equality boundary firstValuator == numAxes == MAX_VALUATORS, fails on the vulnerable parent, and passes with the fix. Do not accept a test that merely gets BadValue before the out-of-bounds write.
4. Do not change CVE status based on this test-only PR, do not merge, and do not broaden into unrelated cleanup.

## Coordination / reporting requirements

- Use real Comms sends to the existing Defiant and Voyager identities. Do not infer delivery from this GitHub task file.
- Return correlated Comms message IDs and delivery/ack status for both ships. Require each ship to send task_progress, then explicit task_done(completed|blocked|yielded) when stopping.
- If Voyager's existing #3266 assignment is genuinely active, leave it running and queue #4170 behind it; report the exact state rather than creating duplicate work.
- Keep Defiant and Voyager changes isolated. Do not claim either ship has started until a Comms receipt and ship response confirm it.
- Publish a concise durable status report with exact sessions, worktrees, branches, HEADs, dirty/unpushed state, task status, and evidence. Enterprise is coordination-only; technical PASS/BLOCK review remains with the owner.
