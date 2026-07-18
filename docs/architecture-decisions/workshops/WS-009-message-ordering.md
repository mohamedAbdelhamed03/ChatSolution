# WS-009 — Architecture Workshop: Message Ordering

| Field | Value |
|---|---|
| **Workshop ID** | WS-009 |
| **Topic** | Message Ordering |
| **Status** | Completed — Awaiting Decision |
| **Owner** | Principal Software Architect |
| **Version** | 1.0.0 |
| **Date** | 2026-07-18 |
| **Feeds Decision** | AD-009 Message Ordering |
| **Evidence Base** | [RS-003 Message Ordering](../../research/RS-003-message-ordering.md) |
| **Respects** | AD-001..AD-008 (Approved) |

> This workshop prepares a decision. It does **not** make the final architecture decision.

---

## Executive Summary

**RS-003** separates **identity** (`MessageId` ULID — already decided in AD-008) from **ordering position**. Evidence favors a **per-conversation server-assigned monotonic `Sequence`** as the authoritative total order, with gap detection, idempotent accept by `MessageId`, optimistic client pending order, and an **HLC-ready** path for future multi-region — without adopting HLC or client-timestamp ordering at launch.

---

## Context

| Constraint | Implication |
|---|---|
| INV-05 | Deterministic total order within a conversation |
| INV-02 / AD-008 | Immutable ULID `MessageId` ≠ order position |
| INV-04 | Idempotent accept / dedup |
| AD-007 | Order scoped per `ConversationId` |
| AD-008 | Envelope already reserves `Sequence`, timestamps |
| AD-010 (next) | Sync cursors will consume Sequence |

---

## Problem Statement

How do we assign, persist, and communicate a deterministic, gap-detectable message order that works offline, multi-device, under retries/duplicates, and remains ready for multi-region — without reading content?

---

## Investigation Topics

### 1. Ordering Model

| Approach | Correctness | Scale | Simplicity | Ops | Offline |
|---|---|---|---|---|---|
| Client timestamps | Fail (skew/tamper) | High | High | Low | Misleading |
| Client sequences only | Fail across devices | High | Med | Low | Local only |
| Lamport clocks | Causal, not total UX order | Med | Med | Med | Needs merge |
| Vector clocks | Causal; bulky | Poor at group size | Low | High | Complex |
| Timestamp / ID-only (Snowflake/ULID) | Approximate | High | High | Low | OK identity |
| Per-conversation server sequence | **Total + gaps** | Hot-conv contention | High | Low | Server assigns on accept |
| HLC global | Causal multi-region | High | Low | High | Good later |
| Hybrid: server Sequence + ULID id | **Best launch fit** | Sharded by conv | High | Low | Pending local |

**Recommendation lean:** RS-003 **Alternative C** — server `Sequence` + ULID identity; HLC later.

### 2. Source of Truth

**Authoritative order = server** at accept time. Clients may propose temporary local order for optimistic UI; reconciliation replaces local order with server `Sequence`. Clients never assign authoritative Sequence.

### 3. Conversation Sequence

- Every **Accepted** message gets exactly one `Sequence` (int64), strictly increasing per `ConversationId`.
- **Allocation:** atomic increment in same DB transaction as message insert (per-conversation counter row or `UPDATE ... RETURNING` on conversation_seq).
- **Uniqueness:** `UNIQUE (ConversationId, Sequence)`.
- **Overflow:** int64 → ~9e18; not a practical concern; monitor.
- **Recovery:** on failover, counter must be durable (same Postgres primary); never reuse sequences; gaps only from rejected txns (see failure section — prefer no gaps from aborted alloc).

**Allocation strategy preference:** single-row counter per conversation (`SELECT ... FOR UPDATE` + increment) or Postgres advisory/atomic update — pick at implementation; architecture requires **transactional allocate-with-insert**.

### 4. Client Ordering

```mermaid
sequenceDiagram
    participant UI as Client UI
    participant Q as Outbox Pending
    participant S as Server
    UI->>Q: Create MessageId ULID; localOrder++
    UI->>UI: Show optimistic (pending)
    Q->>S: Send(MessageId, ciphertext)
    S-->>Q: Ack(Sequence, serverReceivedAt)
    Q->>UI: Rebind bubble to Sequence; resort
```

- **Pending:** ordered by local device send clock / localOrder among Pending only.
- **After ack:** sort key = `(ConversationId, Sequence)`.
- **Conflict:** never invent Sequence client-side; if two Pendings ack out of send order, UI reorders by Sequence.

### 5. Offline Support

- Offline: assign MessageId, enqueue Pending, localOrder.
- On reconnect: flush in localOrder; server assigns Sequence by **accept arrival / txn order**, not client localOrder.
- Delayed uploads intercalate with others’ messages by server accept order — correct for INV-05.
- Retries: same MessageId → idempotent; same Sequence returned.

### 6. Duplicate Detection

| Key | Role |
|---|---|
| `MessageId` (ULID) | Primary idempotency / dedup key |
| Optional `Idempotency-Key` header | Transport-level alias if needed |
| `SenderDeviceId` | Audit / fan-out; not primary dedup |

**Rule:** Second accept with same MessageId returns original Sequence; does not allocate new Sequence; emit `DuplicateDetected` (metric/event).

### 7. Concurrent Sends

Multiple users/devices: server serializes Sequence allocation per conversation (row lock / transactional counter). Deterministic order = commit/accept order under that lock. Cross-conversation: independent sequences (no global order required).

### 8. Multi-Device Synchronization

- All devices sort by Sequence.
- Out-of-order delivery: buffer/reorder by Sequence; gap → fetch range (AD-010).
- Late ack: Pending remains until Ack; other devices learn via sync/push after accept.
- Reconnect: cursor = last contiguous Sequence (AD-010).

### 9. Multi-Region Readiness

| Launch (single region) | Future active-active |
|---|---|
| Server Sequence on primary | Per-region sequences insufficient alone |
| Path: keep MessageId; evolve position to HLC or region+seq | Document trigger (DOC-103) |
| Do not ship dual-primary Sequence today | Avoid split-brain order |

Trade-off: strong order in region now vs. availability across regions later — accept CP-leaning per conversation for launch.

### 10. Failure Scenarios

| Failure | Behavior |
|---|---|
| Server restart | Durable counter in DB; continue |
| DB failover | Promote replica with sync replication or accept brief RPO; counters consistent with data |
| Retry storms | Idempotent MessageId |
| Clock skew | Irrelevant for Sequence; timestamps informational |
| Lost ack | Client retries; same MessageId → same Sequence |
| Partial insert | Transaction rollback → no Sequence leak (allocate in same txn) |
| Duplicate submit | Dedup; no second Sequence |

### 11. Ordering Metadata (align AD-008)

| Field | Authority |
|---|---|
| `MessageId` | Client; identity |
| `ConversationSequence` / `Sequence` | Server; total order |
| `SenderDeviceId` | Client |
| `serverReceivedAt` | Server |
| `clientSentAt` | Client (untrusted) |
| `editVersion` | Server/app on edit (AD-008); **does not** get new Sequence |
| OrderingVersion / schema | Protocol version for future HLC |

**Edits/reactions:** do not allocate new Sequence; they reference MessageId. Optional side-channel events ordered by their own Sequence if modeled as messages — reactions as relations: no Sequence (AD-008).

### 12. Domain Events

| Event | Producer | Consumers |
|---|---|---|
| `SequenceAllocated` (may fold into MessageAccepted) | Messaging | Metrics |
| `MessageAccepted` | Messaging | Sync, fan-out, push |
| `MessageRejected` | Messaging | Client |
| `DuplicateDetected` | Messaging | Metrics/abuse |
| `SequenceGapObserved` | Client/Sync | Catch-up (AD-010) |

### 13. Domain Invariants (draft)

| ID | Invariant | Enforce |
|---|---|---|
| O-INV-01 | Accepted message has exactly one Sequence | D+DB |
| O-INV-02 | Sequence strictly monotonic per conversation | D+DB |
| O-INV-03 | Sequence never reused | D+DB UNIQUE |
| O-INV-04 | Duplicate MessageId → no second message/Sequence | A+DB |
| O-INV-05 | Sequence immutable after persist | D+DB |
| O-INV-06 | Client timestamps never authoritative for order | A |
| O-INV-07 | Edits do not allocate new Sequence | D |

### 14. Future Compatibility

| Feature | Strategy |
|---|---|
| Threads | Same conversation Sequence stream; thread = relation |
| Channels | Same Sequence model |
| Scheduled | Sequence at accept/fire time, not schedule time |
| TTL / edits / reactions | No new Sequence for mutations (AD-008) |
| Sync AD-010 | Cursor = Sequence |
| Analytics / streaming | Emit Accepted with Sequence |
| Multi-region | HLC upgrade path |

---

## Alternatives (RS-003 lettering)

| Alt | Summary | Lean |
|---|---|---|
| A | Client timestamps | Reject |
| B | Time-sortable ID only | Reject for INV-05 gaps |
| C | Server Sequence + ULID id | **Recommend** |
| D | HLC from day one | Defer |

---

## Recommendation

*(Not a decision.)* Adopt **RS-003 Alternative C**: per-conversation server `Sequence` + existing ULID `MessageId`; transactional allocate-with-insert; idempotent dedup; optimistic client pending; HLC-ready multi-region path.

---

## Open Questions

| ID | Question | Owner |
|---|---|---|
| OQ-ORD-01 | Counter implementation: row vs sequence object? | Backend |
| OQ-ORD-02 | Allow intentional gaps after aborted txn? | Architecture |
| OQ-ORD-03 | HLC adoption trigger criteria? | Architecture + SRE |
| OQ-MSG-05 | ULID vs UUIDv7 (identity; AD-008) | Architecture |

---

## References

- [RS-003](../../research/RS-003-message-ordering.md)
- AD-008, AD-007, INV-05
