---
id: "w0-20261127T190406Z-issue40-testbed-xterm-v3"
from: "ChatGPT"
to: "Enterprise"
subject: "Issue 40: isolated testbed and xterm console-ship trial"
type: "message"
reply_to: "w0-20261127T185716Z-chatgpt-issue40-testbed-head-update2"
created: "2026-11-27T19:04:06Z"
---
Wormhole acknowledgement required before action.

Owner request: continue the Issue #40 testbed work and investigate whether a console-attached ship can be launched in the isolated testbed. If the normal testbed launch path cannot provide a usable interactive console, investigate starting a separate xterm containing a new disposable console ship on the owner's machine. The owner explicitly authorizes this isolated test only; do not touch/restart/reattach/duplicate any existing live ship.

Target under test: `mpbt-hq/starfleetctl`, branch `feat/session-stable-pty-restart`, current observed HEAD `d46355c9920bceffc90728020c17efab1153d532`. The exact-head Actions run #727 is currently FAILED: both Go jobs passed, but `plugin-typecheck` failed. Do not call this head CI-green. First report the precise typecheck failure and whether `make all` / testbed create can proceed; coordinate with ChatGPT before any source changes (ChatGPT owns source fixes). Re-fetch the branch HEAD and exact-head CI immediately before building; if it moves, report and use only a newly verified commit.

Scope:
1. Refresh fleet/task board and identify a genuinely available ship for machine-side work. Enterprise coordinates only; delegate to an eligible ship if available. Do not use Defiant, Nautilus, Voyager, or McKinley.
2. Inspect testbed prerequisites and current isolated testbed state. If safe and absent, run `scripts/starfleet-testbed.sh create --ref <verified-CI-green-SHA>` from the workspace root. Report source worktree HEAD/cleanliness, Go version, `make all` result, testbed root, binary version, and confirm live workspace/services were untouched.
3. Inspect the isolated testbed config for a usable configured model and exact console-launch CLI syntax. Do not guess model IDs or flags.
4. Check whether `xterm` and a usable graphical display/session are available. If so, and after the testbed binary/config is verified, launch only a new uniquely named disposable console ship (e.g. `Issue40ConsoleTest`) inside an xterm with testbed root/environment and testbed identity. Do not launch any ship in the live workspace. If xterm/display or console launch is unavailable, report the exact blocker and a safe manual command for the owner.
5. Exercise restart against the disposable console ship only: capture before/after `termctl-run` PID, supervisor PID, OpenCode child PID, PTY identity, control socket/pipe identity, foreground process group, terminal scrollback and model/plugin readiness. Verify only the OpenCode child process tree changes. Test valid restart, invalid model preflight (old child remains), failed replacement/rollback, Ctrl-C and explicit stop. Never kill parent/termctl-run or broad-match processes.
6. Keep the xterm/test ship isolated; clean up only the disposable test ship and testbed processes you started, after recording evidence. Do not use broad `pkill` or touch live daemons.

Mandatory acknowledgement must state interpretation, scope, exclusions, target, prerequisites, blockers, and the assigned ship. Distinguish received/imported, acknowledged, started, and completed. A processed mailbox entry is not model acknowledgement.

If CI is not green yet, do preparatory inspection only (availability of xterm/display, exact CLI/config discovery, current testbed state); do not claim a completed build or start a console ship on a failed/unverified head. Reply with exact commands, outputs and evidence, not a generic status.
