# ADR-0008 — Message ID and Ordering Strategy

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-009 Message Ordering](../architecture-decisions/AD-009-message-ordering.md), [AD-008 Message Model](../architecture-decisions/AD-008-message-model.md) |
| **Related Research** | [RS-003 Message Ordering](../research/RS-003-message-ordering.md), [RS-002 Message Models](../research/RS-002-message-models.md) |

## Context

Messages need a globally unique immutable identity and a deterministic position within a conversation. Conflating these concerns (e.g., ordering only by ID or only by client time) fails INV-05 or offline/multi-device correctness.

## Decision

**Separate identity from order:**

1. **`MessageId`:** Client-generated **ULID** (UUIDv7 acceptable). Immutable. Primary idempotency key.
2. **`Sequence`:** Server-assigned per-conversation monotonic `int64`. Sole authoritative sort key within a conversation.
3. Do **not** order by client timestamps.
4. Keep the design **HLC-ready** for future multi-region without changing `MessageId`.

Detailed allocation rules: **ADR-0010** and **AD-009**.

## Consequences

- Clear contracts for storage indexes, sync cursors (AD-010), and clients.
- Requires a per-conversation sequence allocator.
- Coarse time leakage via ULID is accepted (metadata).

## Alternatives Considered

| Alternative | Reason not chosen |
|---|---|
| Client timestamp order | Violates INV-05 |
| Order by MessageId only | Weak total order / gap detection |
| HLC from day one | Premature for single-region launch |

## References

- AD-009, AD-008, RS-003, INV-02, INV-05
- ADR-0010, ADR-0032
