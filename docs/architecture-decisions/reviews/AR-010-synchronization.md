# AR-010 — Architecture Review: Synchronization

| Field | Value |
|---|---|
| **Review ID** | AR-010 |
| **Topic** | Synchronization |
| **Status** | Completed |
| **Reviewer** | Chief Software Architect (Architecture Review Board) |
| **Reviewed Artifact** | [WS-010](../workshops/WS-010-synchronization.md) |
| **Evidence** | [RS-004](../../research/RS-004-synchronization.md) |
| **Date** | 2026-07-18 |
| **Verdict** | Approve with Changes |

---

## 1. Review Scope

Challenge WS-010’s lean toward RS-004 Alternative B (cursor delta sync + realtime push) against AD-001..AD-009, especially ConversationSequence as canonical cursor.

---

## 2. Challenge Findings

### 2.1 Hidden assumptions

| Assumption | Challenge | Outcome |
|---|---|---|
| "Cursor = last received Sequence" | Non-contiguous receive would advance incorrectly | **Required:** Cursor = highest Sequence such that **all** 1..cursor (within retention window / after backfill base) are applied — **last contiguous** |
| "Push can be ignored" | Clients that only push starve | **Required:** On every reconnect and periodic, run delta sync |
| "Per-conversation cursors only" | Many conversations → many round trips | **Accepted at launch**; optional mailbox watermark later (RS-004 C deferred); document fan-in API: sync multiple conversations in one request |
| Draft AD-010 lettering vs RS-004 | A/B/C vs A/B/C/D | **Required:** Use RS-004 lettering in AD-010 |

### 2.2 Failure recovery

Lost push is fine if pull is mandatory. Cursor loss: **Required** server may store server-side checkpoint hint per device, but **client-durable cursor is primary**; server hint optional. Reset policy: product chooses recent window vs from zero (OQ-SYNC-01).

### 2.3 Duplicate delivery

Push + sync overlap must be safe — MessageId dedup **mandatory** on client. Server sync pages may repeat Sequence ranges on retry — idempotent.

### 2.4 Event ordering

Apply strictly by Sequence; buffer out-of-order push frames. Edits: apply by MessageId + editVersion without new Sequence (AD-008/009).

### 2.5 Operational complexity

Cursor table growth: one row per (device × conversation touched). Mitigate: lazy cursor create on first sync; prune on device revoke.

### 2.6 Scalability

Hot sync after outage: stampeding herd. **Required:** jittered reconnect; batched multi-conversation sync; server rate limits; page size caps.

### 2.7 Security

Backfill to untrusted device forbidden (AD-006). Sync API must check membership at request time; revoked member gets 403 and no further deltas. Ciphertext only.

### 2.8 Mobile

App suspension: treat as disconnect → sync on resume. Large pages cause OOM — hard max page size.

### 2.9 Future evolution

Do not block on global changefeed. Multi-region: cursors remain per-conversation Sequence until HLC migration (AD-009).

---

## 3. Alternative Re-evaluation

| Alt | Survives? |
|---|---|
| A Snapshot primary | **No** |
| B Cursor delta + push | **Yes, with changes** |
| C Global changefeed | **Defer** |
| D CRDT | **No** for messages |

---

## 4. Required Changes to AD-010

1. Cite RS-004; choose Alternative B; align lettering.
2. Normative: **ConversationSequence** = canonical cursor; **contiguous advancement**.
3. Hybrid SignalR + authoritative pull; pull on reconnect mandatory.
4. Multi-conversation batch sync API at launch (not only N single calls).
5. Idempotency MessageId; conflict table from workshop.
6. S-INV-* with enforcement layers.
7. Device trust gate for backfill; authz on every sync.
8. Stampede mitigations; page bounds.
9. Ratify ADR-0016 (and ADR-0011 for cursor pagination).
10. Distinguish live push vs recovery clearly with diagrams.

---

## 5. Verdict

**Approve with Changes.** Overall **9.4**.

---

## 6. Sign-off

| Role | Result |
|---|---|
| Principal Software Architect | Approve with Changes |
| Distributed Systems Engineer | Approve with Changes (contiguous cursor; stampede control) |
| Reliability Engineer | Approve with Changes (pull-on-reconnect mandatory) |
| Security Architect | Approve with Changes (authz + device trust on sync) |
| Messaging Specialist | Approve with Changes (Sequence cursor; MessageId dedup) |
