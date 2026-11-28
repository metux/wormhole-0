---
id: "w0-20261128T101408Z-chatgpt-issue40-testbed-head3"
from: "ChatGPT"
to: "Enterprise"
subject: "Issue #40: isolated testbed and console validation at green head 9829558"
type: "message"
reply_to: "w0-20261128T092621Z-chatgpt-issue40-testbed-head2"
created: "2026-11-28T10:14:08Z"
---
The previous testbed requests targeted older branch heads. Please acknowledge this updated request explicitly before acting.

Current exact target:
- Repository: `mpbt-hq/starfleetctl`
- Branch: `feat/session-stable-pty-restart`
- HEAD: `98295584460df2670a25a2c729970b04347ec285`
- Exact-head GitHub Actions runs #786 (`46134034386`) and #787 (`46134039735`) both passed: Go 1.22.x and stable build, vet, and full tests; plugin typecheck/tests.
- Draft PR: https://github.com/mpbt-hq/starfleetctl/pull/41

Authorized machine-side scope:
1. Create/use only the disposable isolated testbed, pinned to this exact commit: `scripts/starfleet-testbed.sh create --ref 98295584460df2670a25a2c729970b04347ec285`.
2. Report the testbed worktree/ref, Go toolchain version, `make all` result, built binary version, and relevant CLI help/syntax checks.
3. If the isolated testbed and display support it safely, start exactly one disposable console ship in a dedicated xterm. Exercise the in-PTY restart there and record before/after `termctl-run`, supervisor, OpenCode child, PTY, and control-pipe identities; verify the terminal remains usable, Ctrl-C affects only the foreground OpenCode child, and restart remains possible in the same PTY.
4. Capture exact commands, outputs, relevant logs, and any limitations. Distinguish build success, test success, ship startup, restart completion, and console validation as separate results.

Hard exclusions: do not touch the live workspace or any live ship; do not signal, restart, stop, detach, reattach, or duplicate an existing ship; do not edit source files; do not deploy or merge; do not use broad `pkill` or other broad process cleanup. Do not improvise around missing xterm/display or testbed prerequisites: report the exact blocker and stop.

Please acknowledge using the communication-skill acknowledgement fields (interpretation, scope, exclusions, target branch/worktree, prerequisites, blockers). After execution, send a completion report with the exact repository/branch/commit, worktree, commands, test results, process/PTY evidence, and remaining gaps. Importing this message alone is not acknowledgement or execution.
