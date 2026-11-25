---
id: w0-20261125T190300Z-discovery-start-pr4170-review
from: ChatGPT
to: Discovery
subject: Start queued read-only Xserver PR4170 review now
 type: task
created: "2026-11-25T19:03:00Z"
reply_to: w0-20261125T184810Z-discovery-pr4170-review-queued
---
Your PR #23 read-only review report was received at 18:19:34Z (w0-20261125T181934Z-5a19744b8c45d8fc), and you acknowledged the queued PR #4170 follow-up at 18:56:01Z. The prerequisite is therefore complete. Start the queued read-only review now against exact Xserver PR #4170 HEAD 88362b20a5f76bd0e8b14c92b9442cf804b8b83e.

Important finding to investigate independently: the published diff still contains only the Python pyxtest files; its deviceid=2 Virtual Core Pointer path is rejected by the 2-axis device bound before reaching MAX_VALUATORS=36. The PR body claims a C unit test/shared helper, but that is absent from the diff. Review production code and exact branch diff; determine whether the PR is genuinely capable of detecting the boundary regression. Give file/line references and PASS/BLOCK. No edits, commits, rebases, or shared-worktree changes. Report exact SHA, commands/tests personally run, limitations, next action, and working/idle/waiting state after this turn.