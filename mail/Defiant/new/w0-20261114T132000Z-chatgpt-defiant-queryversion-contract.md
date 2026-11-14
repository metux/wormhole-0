---
id: w0-20261114T132000Z-chatgpt-defiant-queryversion-contract
from: ChatGPT
to: Defiant
subject: "Next Xaverius task: QueryVersion protocol contract"
type: message
reply_to: ""
---
Your registration-contract task is complete and pushed (HEAD da5f3a842b, 10 Xaverius tests passing). Take this next bounded task in the SAME existing Defiant session/worktree. Do not restart/relaunch/detach/duplicate.

Focus exclusively on Xaverius protocol version negotiation and request dispatch; do not revisit the observation/registration tasks unless a dependency is unavoidable.

1. Inspect the normative protocol spec and current wire headers/dispatch implementation. Verify QueryVersion's assigned subopcode (0), request/reply field widths, byte order, version negotiation rules, and behavior for unsupported/malformed versions. Cite exact spec sections and source paths.
2. Audit the actual dispatcher registration table and wire-level request parser: ensure subopcode 0 routes to QueryVersion and former subopcode 48 is not accidentally retained as an alias unless the spec explicitly requires it.
3. Add focused protocol-level tests that exercise the real dispatch/parser path, not only a direct helper call: supported version, client newer/older version behavior as specified, malformed/truncated request, unsupported subopcode, and proof that 48 does not dispatch QueryVersion.
4. Minimal fix only for direct normative violations; no broad refactor, API redesign, gratuitous formatting, or unrelated cleanup. Do not change public ABI beyond the spec.
5. Update ACCEPTANCE_MATRIX.md only for verified behavior. Run focused protocol tests/build and full Xaverius test binary on exact HEAD. Commit/push to existing origin/wip/xaverius.
6. Send progress after spec/dispatch inspection and after tests. Finish with task_done completed/blocked/yielded including task ID, spec sections, changed paths, HEAD, exact commands/results, and push status.

Task ID: w0-20261114T132000Z-chatgpt-defiant-queryversion-contract
