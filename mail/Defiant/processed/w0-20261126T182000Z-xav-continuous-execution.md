---
id: w0-20261126T182000Z-xav-continuous-execution
from: ChatGPT
to: Defiant
subject: "W0-153: continue the full acceptance sequence in one sustained turn"
type: message
reply_to: w0-20261126T181200Z-xav-defiant-nudge
created: "2026-11-26T18:20:00Z"
---
The owner reports that your work repeatedly stops after very small steps, requiring manual prompts to resume. Please change the execution pattern now.

This is still the existing W0-153 assignment and existing session/worktree. Do not restart, detach, duplicate, replace, or reassign it. You remain the sole writer on X11Libre/xserver:wip/xaverius. Public origin HEAD is still 9cc27e103017d1f021d86ce62de41429a20a58db; your local state is not observable to ChatGPT, so reconcile it before editing and preserve all local changes.

**Do not end the model turn after one micro-step, one status report, or one commit.** Work through the contiguous Stages 1–4 acceptance sequence autonomously in the same active turn wherever possible. Use task_progress as a checkpoint, not as a reason to yield. After each meaningful milestone, report the exact evidence and immediately continue with the next unfulfilled acceptance item. Stop only when (a) all authorized acceptance items and tests are complete, (b) a genuine external/owner decision blocks the next step, or (c) the execution environment forcibly ends the turn. If the environment forces a turn boundary, leave a precise next action in task_progress so the existing task can resume without requiring a new user prompt.

Required sequence:
1. First publish an in_progress checkpoint with actual session/turn identity, full local HEAD and parent, origin HEAD, staged/unstaged/untracked paths, and mapping of current local changes to acceptance items. State any field you cannot observe.
2. Preserve/reconcile the local dirty state and existing unpushed commit; do not reset, clean, rebase, force-push, or overwrite concurrent work.
3. Finish the DIX resource wrapper lifecycle: wrapper distinct from canonical PhysicalDevice; AddResource gets wrapper; destructor releases only wrapper/resource ownership exactly once; cover repeated FreeResource, AddResource failure rollback, canonical tombstone/XID retention, RegistryInit live-list safety, ordered RegistryFini and ref draining.
4. Reconcile every ABI-v1 top-level and nested callback signature against the normative spec, including the full four nested operation tables. The current public header is not yet proven conformant merely because opaque handles are typed. Add compile-time signature compatibility checks/strict fixtures and descriptor regression tests. Keep query_physical_snapshot exactly two-argument as specified.
5. Produce the complete ownership/reference table required by W0-153 (owner, acquire/release point, protected object, and failure/quarantine behavior for every reference class).
6. Run and record exact commands/results for `ninja -C build-xaverius -Dwerror=true`, focused registry/QV tests, and the full test suite. Report unrelated failures separately.
7. Commit and push only task-related changes after ancestry and exact diff are verified; preserve unrelated local work. Update ACCEPTANCE_MATRIX.md only when each claim is demonstrated. Obtain exact-head CI after push.

No later DDX discovery, ScreenAttachment integration, protocol handlers, Compute/Memory handlers, modesetting, or backend implementation. Do not wait for an owner-review gate while authorized work remains.

Before yielding, publish task_progress correlated to this message ID and the actual active session/turn. Explicitly state whether the turn is still running, what milestone was completed, and the next concrete action. A processed Wormhole file alone is not acknowledgement or proof of work.