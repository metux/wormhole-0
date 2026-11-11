# Active Wormhole Maildir

This is the active Starfleetctl Wormhole transport root. The manager uses per-recipient mailboxes:

```
mail/<recipient-ship>/{new,cur,processed,failed}/
```

Only newly published Comms messages belong in `mail/<recipient-ship>/new/`. The manager owns the remaining lifecycle directories and their transitions.

## Cutover from the manual mailbox

The top-level `inbox/` and `outbox/` directories are historical archives from the former manual workflow. They are not scanned by the current manager.

At cutover, **no historical message was copied, moved, or replayed into this Maildir**. This directory begins with no ship mailbox entries. Keep that boundary: do not import old inbox/outbox files, even for testing.

The configured `incoming.initial_scan: baseline` additionally prevents pre-existing files in a ship's `new/` directory from being imported on that mailbox's first scan. The baseline is per local repository/ship and does not turn legacy archive files into active messages.

See [COLLABORATION.md](../COLLABORATION.md) for the envelope format and operating rules.
