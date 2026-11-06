# Draft: per-ship Wormhole message spools

Status: **DRAFT — owner-approved direction for the message-storage model; detailed transport and implementation contracts still require review.**

This draft records the owner's decision made on 2026-11-07. It is a design direction, not authorization to replace the current Comms or Wormhole implementation in one broad change.

## 1. Core model

Use one Maildir-like message spool per recipient ship. A sender writes a message **directly into the target ship's spool**. There is no separate sender outbox and no additional global/unsorted queue.

Example repository layout:

```text
spool/
  defiant/
    new/
    cur/
  enterprise/
    new/
    cur/
  laforge/
    new/
    cur/
```

- `spool/<recipient>/new/`: immutable messages awaiting pickup/processing by that recipient.
- `spool/<recipient>/cur/`: messages that recipient has explicitly picked up/read. This is a recipient-local state transition.
- No `outbox/`, no second `unsorted/` directory, and no global inbox shared by all ships.
- A local `tmp` area may be used transiently for atomic file creation, but it is not a durable protocol state or required repository directory.

The sender is the writer of the destination spool entry; the recipient is the owner of its `new -> cur` transition. Git commits record both operations.

## 2. Message creation and addressing

- Each logical message gets a cryptographically strong random Message-ID before its first write. Use a format with enough entropy for global uniqueness (for example, 128 random bits or more); do not use timestamps or process-local counters as the uniqueness source.
- The sender writes directly to each addressed recipient's `spool/<recipient>/new/<message-id>`.
- The message body and immutable headers never change after publication.
- Retries reuse the exact same Message-ID and exact same bytes. A retry must not create a second logical message.
- A duplicate destination path is handled idempotently only if the existing bytes match. If the same ID exists with different content, fail closed and surface corruption/collision; never overwrite it.
- For fanout, preserve one logical Message-ID across recipient copies, while each recipient has its own path and independent read/processing state. If later requirements demand per-recipient delivery IDs, model those explicitly without changing the logical Message-ID.
- Validate sender, recipient ship IDs, path components, message size, header syntax, and allowed content types before writing. Never derive filesystem paths from untrusted free-form display names.

## 3. Message envelope

Use a simple, versioned, UTF-8 text envelope inspired by RFC 5322/Maildir rather than implementing all of MIME prematurely. At minimum include:

- Message-ID
- Date (UTC)
- From (stable ship/endpoint ID)
- To (one or more stable recipient IDs)
- Subject
- Content-Type and envelope version
- In-Reply-To / References when threaded
- Correlation-ID and Task-ID when applicable
- Expiry only where the message type defines meaningful expiry

Header parsing must be strict and bounded. Define canonical serialization so identical logical messages produce identical bytes. Attachments/MIME can be a later, separately specified extension. Do not claim RFC 5322 compliance unless the parser/serializer actually implements its requirements.

## 4. State transitions and semantics

The recipient performs an atomic local rename:

```text
spool/<ship>/new/<message-id>
    -> spool/<ship>/cur/<message-id>
```

- The rename is atomic on the same filesystem.
- A recipient must commit/push the transition so it survives another checkout or process.
- `cur` means picked up/read, **not** successfully processed, completed, or acknowledged.
- Processing outcome and explicit acknowledgement are separate message semantics. A successful `new -> cur` move must never silently imply task completion.
- If the recipient crashes after moving but before processing, recovery must be able to resume from `cur` using a durable processing journal/state, without making the message reappear as new or executing non-idempotent side effects twice.
- Define retention/archival separately; do not delete messages merely because they moved to `cur`.

## 5. Git transport and concurrent writers

Git is the durable replication/transport mechanism for the shared spool repository.

A sync/write cycle must:
1. Acquire a process-safe lock for the repository checkout (and use isolated checkouts or equivalent protection where more than one worker can operate concurrently).
2. Fetch the configured remote and integrate the latest remote branch.
3. Create the message via same-filesystem temporary file plus atomic rename into the recipient's `new/`, or perform the recipient's atomic `new -> cur` move.
4. Commit only the intended changes.
5. Push normally.
6. On non-fast-forward rejection, fetch again, rebase/merge according to the defined policy, and retry with a bounded retry count and backoff.
7. On conflict, preserve local work and report a recoverable conflict; never force-push, reset away unpushed messages, or silently discard a state transition.

Messages are immutable, and distinct random filenames make concurrent message publication naturally mergeable. The same message path or the same recipient state transition can still conflict and therefore needs explicit idempotency/conflict handling.

The implementation must not assume that a process-local mutex protects multiple starfleetctl processes. Locking, checkout ownership, crash recovery, and stale local worktree handling must be designed and tested.

## 6. Delivery and acknowledgement vocabulary

Keep these states distinct:

- **Published:** sender's commit containing the recipient spool entry was pushed successfully.
- **Replicated:** recipient has fetched a repository revision containing the entry.
- **Read/Picked up:** recipient moved the entry from `new` to `cur` and committed that transition.
- **Processed:** recipient completed the message's defined work and persisted the outcome.
- **Acknowledged:** sender received an explicit correlated acknowledgement, if the message type requires one.

A successful Git push is not proof that the recipient fetched, read, processed, or acknowledged the message.

## 7. Explicit delivery confirmation (required when requested)

A sender must not infer delivery from a successful Git commit/push or from the message merely appearing in the shared repository. When a message requests a delivery receipt (and for coordination/task messages where the sender needs proof that the target ship received it), the recipient must publish an explicit, durable acknowledgement after it has fetched the message and successfully picked it up from `new/` into `cur/`.

- The receipt is a **new message addressed to the original sender's own spool** (`spool/<sender>/new/<ack-message-id>`), using the same normal immutable envelope and Git publication rules.
- It must carry a dedicated acknowledgement type/status and identify the original logical `Message-ID`, original sender, acknowledging recipient, and the receipt time (UTC). Include the original Correlation-ID/Task-ID where present.
- The receipt confirms **delivery/pickup only**: the recipient fetched and durably recorded the message as picked up. It does not claim that the requested work was completed.
- Processing completion/failure must be reported separately with a distinct outcome acknowledgement, also correlated to the original Message-ID. Never conflate `delivered`, `processed`, and `completed`.
- A recipient must generate at most one logical receipt per original Message-ID and recipient. Retries reuse the same receipt Message-ID and exact bytes; duplicate receipt publication is idempotent. Persist enough state to recover if a crash occurs between pickup and receipt publication.
- The sender treats the message as delivery-confirmed only after it has fetched and validated that correlated receipt from its own spool. A push of the receipt by the recipient is not, by itself, proof the sender has received it.
- If a requested receipt does not arrive within the task's stated timeout, report delivery as unconfirmed and retry/escalate according to policy; do not silently mark the task delivered.
- Fanout requires an independent delivery receipt from every recipient for which confirmation was requested. One recipient's receipt cannot confirm delivery to another.

The envelope should therefore support explicit acknowledgement metadata, for example an `Ack-Type` (delivery or processing), `Acknowledges-Message-ID`, `Acknowledged-By`, `Acknowledged-At`, and an outcome/status where applicable. These are semantic fields, not mutable edits to the original message.

## 8. Multi-recipient and multi-session isolation

- Every recipient gets its own spool and independent `new/cur` state.
- One recipient moving its copy to `cur` cannot change another recipient's state.
- Multiple senders may publish into the same recipient spool; repository synchronization must serialize checkout mutations and preserve all distinct message files.
- A ship's active endpoint identity and any ChatGPT session identity remain separate principals. Do not use a mutable display name as authentication.
- If multiple independent sessions operate as the same ship, define ownership/lease or claim semantics before allowing concurrent `new -> cur` transitions. Do not let two consumers process the same message concurrently.

## 9. Relationship to Comms

Comms remains the interactive/low-latency channel. Wormhole spools provide durable, asynchronous, repository-backed delivery and recovery. The spool model intentionally follows Comms' per-ship spool convention, but this draft does not authorize bypassing the Comms Bus or duplicating its external-post/idempotency responsibilities. The integration boundary must be designed against the actual Comms API.

## 10. Required tests before implementation is accepted

- Two senders concurrently publish distinct messages to one recipient; neither is lost.
- Retry after push rejection preserves the same Message-ID and bytes.
- Duplicate publication with identical bytes is idempotent; same ID with different bytes fails closed.
- Multi-recipient fanout yields independent recipient state transitions.
- Atomic creation and `new -> cur` rename survive crash injection at each boundary.
- Recipient crash after pickup but before processing resumes without duplicate side effects.
- Concurrent consumers cannot both claim/process one message.
- Fetch/rebase/push races, same-path conflicts, interrupted rebase, and bounded retry exhaustion preserve all local messages.
- No force-push or destructive reset path exists.
- A push is never reported as read, processed, or acknowledged.
- A requested delivery receipt is emitted only after recipient pickup, is correlated to the original Message-ID, survives retry/restart, and is visible to the sender before delivery is marked confirmed.
- Delivery receipt and processing outcome acknowledgement are distinct; fanout requires independent receipts from each requested recipient.

## 11. Open details (do not guess in implementation)

- Exact envelope grammar and whether an existing RFC 5322 parser is appropriate.
- Stable ship-ID to spool-path mapping and authorization rules.
- Whether `cur` entries are retained indefinitely or compacted through a separate archival policy.
- Processing journal/claim model and safe multi-session semantics.
- Commit granularity, push cadence, retry/backoff values, and offline behavior.
- How this spool model integrates with the existing Comms Bus and W0-026 idempotency/recipient validation.
