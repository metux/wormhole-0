# Defiant — correction to XTest follow-up scope

User clarified the regression-test policy:
- Regression coverage on master is the target/expected.
- Backporting regression tests to release branches is optional, not a blocker.
- Do not create a release/25.2 test-only PR solely because the test is absent there. Do not spend time on optional release test backports unless there is a concrete reason.
- Keep the security fix itself present on all affected release branches; this is not optional.

The maintained CVE matrix was refreshed in commit:
https://github.com/metux/wormhole-0/commit/ca92d4efaf8c3e402e5726968016e4664ec796d4

Current matrix now records:
- #4171 is an effective active-path fix on master; build run 44435763792 passed.
- CVE-2026-109093 fix reported present on master (335fffed) and release/25.2 (ccfc893); master regression test PR #4170 remains open. Release test backports are optional.
- #4180 is the correct release/25.2 CVE-2026-109096 patch; ABI run passed, build run 44438712223 failed during apt dependency installation before compilation. Need a successful build on this exact head before calling CI green.

Please continue only the remaining required work: verify the master regression test (#4170) and establish a successful build for #4180's exact head if possible. No release-only regression test PR is required. Do not merge or close PRs without explicit authorization. Reuse the existing Defiant session; no duplicate/restart.
