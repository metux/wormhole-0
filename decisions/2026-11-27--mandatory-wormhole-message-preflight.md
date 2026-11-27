# Decision: mandatory preflight validation for outbound Wormhole messages

**Date:** 2026-11-27  
**Status:** Accepted  
**Scope:** All senders publishing active messages to `mail/<recipient>/new/`

## Context

On 2026-11-27, the actionable XAverius continuation message
`w0-20261127T103800Z-xav-nautilus-no-review-gate` was published with an
unquoted YAML subject containing a colon. The manager rejected it with:

`invalid message front matter: yaml: line 4: mapping values are not allowed in this context`

The message was moved to the manager-owned `failed/` directory and never
imported, injected, or acted upon. This caused an apparent agent inactivity
that was actually a transport-format failure. A later corrected message was
successfully imported.

The failure was preventable before publication. Relying on the manager to
reject malformed messages is too late: the sender may incorrectly assume the
instruction was delivered.

## Decision

1. Every sender MUST validate the exact final message file before committing
   it under `mail/<recipient>/new/<message-id>.md`.
2. Use the validator bundled with the
   `X11Libre/mpbt-workspace:mtx/agent-config` branch at
   `.claude/skills/wormhole-communication/scripts/validate_message.py`.
3. Every YAML front-matter value MUST be a JSON double-quoted string. This is
   valid YAML and avoids ambiguous plain-scalar parsing, including colons in
   subjects. Additional project-specific metadata is allowed, but values must
   also be quoted strings.
4. The validator MUST exit zero with a `VALID` result before publication. A
   failed validation means do not commit; correct the file and rerun.
5. Validation covers front-matter delimiters, required fields, duplicate keys,
   path-safe identifiers, filename/message-ID agreement, recipient/mailbox
   agreement, canonical UTC timestamp, UTF-8 decoding, and non-empty body.
6. Never edit manager-owned `cur/`, `processed/`, or `failed/` entries to
   repair a message. Publish a new immutable message with a new ID, explicitly
   referring to the failed message ID.
7. After publication, verify the lifecycle separately: `new/` publication,
   `cur/` claim, `processed/` successful import, active-session injection,
   acknowledgement, and actual model-turn/work evidence. Do not equate any
   earlier stage with a later one.

## Validation command

```sh
python3 .claude/skills/wormhole-communication/scripts/validate_message.py \
  mail/<recipient>/<message-id>.md --recipient <recipient>
```

The validator tests are in the same `scripts/` directory and can be run with:

```sh
python3 -m unittest discover \
  -s .claude/skills/wormhole-communication/scripts -p 'test_*.py' -v
```

## Consequences

- Message authors must use the shared communication skill and validator.
- Subjects and all other front-matter values are consistently quoted.
- Malformed YAML, routing mistakes, duplicate keys, and empty messages are
  caught before publication rather than after a manager-side failure.
- This is a sender-side preflight control. It does not replace manager-side
  validation or the requirement to verify downstream delivery and execution.
