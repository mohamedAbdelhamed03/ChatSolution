# ADR-0018 — Idempotency

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-042 Delivery Semantics](../architecture-decisions/AD-042-delivery-semantics.md), [AD-009 Message Ordering](../architecture-decisions/AD-009-message-ordering.md) |
| **Related Research** | [RS-005 Delivery & Acknowledgements](../research/RS-005-delivery-and-acknowledgements.md) |
| **Workshop / Review** | [WS-042](../architecture-decisions/workshops/WS-042-delivery-and-acknowledgements.md), [AR-042](../architecture-decisions/reviews/AR-042-delivery-and-acknowledgements.md) |

## Context

The platform operates under **at-least-once** transport (push, sync, HTTP retries). **INV-04** requires retriable commands to be safe to replay without duplicate side effects. AD-009 establishes MessageId idempotency on accept. AD-010 requires MessageId dedup on client apply. AD-042 adds DeliveryAck projections that must tolerate duplicate submissions.

Without a unified idempotency strategy, delivery paths produce false duplicate ticks, double-counted receipts, or divergent multi-device state.

## Decision

1. **Primary idempotency key for messages:** immutable **`MessageId`** (ULID) on accept, apply, and dedup (AD-009, AD-010).
2. **DeliveryAck idempotency key:** **`(MessageId, RecipientDeviceId)`** — upsert semantics; duplicate acks are no-ops (AD-042 D-INV-02).
3. **Accept idempotency:** duplicate accept with same MessageId returns the same AcceptAck + Sequence without a second insert (AD-009).
4. **Client apply idempotency:** re-applying the same envelope (push + sync overlap) must not duplicate local messages (AD-010 S-INV-02, S-INV-06).
5. **Server-side dedup store:** maintain durable uniqueness constraints matching the keys above; retries read existing outcome instead of re-executing side effects.
6. **Idempotency scope:** applies to all retriable write paths in Messaging, Delivery, and Sync until superseded by path-specific ADRs (e.g., read receipts AD-044).
7. **Observability:** track duplicate-effect suppression metrics; never log plaintext or keys (INV-11).

Normative detail: **AD-042**, **AD-009**, **AD-010**, **INV-04**.

## Consequences

### Positive

Safe retries under flaky mobile networks; predictable multi-device convergence; clear keys for DB unique indexes; aligns with industry E2EE messenger practice.

### Negative

Dedup storage and unique-index maintenance; clients must generate stable MessageIds before retry; operational alerting on anomalous duplicate rates.

## Alternatives Considered

| Alternative | Reason not chosen |
|---|---|
| Transport-level exactly-once | Impractical across heterogeneous clients and disconnects |
| Timestamp-based dedup | Clock skew; weaker than immutable MessageId |
| No server dedup (client-only) | Insufficient for accept and projection integrity |

## References

- AD-042, AD-009, AD-010, AD-008
- RS-005, INV-04 (`29.5-system-invariants`)
- ADR-0008, ADR-0010, ADR-0016
