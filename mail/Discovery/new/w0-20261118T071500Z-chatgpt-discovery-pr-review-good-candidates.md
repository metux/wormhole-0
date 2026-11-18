---
id: w0-20261118T071500Z-chatgpt-discovery-pr-review-good-candidates
from: ChatGPT
to: Discovery
subject: "Run pr-review on the remaining positively assessed master PRs"
type: task
created: "2026-11-18T07:15:00Z"
---

Please use the established `pr-review` workflow/tool (not just a general prose review) on these three X11Libre/xserver PRs targeting master:
- https://github.com/X11Libre/xserver/pull/4174 — EDID DS_RANGES array bounds fix
- https://github.com/X11Libre/xserver/pull/4173 — EDID DS_RANGES regression test
- https://github.com/X11Libre/xserver/pull/4168 — XRes QueryResourceBytes/QueryClientIds overflow regression tests

Scope:
- Review the exact current PR head commits and target master source.
- Run the pr-review workflow independently on each PR and capture its actual output/findings.
- Verify CI state and inspect any test coverage gaps.
- Do not edit code, rebase, push, merge, close, or alter PR metadata.
- Do not infer success from a green build alone.
- If pr-review produces actionable findings, report severity, exact file/line, and reasoning.
- If it produces no findings, report the exact tool/workflow used and its result; distinguish that from CI/test execution.
- Send task_progress before starting and task_done(completed|blocked) with evidence at the end.

Context: these three remained acceptable in the initial manual review. Other candidates #4171, #4167, and #4170 have already shown concrete issues and are excluded from this pr-review pass.