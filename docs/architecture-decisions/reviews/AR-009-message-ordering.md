# AR-009 — Architecture Review: Message Ordering

| Field | Value |
|---|---|
| **Review ID** | AR-009 |
| **Topic** | Message Ordering |
| **Status** | Completed |
| **Reviewer** | Chief Software Architect (Architecture Review Board) |
| **Reviewed Artifact** | [WS-009](../workshops/WS-009-message-ordering.md) |
| **Evidence** | [RS-003](../../research/RS-003-message-ordering.md) |
| **Date** | 2026-07-18 |
| **Verdict** | Approve with Changes |

---

## 1. Review Scope

Challenge WS-009’s lean toward RS-003 Alternative C (per-conversation server Sequence + ULID identity) under concurrency, failover, offline, and multi-region readiness.

---

## 2. Challenge Findings

### 2.1 Hidden assumptions

| Assumption | Challenge | Outcome |
|---|---|---|
| "No gaps from aborted txns" | If Sequence allocated before insert commits, gaps appear | **Required:** Allocate Sequence **in the same transaction** as insert; aborted txn releases lock without publishing Sequence |
| "int64 forever" | Fine practically | Document; no wrap strategy needed at launch |
| "Edits keep Sequence" | Clients may expect edit to "move to bottom" | **Required:** Product UX may bump display; **ordering position stays**; emit MessageEdited; AD-008 already says this — restate |
| Draft AD-009 lettering ≠ RS-003 | Confusion | **Required:** AD-009 uses **RS-003 A/B/C/D** lettering |

### 2.2 Race conditions

| Race | Disposition |
|---|---|
| Two accepts same conversation | Serialized by counter lock → deterministic |
| Duplicate MessageId concurrent | Unique MessageId constraint; one winner; other DuplicateDetected |
| Pending localOrder vs Sequence | UI must resort on ack — document as mandatory client rule |

### 2.3 Clock / distributed edge cases

Client clock skew irrelevant for Sequence — **good**. `serverReceivedAt` must not be used as sort key when Sequence present. Multi-region dual-primary Sequence **forbidden** until HLC design.

### 2.4 Hot conversation contention

Per-conversation lock is the scalability limit (RISK-03). Mitigation: efficient counter update; conversation shard affinity; monitor p99 allocate latency. Not a reason to switch to ID-only ordering.

### 2.5 Storage

`UNIQUE (ConversationId, Sequence)` + index `(ConversationId, Sequence)` for range sync. Aligns AD-008.

### 2.6 Recovery complexity

Lost ack + retry is simple with MessageId idempotency. **Gap in sync** (missing Sequence) is AD-010 responsibility; ordering decision must guarantee contiguous allocation under success path and define whether gaps can exist (failed after commit but before fan-out ≠ sequence gap).

**Clarify:** After successful commit, Sequence is durable even if client never got ack. Sync recovers. True holes only if we skip numbers — **policy: do not skip**; only gaps clients see are **not-yet-fetched** or **purged tombstones still occupying Sequence**.

### 2.7 Tombstone / purge

Purging message rows must not reuse Sequence. Either keep tombstone stub with Sequence or document that purge creates intentional gaps and sync treats gap as "gone" vs "missing". **Required for AD-009:** Launch — **retain tombstone metadata row** (AD-008) so Sequence space remains explainable; hard purge later may create gaps marked as purged in AD-010.

### 2.8 Future scalability

HLC path must not change MessageId (RS-003). Sequence may become HLC component or secondary — document as evolution, not dual write now.

---

## 3. Alternative Re-evaluation

| Alt | Survives? |
|---|---|
| A Client timestamps | **No** |
| B ID-only order | **No** for INV-05 gap detection |
| C Server Sequence + ULID | **Yes, with changes** |
| D HLC day one | **No** for launch |

---

## 4. Required Changes to AD-009

1. Cite RS-003; use RS-003 lettering (choose C).
2. Normative: transactional Sequence allocate + insert; MessageId idempotency.
3. Explicit client optimistic UI rules and resort-on-ack.
4. Edits/reactions do not get new Sequence.
5. Invariants O-INV-* with D/A/DB.
6. Multi-region: single-region authority now; HLC-ready; no dual Sequence.
7. Tombstones retain Sequence; define gap meaning for sync handoff.
8. Sequence source: require atomic per-conversation counter in DB transaction (implementation choice noted as OQ-ORD-01).
9. Ratify ADR-0008 + ADR-0010.
10. Align with AD-008 field names (`Sequence`).

---

## 5. Verdict

**Approve with Changes.** Overall **9.4**.

---

## 6. Sign-off

| Role | Result |
|---|---|
| Principal Software Architect | Approve with Changes |
| Distributed Systems Engineer | Approve with Changes (txn allocate; no dual-primary) |
| Database Architect | Approve with Changes (UNIQUE + counter in txn) |
| Reliability Engineer | Approve with Changes (idempotent retry; durable Sequence) |
| Security Architect | Approve with Changes (server authority; no content in order metadata) |
