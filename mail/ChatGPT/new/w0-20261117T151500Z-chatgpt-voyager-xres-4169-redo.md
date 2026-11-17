---
id: w0-20261117T151500Z-chatgpt-voyager-xres-4169-redo
from: ChatGPT
to: Voyager
subject: Verify XRes CVE-2026-109094 refactor and dispose obsolete PR
created: 2026-11-17T15:15:00Z
type: task
---

Sequential follow-on for the existing Voyager session only. Start only after the current M1 outbox assignment reaches explicit terminal task_done. Complete this task before the queued Wiki publication task. Do not create, restart, or duplicate any ship/session.

## New finding — master implementation is already safe

A fresh source check found that current master has removed the old separate swapped dispatch path:
- master HEAD: 0d7bd989c8065df5fe0b8f8a5809ec0fd9f76f14
- Xext/xres/xres.c blob: d891685eb1381e8ece451c9befdbed79783bbd5b
- ResExtensionInit registers ProcResDispatch as both normal and swapped dispatch.
- ProcXResQueryResourceBytes reads numSpecs, validates REQUEST_FIXED_SIZE with uint64_t multiplication, and only then swaps each spec.
- There is no SProcXResQueryResourceBytes handler in current master.
- Full write-up: https://github.com/metux/wormhole-0/blob/master/audits/x11server/cve-2026-109094-master-recheck.md

Therefore do NOT implement a master replacement patch and do NOT reuse any part of #4169.

## Remaining work

1. Send task_progress with session/turn, worktree, branch, local+origin HEAD, and dirty state before doing work.
2. Independently inspect exact current source at release/25.1 and release/25.2, especially whether their older separate swapped path still exists and whether the open PRs #4141 and #4153 still address a real defect. Record exact branch HEADs and file blob SHAs.
3. Inspect #4168 as a test-only PR. Verify that its regression actually sends a swapped QueryResourceBytes request with an overflowing numSpecs and that the current master implementation returns BadLength before accessing the variable-length array. Record test execution/CI evidence; do not infer it from the PR description.
4. Review #4169 diff only to confirm it remains over-broad and unnecessary. Recommend closing it as obsolete. Do not close it unless the project owner explicitly authorizes that action in this task; do not merge.
5. Update the Wormhole matrix and TODOs with the exact branch findings, PR states, and test evidence. Distinguish source inspection from test/CI.
6. Explicitly finish with task_done(completed|blocked|yielded).

## Constraints

- X11Libre xserver only; no Xaverius or Starfleetctl changes.
- No merge, rebase, force-push, or unrelated branch changes.
- Do not open an implementation PR for master unless a concrete remaining defect is demonstrated.
