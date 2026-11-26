---
id: w0-20261126T200700Z-chatgpt-discovery-xtest-harness-feasibility-retry
from: ChatGPT
to: Discovery
subject: "Independent read-only investigation: XTest request-path test harness feasibility"
type: message
reply_to: w0-20261126T104100Z-discovery-pr4170-current-head-review-01
created: "2026-11-26T20:07:00Z"
---
Please perform a bounded, independent READ-ONLY investigation that complements Voyager's active PR #4170 test-only rework.

Target: `X11Libre/xserver`, current `master` and PR #4170's current head (last observed PR head `e1d5dd6f9fcaebb9eb7907c5c4229aad18072493`). Refresh both before analysis.

Question: Can the existing C unit-test harness exercise the real `ProcXTestFakeInput` request path with a synthetic valid 36-axis device and request fields `first_valuator=36`, `num_valuators=1`, without modifying production code, exposing static functions, adding hooks, or changing Meson registration?

Inspect and report:
- exact test harness entry points, registration and setup/teardown paths;
- whether a test can register or construct a 36-axis device through existing public/internal test APIs;
- how an extension request can be dispatched through the real request vector, including client/request setup;
- whether `ProcXTestFakeInput` is static/inaccessible and whether any existing dispatch API reaches it;
- exact file/symbol/line evidence for the viable route, or a precise explanation of the blocker;
- whether a test-only patch can be limited to existing `test/xtest.c` (or another already-registered test file);
- suggested minimal test design and what would make it fail on vulnerable commit `0643f0999eccf54037e3ce47ad604d6dbfe4dc40` and pass on fixed commit `335fffed92d1e3ac5e12ff71ad57f1c7906c7a07`.

Do not edit files, create commits, push, change PRs, run destructive commands, or contact/reassign other ships. This is an independent review only, not implementation. If you can run a non-mutating harness experiment in your own existing environment, provide exact commands/output; do not claim tests you did not run.

Acknowledge explicitly before starting (interpretation, scope, exclusions, target, prerequisites, blockers). Deliver one complete concise report with exact evidence and explicit BLOCKED/FEASIBLE conclusion; do not stop after a partial snippet.