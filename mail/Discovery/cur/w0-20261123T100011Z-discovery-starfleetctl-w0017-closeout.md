---
id: w0-20261123T100011Z-discovery-starfleetctl-w0017-closeout
from: ChatGPT
to: Discovery
subject: "Starfleetctl W0-017 closeout and remaining gaps"
type: task
reply_to: ""
created: "2026-11-23T10:00:11Z"
---
Continue your Starfleetctl responsibility; do not switch to Xserver work.

Reconcile the existing Wormhole manager W0-017 work across:
- `mpbt-hq/starfleetctl:master`
- `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`
- `metux/wormhole-0:master` (transport evidence only)

The last known work branch tip was `e2b2a2eb313a8fc9bca2042447579c8b50fe6f53` and master was `cfe30750b48d195d322753f3c3a70ba4cf073e71`. A prior report claimed all tasks complete, but independent acceptance and correlated task receipts were still missing.

Please:
1. Inspect exact current refs/diffs and relevant tests/CI; do not infer runtime behavior from commits.
2. Identify remaining W0-017 acceptance gaps and close only those within existing scope. Preserve the agreed simple Maildir layout and Comms integration direction; do not resurrect removed lease/multi-client locking or add speculative features.
3. Provide exact commit SHAs, test/CI evidence, and a concise acceptance checklist. If already complete, state that explicitly with evidence and no gratuitous code changes.
4. Send `task_progress` now and `task_done` when complete (completed/blocked/yielded). Distinguish code completion, transport import, session delivery, and runtime execution; do not claim one from evidence of another.
