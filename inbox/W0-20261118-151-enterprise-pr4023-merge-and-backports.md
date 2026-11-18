# Enterprise task: merge PR #4023 and prepare backports

**Priority:** parallel task; do not interrupt the existing coordination work.

## Scope

Repository: https://github.com/X11Libre/xserver  
PR: https://github.com/X11Libre/xserver/pull/4023  
Target: `master`  
Reviewed head: `472b5f6a7ef1ce24348a8599f55f5dae5eb94941`

The PR fixes the syslog priority passed by `LogSyslogWrite()`: use `LOG_INFO`, not `LOG_PID` (which is an `openlog()` option and numerically maps to ALERT).

## Required sequence

1. Re-fetch PR metadata and verify the head SHA is still exactly the reviewed SHA above.
2. Verify that **all required CI/checks are completed and green on the current PR head**. The workflow run currently observed for this SHA is Build X servers, run 44289930111, completed/success. Do not treat that single observed run as proof that no other required checks exist; inspect the PR's full current check status.
3. If and only if the head is unchanged and every required check is green, merge PR #4023 into `master`. If not, report the exact blocker and wait; do not bypass checks.
4. After the merge, create separate backport PRs for:
   - `release/25.0`
   - `release/25.1`
   - `release/25.2`
5. Prefer a clean cherry-pick of the minimal one-line fix. Resolve any branch-specific conflict carefully; do not introduce unrelated changes.
6. Run/verify each backport PR's required CI. Do not merge a backport while checks are pending or failing.
7. Report all PR URLs, source and resulting commit SHAs, CI run/check outcomes, and any conflicts/blockers. Verify the fix is present in all four target branches after merges.

Keep this task separate from other work. Do not claim completion without GitHub evidence and actual merged commits.
