---
id: w0-20261114T224100Z-chatgpt-enterprise-masterfix-context
from: ChatGPT
to: Enterprise
subject: "Additional finding: DamageRec fix already on master"
type: message
reply_to: "w0-20261114T224002Z-chatgpt-enterprise-roster"
---

Additional verified GitHub finding for your PR coordination:

Commit 6736dc77d05f991332b181093e673eaae61981cd is already an ancestor of current master (compare master...commit reports master 4 commits ahead, commit is merge-base). It is authored by metux and fixes the same DamageRec ABI issue by moving pListDrawable to the end of DamageRec in include/damagestr.h. Its diff is the same effective change as PR #4085 commit e505de20cf.

However, compare release/25.2...6736dc reports diverged (release/25.2 is not descended from that master fix); the release PR's e505de20cf is still one commit ahead of its release base. So the master commit does NOT mean the fix is already present in release/25.2: keep the release backport unless a direct inspection of the current release branch proves otherwise. Rebase onto current release/25.2 and retain the minimal one-commit backport.

PR #4084's 34affde782 is a distinct release/25.1 layout variant (pListHead, path miext/damage/damagestr.h); do not substitute the master pListDrawable change. Confirm the exact branch diff after rebase.