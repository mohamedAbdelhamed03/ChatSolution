# ADR-0032 — Immutable Message Model

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-008 Message Model](../architecture-decisions/AD-008-message-model.md) |
| **Related Research** | [RS-002 Message Models](../research/RS-002-message-models.md) |
| **Workshop / Review** | [WS-008](../architecture-decisions/workshops/WS-008-message-model.md), [AR-008](../architecture-decisions/reviews/AR-008-message-model.md) |

## Context

The platform requires a stable representation of messages under E2EE (INV-01), immutable identity (INV-02/INV-12), multi-device sync, and rich relations (edit, delete, reply, reaction, forward, attachments). RS-002 and AD-008 select an immutable record with relations rather than in-place mutation or full event sourcing at launch.

## Decision

1. **Message** is an **aggregate root** in the Messaging module (not inside the Conversation aggregate).
2. **MessageId** is a client-generated **ULID** (UUIDv7 acceptable equivalent).
3. Store **ciphertext + bounded metadata envelope**; never plaintext content, reaction emoji, or reply previews.
4. **Edits** increment `editVersion` and replace ciphertext; MessageId unchanged; launch retains latest version only.
5. **Deletes/recalls** use **tombstones** (clear ciphertext; keep id/sequence position).
6. **Attachments** are encrypted object-storage references on the message.
7. **Replies** use `replyToMessageId` in the same conversation; **reactions** are relation rows with encrypted payloads; **forwards** are new messages with `isForwarded=true` (no source id by default).
8. **Delivered/Read** live in **separate projections**, not on the Message aggregate state.
9. **`Sequence`** field is reserved for per-conversation total order (**AD-009**); sync dedup uses MessageId (**AD-010**).

Normative detail: AD-008.

## Consequences

### Positive

- Correct multi-device edit/delete semantics; INV-compliant; extensible types/relations; clean module boundary with Conversations.

### Negative

- Tombstone/version retention jobs; disciplined metadata minimization required.

### Neutral

- Ordering algorithm and sync protocol decided in AD-009/AD-010.

## Alternatives Considered

| Alternative | Reason not chosen |
|---|---|
| Mutable in-place messages (RS-002 A) | Breaks INV-02/12; sync races |
| Full event-sourced message log (RS-002 C) | Excessive complexity/storage for launch |
| Opaque blob without structural metadata | Blocks receipts/relations/server mediation |

## References

- AD-008 Message Model (Approved)
- RS-002 Message Models
- AD-004, AD-006, AD-007
- ADR-0008 / ADR-0010 / ADR-0013 (related)
- `docs/30-domain/30-domain-model-overview.md`
