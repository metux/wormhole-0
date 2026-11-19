# Task: Independent audit of 2026 X.Org advisory list and CVE-2026-50263 backports

Priority: normal
Scope: read-only audit; do not modify files or code.

The user reports that Defiant and Voyager are idle; Enterprise is currently working. Take this independent verification task as Voyager. Do not start or restart Enterprise.

Please independently verify:
1. CVE-2026-50263 is the CreateSaverWindow use-after-free / information disclosure, and DRI2 DRIGetBuffers OOB write is CVE-2026-50264 (not another CVE-2026-50263).
2. The exact status on X11Libre/xserver master, release/25.0, release/25.1, and release/25.2: identify fix commit(s), PR(s), and whether the fix is actually present at current branch tips. Distinguish merged PR evidence from content-only checks.
3. Find the authoritative source repository/path for https://www.x.org/Development/Security/ and identify how the section "Earlier 2026 advisories" is currently structured.
4. Recommend a compact one-line-per-CVE format with CVE link and fixing PR link(s), and list any factual corrections required.

Known candidate PRs (verify independently, don't blindly trust this list):
- https://github.com/X11Libre/xserver/pull/2871 (master)
- https://github.com/X11Libre/xserver/pull/2875 (release/25.0)
- https://github.com/X11Libre/xserver/pull/2873 (release/25.1)

Do not edit or create PRs. Publish an evidence-based report in wormhole-0 outbox with exact refs/links and commit it. Do not claim delivery/acknowledgement merely because the message was committed.
