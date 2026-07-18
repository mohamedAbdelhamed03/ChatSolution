# ADR-0016 — Offline Synchronization

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-010 Synchronization](../architecture-decisions/AD-010-synchronization.md) |
| **Related Research** | [RS-004 Synchronization](../research/RS-004-synchronization.md) |
| **Workshop / Review** | [WS-010](../architecture-decisions/workshops/WS-010-synchronization.md), [AR-010](../architecture-decisions/reviews/AR-010-synchronization.md) |

## Context

Devices disconnect frequently. Multi-device users require convergent, gap-free conversation state. The platform already has per-conversation server `Sequence` (AD-009) and immutable `MessageId` (AD-008). Synchronization must not decrypt content (INV-01).

## Decision

1. Use **hybrid push + pull**: SignalR for live delivery; **delta sync** as authoritative recovery.
2. Canonical sync cursor = **ConversationSequence** per `(DeviceId, ConversationId)`, advanced only for **contiguous** applied Sequences.
3. On reconnect/resume, clients **must** run delta sync (not push-only).
4. Apply sync/push payloads **idempotently** by `MessageId`.
5. Support **batched multi-conversation** sync requests at launch.
6. New-device backfill is paged, recent-first, and gated by **device trust** (AD-006).
7. Outbound offline: Pending queue with MessageId retries (AD-009).

Normative detail: **AD-010**. Pagination mechanics: **ADR-0011**.

## Consequences

### Positive

Deterministic convergence, efficient catch-up, E2EE-preserving backfill, clear operational model.

### Negative

Cursor state management; reconnect stampede controls; backfill product policy knobs.

## Alternatives Considered

| Alternative | Reason not chosen |
|---|---|
| Full snapshot each connect (RS-004 A) | Too costly |
| Global changefeed day one (RS-004 C) | Deferred |
| CRDTs for messages (RS-004 D) | Unnecessary with server Sequence |

## References

- AD-010, RS-004, AD-009, AD-008, AD-006
- ADR-0011, ADR-0008, ADR-0010
