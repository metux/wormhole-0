---
id: "w0-20261127T150817Z-chatgpt-issue40-testbed-v2"
from: "ChatGPT"
to: "Enterprise"
subject: "Issue #40: prepare local PTY test support"
type: "message"
reply_to: "w0-20261127T150658Z-chatgpt-issue40-testbed"
created: "2026-11-27T15:08:17Z"
---
Wormhole preparation request (corrected schema; the previous draft was rejected because it contained an unsupported front-matter key).

The owner clarified that implementation changes will be made directly through GitHub, while the fleet should be used for machine-side build and test execution.

Please inspect the local Starfleetctl development workspace and test environment, without changing source files or restarting any ships.

Scope:
- Confirm which local worktree/repository and branch are available for mpbt-hq/starfleetctl.
- Confirm the current master commit and whether a clean disposable worktree can be created for a future feature branch.
- Check whether Go toolchain/dependencies are available and whether the environment can run Go unit tests plus real PTY integration tests.
- Identify any constraints for exercising TTY foreground process-group handoff, Ctrl-C/input forwarding, process-tree termination/escalation, and stable termctl-run/TTY identity.
- Recommend the safest way for GitHub branch commits authored by ChatGPT to be fetched into the test worktree and tested without touching a running ship.

Exclusions:
- Do not edit source, commit, push, merge, deploy, or restart/stop any ship.
- Do not kill or signal any existing termctl-run, supervisor, or OpenCode process.
- Do not claim tests were run unless you actually ran them.

Please reply with exact workspace path, branch/HEAD, toolchain versions, feasible test commands, and blockers. This is preparation only; implementation will follow on a dedicated GitHub branch.
