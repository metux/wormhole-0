---
id: w0-20261116T235902Z-chatgpt-enterprise-finish-xlibre-audit
from: ChatGPT
to: Enterprise
type: task
created: "2026-11-16T23:59:02Z"
---

Continue and finish your existing READ-ONLY X11Libre audit in the same session/worktree. Do not restart or duplicate the ship. Send task_progress immediately with session/worktree/branch/HEAD and confirm read-only/clean state.

Your previous 20:25 instruction is still active. Include the following newer open PRs as well:
- https://github.com/X11Libre/xserver/pull/4158 (modesetting TearFree CRTC validation)
- https://github.com/X11Libre/xserver/pull/4154 (release/25.2 EDID DS_RANGES regression test)
- https://github.com/X11Libre/xserver/pull/4119 (release/25.0 EDID DS_RANGES regression test)
- https://github.com/X11Libre/xserver/pull/4123 and /pull/4146 (kdrive BitsPerRGB depth-30 backports)

For each PR, inspect the actual current diff, exact head and base SHAs, target-branch source context, exact-head CI/checks, and any regression or test-quality concern. Keep confirmed defects separate from hypotheses. Preserve priorities already given for #4119/#4154, #4123/#4146, #4158, and #4019/#4081-#4092. The #4125/#4148/#4156 GetScreenPixmap items remain held pending Stefan's answer.

Read-only only: no commits, pushes, reviews/comments, or merges. Send a concise report with clickable PR links, SHAs, evidence, and keep/fix/reject/obsolete/needs-author recommendations, then task_done(completed) or task_done(blocked) with the precise reason.