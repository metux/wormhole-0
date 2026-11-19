# Task: Normalize the "Earlier 2026 advisories" section

Priority: normal
Scope: documentation only. Do not touch Xserver implementation code.

The user reports that Defiant and Voyager are idle; Enterprise is currently working. Take this task as Defiant. Do not start or restart Enterprise.

Goal:
Find the source repository/file that generates or contains the X.Org Security page section titled "Earlier 2026 advisories" (https://www.x.org/Development/Security/). Update that section so its presentation matches the newer advisory groups above it:
- one line/bullet per CVE (no multiple CVEs on one line);
- link each CVE ID to the relevant advisory/CVE record where an authoritative link exists;
- link the relevant fixing PR(s) for X11Libre/xserver (use actual PR URLs; do not invent);
- preserve dates, affected components, and fixed-version information;
- keep ordering consistent with the surrounding page;
- correct the apparent duplicate/misassigned CVE ID in the June 2, 2026 group: CreateSaverWindow UAF is CVE-2026-50263; DRI2 DRIGetBuffers OOB write is CVE-2026-50264. Verify this against authoritative sources before editing.

Known xserver PRs for CVE-2026-50263:
- master: https://github.com/X11Libre/xserver/pull/2871
- release/25.0: https://github.com/X11Libre/xserver/pull/2875
- release/25.1: https://github.com/X11Libre/xserver/pull/2873
The same fix is already present on release/25.2; verify its history/PR before mentioning it.

Constraints:
- First identify and report the exact source repo, path, branch, and current format. Do not assume the website source repository.
- Make only the requested advisory-list formatting/metadata corrections; no broad page rewrite.
- Preserve all other content.
- If source is not writable/identifiable, report findings and stop rather than editing a guessed repo.
- Run the relevant docs/build or link checks if available.
- Commit and push a focused change; open a PR if that is the project's workflow.
- Report exact diff, commit, PR, checks, and any unresolved links.
