# AD-008 — Message Model (Decision Recommendation)

## Question

How is a message represented on the backend — its identity, structure, and stored fields — given that content is ciphertext and the server operates only on metadata?

---

## Background

**Business context:** The message is the atomic unit of the product. Its model determines what features are possible (edits, replies, reactions, forwards, receipts) and what the server can and cannot see.

**Technical context:** The server stores an encrypted payload plus routing metadata. It assigns an immutable identity and sequences messages for ordering (AD-009). Rich features (reply, reaction, edit) must be expressed without exposing content.

**Constraints:** Backend never decrypts (INV-01); one immutable Message ID (INV-02); no ID reuse across versions (INV-12); metadata minimization (AD-021).

---

## Requirements

- **Functional:** Store/route an encrypted message; support replies, reactions, edits, deletes, forwards, attachments — all over metadata + ciphertext.
- **Non-Functional:** Compact metadata; efficient retrieval and pagination (AD-035).
- **Security:** No plaintext, no content-derived metadata that leaks meaning; minimal necessary fields.
- **Scalability:** Model supports partitioning by conversation/time (AD-033).

---

## Alternatives

### Alternative A — Single opaque encrypted blob + minimal envelope metadata
- **Pros:** Maximum privacy (server sees least); simplest storage.
- **Cons:** Features like reply/reaction/edit need some structural metadata; pure-blob forces clients to encode everything, complicating server-side ordering and receipts.

### Alternative B — Encrypted content blob + explicit structural metadata (type, replyToId, edit version, timestamps, sender device, attachment refs)
- **Pros:** Enables all planned features while keeping content encrypted; server can route, order, paginate, and manage receipts on metadata; clear separation of concerns.
- **Cons:** Must carefully bound metadata to avoid leakage (e.g., reaction emojis should be encrypted; only counts/relations as needed).

### Alternative C — Fully client-defined schema stored as encrypted blob, server stores only ID + conversation + sequence + timestamp
- **Pros:** Strongest metadata minimization.
- **Cons:** Server cannot support server-mediated features (receipts, ordering nuances, admin metadata) well; pushes complexity and consistency risk to clients.

---

## Industry Research

- **Documented/informed pattern:** E2EE systems store an encrypted payload plus the minimal metadata required for routing and ordering (sender, conversation, timestamp, message id). Reactions and edits are typically modeled as related encrypted messages/events referencing the original by id.
- **Signal (documented):** Emphasizes minimal server-visible metadata (sealed sender reduces sender metadata exposure) — an informed direction for AD-021.
- **Matrix (documented):** Represents edits, reactions, and replies as events that relate to an original event id — a documented "relations" model consistent with Alternative B.

---

## Recommendation

**Recommend Alternative B:** an **encrypted content payload** plus a **bounded, explicit metadata envelope**: immutable `MessageId`, `ConversationId`, `SenderUserId`/`SenderDeviceId`, server `Sequence` (AD-009), server-receive timestamp, `type`, `replyToMessageId`, `editVersion`/`supersedesMessageId`, deletion tombstone flag, and attachment references (to encrypted blobs). Sensitive sub-content (e.g., reaction emoji, reply preview) is **encrypted**, not stored in clear. Metadata is minimized per AD-021.

**Why:** It enables every planned feature (replies, reactions, edits, deletes, forwards, receipts, attachments) and reliable server-side ordering/pagination while keeping all human-meaningful content encrypted. It aligns with the documented "relations" pattern and cleanly supports partitioning and sync.

**Trade-offs:** Requires disciplined metadata boundaries (AD-021 review) to prevent leakage; edit/delete introduce versioning/tombstones (AD-027/AD-030/AD-031).

**Risks:** Metadata over-collection leaking behavior (mitigated by AD-021 minimization contract); incorrect version/tombstone handling (addressed in AD-027/AD-030).

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Feature-complete, privacy-respecting message model; supports ordering, pagination, receipts.
- **Negative:** Metadata governance overhead; versioning/tombstone complexity.
- **Future Impact:** Foundation for AD-009 (ordering), AD-027 (lifecycle), AD-030 (versioning), AD-032/033 (storage/partitioning), AD-042 (delivery).

---

## Affected Documents

- `docs/30-domain/36-message-lifecycle.md`
- `docs/85-protocol/85.6-message-metadata.md`
- `docs/85-protocol/85.4-encryption-envelope.md`
- `docs/50-data/51-postgres-schema-and-migrations.md`

## Affected ADRs

- ADR-0008 (Message ID/Ordering), ADR-0013 (Message Versioning)

## Affected Modules

- Messaging

## Open Questions

- Exact minimal metadata field set (subject to AD-021 security review)?
- Are reactions modeled as related encrypted messages or as bounded metadata counts?
- How are attachment references represented without leaking media type/size precisely?

## Approval

- **Status:** Under Review
- **Owner:** Architecture
- **Review Date:** (pending)
- **Decision Date:** (pending)
