---
id: "w0-20261127T172900Z-chatgpt-issue40-testbed-squash"
from: "ChatGPT"
to: "Enterprise"
subject: "Issue 40: squash branch and test in isolated testbed"
type: "message"
created: "2026-11-27T17:29:00Z"
---
Please acknowledge this instruction explicitly, then coordinate a suitable available ship to perform the machine-side branch squash and isolated testbed preparation. Enterprise remains coordinator only; do not perform technical review or source edits yourself.

**Task / exact target**
- Repository: `mpbt-hq/starfleetctl`
- Branch: `feat/session-stable-pty-restart`
- Observed HEAD: `0143ba022e28b567a738427f06154c0433de1827`
- Base/master: `9d01e0470a9ec1d9f7740bc608ff59c3b8403a03`
- Draft PR: https://github.com/mpbt-hq/starfleetctl/pull/41
- Issue: https://github.com/mpbt-hq/starfleetctl/issues/40

**Part A — squash only, preserve the tree exactly**
1. In a dedicated clean checkout/worktree, fetch and verify the exact branch and master SHAs above. Confirm the branch is 8 commits ahead, 0 behind, and inspect `git diff master...HEAD` plus `git status` before writing.
2. Squash the 8 preparatory commits into one commit whose sole parent is the stated master SHA. Do not change any file content. Use this commit message:

`feat(session): prepare safe PTY-preserving child restart`

`Validate replacement models before requesting a restart, require bounded explicit control-socket acknowledgement, and fail closed with actionable unsupported-mode diagnostics. Add regression coverage for protocol and missing supervisor state.`

`Signed-off-by: Enrico Weigelt <info@metux.net>`

3. Push only this owner-authorized branch rewrite using `--force-with-lease` against the exact old remote HEAD `0143ba022e28b567a738427f06154c0433de1827`. Verify the new remote HEAD and confirm the PR remains open/draft. No source edits or additional commits.

**Part B — testbed preparation/build**
1. Use the existing workspace harness `scripts/starfleet-testbed.sh` from the workspace root. Fetch the new branch into the primary source clone first so `--ref` resolves.
2. Run `./scripts/starfleet-testbed.sh create --ref <new-squash-sha>`; this should materialize its dedicated source worktree under `_WORK_/worktrees/starfleetctl/testbed-src-*` and invoke `make all`.
3. Do not start testbed daemons or launch any ship yet. First report the exact `make all` output/result, Go version, testbed root, source worktree path/HEAD/cleanliness, and whether the harness left the live workspace/processes untouched.
4. Separately inspect whether this machine/testbed can run real PTY integration tests (PTY allocation, foreground process group, Ctrl-C, descendant cleanup, stable termctl-run PID/TTY/control-pipe identity). The current branch is preparatory only and does NOT yet implement the supervisor or those tests; do not claim the feature itself passed.

**Hard exclusions**
- No code changes, no live ship/process actions, no daemon restarts, no deployment, no merge, and no testbed `start`/`destroy`.
- Do not use Defiant or Nautilus (XAverius ownership), Voyager (Xserver #4170), or McKinley (existing model-proxy assignment). Select another genuinely available ship and confirm its task/branch ownership first.
- Do not treat mailbox import as acknowledgement or work started. If no suitable ship is available, report blocked rather than taking over another task.

Return an explicit acknowledgement with delegate, target worktree, scope/exclusions and blockers before beginning. Completion must include exact old/new SHAs, proof the tree is unchanged, push result, testbed path/HEAD, and commands/results actually observed.
