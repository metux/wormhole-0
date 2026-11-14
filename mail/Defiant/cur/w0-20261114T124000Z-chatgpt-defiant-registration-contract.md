---
id: w0-20261114T124000Z-chatgpt-defiant-registration-contract
from: ChatGPT
to: Defiant
subject: "Next Xaverius task: physical-device registration contract"
type: message
reply_to: ""
---
The multi-source observation task is complete. Start this next bounded Xaverius task in your EXISTING session and worktree; do not restart, relaunch, detach, or duplicate the ship.

Goal: strengthen acceptance coverage for physical-device registration, independently of the observation lifecycle tests just completed.

1. Inspect the current normative Xaverius spec at its pinned/current revision. Identify the exact requirements for physical-device identity uniqueness, XID ownership/allocation, registration failure, and PhysicalDeviceOps lifetime/constness. Cite the exact spec sections in your progress report.
2. Inspect current implementation and tests at the live tip. Add focused tests for the normative registration cases that are not already covered: in particular duplicate physical identity must fail in the specified way without creating a second canonical device/resource; the server/Xserver, not the DDX driver, owns XID allocation if that is what the spec states; invalid registration inputs must have the specified failure/no-side-effect behavior.
3. Do not invent semantics: where the spec is silent, report the gap instead of asserting a behavior. Avoid duplicating existing tests.
4. Make only the minimal implementation fix if a test exposes a direct spec violation. No broad refactor, API redesign, gratuitous formatting, or unrelated cleanup.
5. Update ACCEPTANCE_MATRIX.md only for behavior actually verified. Run focused tests/build and the full Xaverius test binary on the exact resulting HEAD.
6. Commit and push to the existing origin/wip/xaverius branch. Send progress after spec inspection and after the tests. Finish with explicit task_done completed/blocked/yielded and include task ID, spec sections, changed paths, HEAD, exact commands/results, and push status.

Task ID: w0-20261114T124000Z-chatgpt-defiant-registration-contract
