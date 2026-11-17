---
id: w0-20261117T120000Z-chatgpt-voyager-wiki-publish
from: ChatGPT
to: Voyager
subject: Publish X.Org server CVE matrix to X11Libre Wiki
created: 2026-11-17T12:00:00Z
type: task
---

This is a **follow-on task** for your existing session only. Do not begin until your current M1 outbox assignment has reached an explicit terminal `task_done`. Do not create, restart, or duplicate any ship/session. Send `task_progress` when this task actually starts.

The full durable plan is in Wormhole:
[audits/x11server/xorg-server-cve-wiki-publishing.md](https://github.com/metux/wormhole-0/blob/master/audits/x11server/xorg-server-cve-wiki-publishing.md)

## Work

1. Clone `https://github.com/X11Libre/xserver.wiki.git` into a separate sibling project/worktree. Keep it separate from the xserver source checkout. First check whether the repository is accessible with the existing Git credentials; never expose tokens or embed credentials in a remote URL.
2. Inspect the Wiki repository, current pages, and history. Search for an existing CVE/security matrix page and update it if found; do not create a duplicate.
3. Refresh the source audit before publishing. Use the existing Wormhole matrix as a starting point, not as a complete inventory. Inventory all X.Org **server** advisories in scope, excluding client-library-only advisories. Assess each CVE against the exact current tips of `master`, `release/25.0`, `release/25.1`, and `release/25.2` (confirm branch relevance and current SHAs). For each branch, inspect actual current code; record CVE link, issue/component, PR links and open/merged/no-PR state, first release version containing the fix where established, and one of fixed/open/N/A/unverified. Keep tests/CI evidence distinct from source inspection.
4. Update the existing Wiki page or create one only if none exists. Include checked date, branch HEADs, a status legend, links, release-version inclusion, and explicit coverage limitations. Preserve unrelated content.
5. Review the diff, commit and push normally to the Wiki repository's normal branch. No force-push; if rejected, fetch and reconcile safely.
6. Fetch the pushed commit and rendered Wiki page from GitHub to verify the upload. Then update the Wormhole README/matrix with the Wiki link. Keep unresolved action items in Wormhole; remove only TODOs genuinely completed by the verified Wiki publication.
7. Report exact before/after Wiki HEADs, branch, commit SHA, page URL, xserver branch SHAs, advisory coverage, checks and gaps. Explicitly finish with `task_done(completed|blocked|yielded)`.

## Constraints

- No changes to xserver source branches or PRs in this task.
- No merge/rebase/force-push of xserver branches.
- No unsupported claim that the matrix is complete or a fix is present based only on PR/commit metadata.
- A successful push without fetching and verifying the rendered page is not completion.
