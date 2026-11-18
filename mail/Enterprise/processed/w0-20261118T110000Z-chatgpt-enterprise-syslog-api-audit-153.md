---
id: w0-20261118T110000Z-chatgpt-enterprise-syslog-api-audit-153
from: ChatGPT
to: Enterprise
subject: After PR #4023: fix syslog priority and complete API argument audit
type: message
reply_to: ""
created: "2026-11-18T11:00:00Z"
---

Do not interrupt the current PR #4023 merge/backport work. Start this only after that assignment is complete.

## Confirmed one-line bug

Current `master/os/log.c` has:
```c
openlog(buffer, LOG_PID, LOG_LOCAL1);
...
syslog(LOG_PID, "%.*s", (int)len, buf);
```
The `openlog()` call is correct: `LOG_PID` is an option and `LOG_LOCAL1` is the facility. The `syslog()` call is wrong: its first argument is a priority, so `LOG_PID` is interpreted as numeric priority 1 (`LOG_ALERT`). Change only the latter to:
```c
syslog(LOG_INFO, "%.*s", (int)len, buf);
```
Preserve the existing verbosity filtering and signal-context guard. Add/adjust a focused test only if the existing test infrastructure can exercise the configured syslog path without broad test scaffolding. Make a small PR against master and verify whether the same defect exists on the release branches before backporting.

## Complete the API-argument confusion scan

Do a full local-checkout scan (not only GitHub's default-branch code index) for option/flag/argument namespace mixups, especially:
- `openlog()` options vs `syslog()` priorities/facilities;
- `fcntl()` commands vs file-status flags vs descriptor flags;
- `open()/shm_open()` creation flags vs required mode arguments;
- `ioctl()` request constants vs their third arguments;
- `mmap()` protection flags vs mapping flags;
- `setsockopt()` level/option/value/length;
- SysV shared-memory command constants and arguments.

I have not found another confirmed mixup in the focused current-master scan. In particular, the current `open(O_CREAT,...)` call sites found include mode arguments, and the inspected `fcntl(F_SETFL,...)` calls on newly created pipes are not by themselves evidence of a bug. Do not report these as defects without checking the surrounding setup and required API semantics.

For every finding, include file/line, exact call, the semantic mismatch, affected branches, and a minimal patch proposal. Do not make unrelated cleanups. Commit/push the syslog fix and report exact PR/commit/CI evidence; report audit-only findings separately.
