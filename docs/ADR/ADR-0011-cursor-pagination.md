# ADR-0011 — Cursor Pagination

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-010 Synchronization](../architecture-decisions/AD-010-synchronization.md), [AD-009 Message Ordering](../architecture-decisions/AD-009-message-ordering.md) |
| **Related Research** | [RS-004 Synchronization](../research/RS-004-synchronization.md) |

## Context

History backfill and incremental sync must page large conversations efficiently using the AD-009 total order.

## Decision

1. Paginate message history and sync deltas by **ConversationSequence** ranges: `Sequence > cursor ORDER BY Sequence ASC LIMIT pageSize` (catch-up) and descending ranges for recent-first backfill.
2. Do **not** use offset/limit pagination for sync correctness.
3. Enforce a **maximum page size** server-side.
4. Clients treat pages as idempotent; advance contiguous sync cursor only after successful apply.
5. Attachment bytes are **not** inlined in pages; only refs (AD-008).

## Consequences

Stable, gap-aware sync and history APIs; index-friendly range scans on `(ConversationId, Sequence)`.

## Alternatives Considered

| Alternative | Reason not chosen |
|---|---|
| Offset pagination | Breaks under inserts; poor for sync |
| Timestamp cursors | Clock skew; not INV-05 |
| Opaque encrypted sync tokens hiding Sequence | Extra indirection; Sequence already metadata |

## References

- AD-010, AD-009, ADR-0016, AD-008
