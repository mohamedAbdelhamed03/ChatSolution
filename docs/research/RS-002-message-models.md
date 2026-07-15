# RS-002 — Message Models (Research)

| Field | Value |
|---|---|
| **Status** | Research — Evidence Only |
| **Owner** | Principal Software Architect |
| **Version** | 1.0.0 |
| **Last Updated** | 2026-07-15 |
| **Feeds Decision** | AD-008 Message Model |
| **Respects Constraints** | AD-001..AD-006 |

## Executive Summary

Across messaging platforms, the message is modeled as an **immutable core record** with mutations (edits, deletes, reactions, pins) expressed as **related events referencing the original by ID** rather than in-place overwrites. Edits keep history (or at least an "edited" marker); deletes are tombstones/soft-deletes with optional delete-for-everyone; reactions/replies/quotes are relations. In E2EE systems, all human-meaningful sub-content (edited text, reaction emoji, reply preview) must remain ciphertext; the backend stores only bounded relational metadata. This favors, for AD-008, an **immutable message record + bounded metadata envelope + relations model**, with edit-as-new-version and soft-delete tombstones.

## Problem Statement

We must decide how a message is represented — identity, mutability, and the modeling of edits, deletes, reactions, replies, quotes, forwards, attachments, mentions, and pins — under E2EE where the server sees ciphertext + metadata only.

## Why This Decision Matters

The message model determines which features are possible and how privacy is preserved. It dictates edit/delete semantics across devices, storage growth, sync/ordering interplay (AD-009/AD-010), and whether the backend can inadvertently learn content through over-collected metadata. It is foundational to the entire messaging domain.

## Industry Research

- **WhatsApp (documented):** Supports edit (within a time window), delete-for-me and delete-for-everyone (within a window), reactions, replies/quotes, forwarding (with "forwarded" labeling and forward limits), attachments, mentions, and pinned messages; all content E2EE.
- **Signal (documented):** Edit and delete-for-everyone (within windows), reactions, replies/quotes, view-once media, disappearing messages; strong metadata minimization.
- **Telegram (documented):** Edit with history-less in-place update (edited marker), delete-for-everyone (unlimited in many cases), reactions, replies, forwarding with attribution, pinned messages; cloud chats server-readable.
- **Discord (documented):** Edits update in place with an "edited" marker, soft/hard delete, reactions (emoji), replies, pins, attachments, mentions/roles; not E2EE; rich relations model via message references.
- **Slack (documented):** Edit history/edited marker, delete, reactions, threaded replies, pins, attachments, mentions; not E2EE.
- **Microsoft Teams (documented):** Edit/delete with markers, reactions, threaded replies, mentions, pins; not E2EE for channel messages.

## Publicly Documented Practices

- **Fact:** WhatsApp/Signal implement edit and delete-for-everyone as **time-bounded** operations.
- **Fact:** Forwarding is commonly labeled and sometimes rate-limited (WhatsApp forward limits) to curb virality/abuse.
- **Fact:** Reactions, replies, and edits are represented as events/relations referencing an original message ID (documented in Matrix; behaviorally consistent across platforms).
- **Fact:** Disappearing/expiring messages (TTL) are offered by Signal, WhatsApp, and Telegram.

## Common Architectural Patterns

- **Pattern:** **Immutable message + relations** — the original message is never mutated in place; edits create a new version linked to the original ID; reactions/replies are separate related records.
- **Pattern:** **Soft delete / tombstone** — deletion replaces content with a tombstone (metadata marker) preserving ordering position and the immutable ID; hard delete/purge runs later per retention.
- **Pattern:** **Attachment-by-reference** — media stored separately (encrypted blob) and referenced by the message.
- **Pattern:** **Bounded metadata** — server stores only relational metadata; sensitive sub-content stays encrypted.

## Alternative Designs

### Alternative A — Mutable messages (in-place edit/delete)
Overwrite content on edit; remove row on delete.

### Alternative B — Immutable message record + relations (edit-as-new-version, soft-delete tombstone, reactions/replies as relations)
Original immutable; mutations modeled as related events referencing the original ID.

### Alternative C — Full event-sourced message log (every change is an event; state is a projection)
The message is a fold over an append-only event stream.

## Advantages

- **A:** Simple storage; small footprint; trivial reads.
- **B:** Preserves immutable identity (INV-02/INV-12); consistent multi-device edit/delete via events; auditable; supports edit history and tombstones; privacy-friendly (bounded metadata).
- **C:** Complete history and auditability; natural fit for sync via event log; strong consistency reasoning.

## Disadvantages

- **A:** Breaks immutable-ID guarantees; hard to sync edits/deletes deterministically across devices; loses history; race conditions.
- **B:** More records (versions, tombstones, relations); requires disciplined metadata boundaries and retention/purge for tombstones.
- **C:** Highest storage and complexity; projections everywhere; likely over-engineered for a messaging product at launch.

## Trade-offs

**A** optimizes simplicity but violates our invariants (immutable ID, deterministic multi-device semantics) and is unacceptable. **C** offers the cleanest history/sync story but at significant storage and cognitive cost. **B** captures most event-sourcing benefits (immutability, relations, deterministic sync) without a full event-store, aligning with how leading platforms actually behave and with AD-009/AD-010.

## Security Considerations

- Edited text, reaction emoji, reply previews, and mentions targets that reveal content must be **encrypted**; only the relation (references original ID, is-a-reaction) is metadata.
- Delete-for-everyone removes ciphertext and leaves a tombstone; it cannot guarantee deletion from a recipient who already decrypted/exported — a documented E2EE limitation to communicate.
- Forwarding re-encrypts on the client for new recipients (AD-004); server sees only a new message with optional "forwarded" metadata.
- Metadata minimization (AD-021 upcoming) governs what relational fields are stored.

## Scalability Considerations

- Immutable-with-relations scales via append + partitioning (AD-009/AD-033); tombstones and old versions are purged by retention to bound growth.
- Reactions can be high-volume; modeling them as compact related records (or bounded aggregates) avoids hot-row contention on the original message.

## Operational Considerations

- Retention/purge jobs must reconcile tombstones and expired (TTL) messages while preserving ordering integrity.
- Edit/delete windows are policy knobs requiring configuration (feeds AD-030/AD-031 and later config decisions).

## Mobile Considerations

- Clients must render edited/deleted/expired states and reconcile relations after sync; a relations model maps well to local stores.
- Attachment-by-reference lets clients lazy-load media, saving bandwidth/battery.

## Backend Considerations

- Immutable core simplifies idempotent writes (INV-04) and caching.
- Relations (reply/reaction/edit) are additional writes referencing the original ID; handlers live in the Messaging slice.

## Database Implications

- `Message` (immutable): `MessageId` (ULID), `ConversationId`, sender, sequence, timestamp, `type`, ciphertext payload, `replyToMessageId`, `supersedesMessageId`/`editVersion`, `isTombstoned`, attachment refs.
- Reactions/pins as related tables keyed by `MessageId`; edit versions chained via `supersedesMessageId`.
- Partition by conversation + time; retention/purge for tombstones and TTL messages.

## Future Evolution

- Add disappearing-messages TTL and scheduled messages.
- Threads (parent-referencing messages) fit the relations model if introduced.
- Richer reactions (custom/animated) remain compatible.

## Recommendation

*(Evidence-based recommendation; not an approval — AD-008 will decide.)* Adopt **Alternative B: an immutable message record plus a bounded metadata envelope and a relations model** — edit-as-new-version (preserving the original immutable ID), soft-delete tombstones with optional time-bounded delete-for-everyone, and reactions/replies/quotes/forwards as relations, with all sensitive sub-content encrypted and attachments stored by reference. This mirrors documented platform behavior and upholds INV-01/INV-02/INV-12, corroborating AD-008's direction.

## Open Questions

- Edit and delete-for-everyone time windows?
- Are reactions modeled as related encrypted messages or bounded metadata counts?
- Is full edit history retained, or only an "edited" marker plus latest version?
- Forwarding attribution/limits for abuse control (coordinate with AD-021 and abuse decisions)?

## References

- WhatsApp Encryption Overview (whitepaper); WhatsApp feature docs (edit, delete, reactions, communities, forward limits).
- Signal documentation/blog (edit, delete, disappearing messages, reactions).
- Telegram documentation (edit, delete, reactions, forwarding).
- Discord / Slack / Microsoft Teams documentation (edits, threads, reactions, pins).
- Matrix specification (event relations: edits, reactions, replies).
