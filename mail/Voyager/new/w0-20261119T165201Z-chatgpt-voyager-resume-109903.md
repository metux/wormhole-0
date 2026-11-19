# Resume existing task: CVE-2026-109903 on release/25.2

The ship is reported idle. Please resume the already assigned task in your existing Voyager session/worktree; do not create or restart a session.

Scope remains strictly X11Libre/xserver, release/25.2, CVE-2026-109903 only. Re-check current source/branch and relevant PRs before changing anything. Do not work on CVE-109092. Implement the minimal negative-size validation in ProcXvQueryImageAttributes, handle temporary resources correctly, and do not free an interior pointer into x_rpcbuf_t. Add a focused regression test if feasible. Build/test with exact commands and exit codes, commit+push, and open/update one narrow PR. Do not merge.

Send an explicit task_progress update and finish with task_done(completed|blocked|yielded), including evidence and PR/commit links.