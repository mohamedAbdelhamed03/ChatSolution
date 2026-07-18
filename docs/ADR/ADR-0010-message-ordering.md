# ADR-0010 — Per-Conversation Message Ordering

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-009 Message Ordering](../architecture-decisions/AD-009-message-ordering.md) |
| **Related Research** | [RS-003 Message Ordering](../research/RS-003-message-ordering.md) |
| **Companion** | [ADR-0008 Message ID and Ordering Strategy](ADR-0008-message-id-and-ordering-strategy.md) |

## Context

INV-05 requires a deterministic, gap-detectable total order of messages within each conversation under concurrency, retries, offline send, and multi-device sync.

## Decision

1. On accept, allocate `Sequence` via **atomic per-conversation counter in the same DB transaction** as the message insert.
2. Enforce `UNIQUE (ConversationId, Sequence)`.
3. Sequence is **strictly monotonic** for successive successful accepts; **never reused**; **immutable** after persist.
4. Duplicate `MessageId` → return existing Sequence; do not allocate.
5. Edits, tombstones, and reactions **do not** allocate a new Sequence.
6. Tombstone rows **retain** Sequence.
7. Clients sort by Sequence; optimistic `localOrder` is temporary until Ack.
8. Single-region Sequence authority at launch; multi-region requires future HLC design (no dual-primary Sequence).

Normative detail: **AD-009**.

## Consequences

### Positive

Gap detection, idempotent retries, stable sync cursors, predictable UI reconciliation.

### Negative

Hot-conversation counter contention; multi-region ordering deferred.

## Alternatives Considered

See AD-009 / RS-003 Alternatives A–D. Chosen: server Sequence (RS-003 C).

## References

- AD-009, ADR-0008, RS-003, AD-008, INV-05
- Future: ADR-0016 (sync), DOC-103 (multi-region)
