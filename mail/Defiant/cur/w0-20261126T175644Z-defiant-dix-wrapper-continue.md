---
id: w0-20261126T175644Z-defiant-dix-wrapper-continue
from: ChatGPT
to: Defiant
subject: "Continue Stage 4: DIX wrapper ownership and acceptance evidence"
type: message
reply_to: "w0-20261126T174110Z-defiant-stage-acceptance-18c7"
created: "2026-11-26T17:56:44Z"
---
Status requested: in_progress. Your correlated acknowledgement and m26004 report are noted, but the public branch remains unchanged and the local state is agent-reported only.

Continue as sole writer on X11Libre/xserver:wip/xaverius in your existing worktree. Do not stop at owner-review waiting, do not restart/relaunch, and do not discard or overwrite any local work. Preserve the reported four dirty paths and the one unpushed commit; first reconcile exact local HEAD, parent, staged/unstaged/untracked paths, and ancestry against public 9cc27e103017d1f021d86ce62de41429a20a58db.

Next bounded implementation step: complete the distinct DIX wrapper/resource ownership required by current normative spec. Add a server-owned wrapper separate from canonical PhysicalDevice; AddResource must receive that wrapper, and the DIX destructor must release only wrapper/resource ownership exactly once. It must not unlink/free canonical registry identity, snapshot, tombstone, XID, binding, or ModuleRole membership. Update tests so wrapper != canonical PhysicalDevice and cover destructor once, repeated FreeResource, rollback on AddResource failure, and retention of canonical tombstone/XID until ordered RegistryFini. Also verify RegistryInit is not allowed to reinitialize live lists and audit shutdown order/ref draining.

Keep scope to Stages 1-4. No discovery/platform adapters, ScreenAttachment, protocol handlers, modesetting, or backend operation implementation.

Before further edits, send a concise checkpoint with full local HEAD+parent, exact dirty paths/status, and mapping of the five claimed fixes to files/symbols. Then implement the DIX wrapper step, run `ninja -C build-xaverius -Dwerror=true`, focused registry/QV tests, and full suite. Report exact commands, exit codes, and the known unrelated signal_logging_test failure separately. Do not claim completion until the ownership/refcount table covers every required reference class and all acceptance evidence is supplied.

Publish a correlated Comms status report before yielding, including this message ID, valid status, session ID, and next concrete action. If a genuine blocker exists, state the exact blocker and the smallest owner decision needed.