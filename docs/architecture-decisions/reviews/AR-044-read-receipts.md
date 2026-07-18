# AR-044 — Architecture Review: Read Receipts

| Field | Value |
|---|---|
| **Review ID** | AR-044 |
| **Topic** | Read Receipts |
| **Status** | Completed |
| **Reviewer** | Chief Software Architect (Architecture Review Board) |
| **Reviewed Artifact** | [WS-044](../workshops/WS-044-read-receipts.md) |
| **Evidence** | [RS-006](../../research/RS-006-read-receipts.md) |
| **Feeds Decision** | AD-044 Read Receipts |
| **Date** | 2026-07-18 |
| **Verdict** | Approved for Architecture Decision |

---

## Executive Summary

This review evaluates the proposed Read Receipt architecture for the Messaging Services layer.

The proposed design introduces a **user-scoped read watermark** (`readUpToSequence` per `(UserId, ConversationId)`) as the system of record, with per-message read state derived for UI. Read is defined as **intentional user view**, independent from AD-042 device delivery.

The review confirms that the model preserves the Messaging Core and Delivery baselines, satisfies the Sprint 2 charter criteria, and provides deterministic multi-device read consistency with explicit privacy controls.

No architectural conflicts were identified.

---

## 1. Review Scope

Per the Sprint 2 charter, this review confirms:

* No mutation of the Message Aggregate.
* No impact on ConversationSequence ordering.
* Compatibility with AD-042 delivery semantics.
* Synchronization remains authoritative.
* Aggregate boundaries remain unchanged.
* Read state is reconstructible.
* Multi-device consistency is preserved.

Out of scope: presence, typing indicators, push notifications, analytics, moderation.

---

## 2. Architectural Consistency

| Architecture Decision | Result |
|---|---|
| AD-007 — Conversation Model | Compatible |
| AD-008 — Message Model | Compatible |
| AD-009 — Message Ordering | Compatible |
| AD-010 — Synchronization | Compatible |
| AD-042 — Delivery Semantics | Compatible |

No contradictions were identified. Read receipts consume the acknowledgement ladder without altering it: AcceptAck (AD-009) → DeliveryAck (AD-042) → ReadCursor (AD-044).

---

## 3. Review Findings

### 3.1 User-Scoped Read Watermark

Approved. `readUpToSequence` per `(UserId, ConversationId)` correctly represents user attention. Storage is O(users × conversations), independent of message volume. A message M is read for user U iff `Sequence(M) ≤ readUpToSequence(U)`.

**Result:** Approved

### 3.2 Independence from Delivery

Approved. Delivery confirms device apply (AD-042); Read confirms intentional user view. The projections are stored and evolved independently (R-INV-09). ReadCursor advancement requires local message availability, ordering Read after Delivery on the same device, without coupling the two records.

**Result:** Approved

### 3.3 Aggregate Boundaries

Approved. Read state lives exclusively in the ReadCursor projection. Message remains immutable; Conversation owns membership only. No new fields on core aggregates.

**Result:** Approved

### 3.4 Ordering

Approved. Read never allocates, modifies, or reorders ConversationSequence. Read timestamps are metadata, never sort keys.

**Result:** Approved

### 3.5 Multi-Device Consistency

Approved. The watermark is user-scoped: reading on one device advances the cursor for the user; other devices converge via AD-010 read deltas. Offline concurrent reads merge deterministically by **max Sequence**. Per-device read as the system of record is rejected.

**Result:** Approved

### 3.6 Synchronization

Approved. ReadCursor deltas ride the existing AD-010 hybrid: push best-effort, delta sync authoritative. Read state is recoverable on reconnect. Read updates are not synchronization checkpoints and never affect message cursors.

**Result:** Approved

### 3.7 Privacy Model

Approved with one required clarification (see §6). Outbound ReadAck to peers is gated by privacy settings (global and per-conversation). When disabled, the user's own devices still synchronize read state for self-consistency. Privacy changes apply at emission time; historical read state is never forged or retracted.

**Result:** Approved

### 3.8 Reconstructibility

Approved. Read state is a projection keyed by `(UserId, ConversationId)`; clients can re-report their local watermark to rebuild server state after projection loss. An event-sourced audit log remains a deferred option (RS-006 D).

**Result:** Approved

### 3.9 Group Aggregation

Approved. Sender-side group views derive from per-member watermarks (e.g., "read by N members up to Sequence S"). No per-message × per-reader matrix is stored for full history at launch.

**Result:** Approved

---

## 4. Rejected Alternatives

### Per-Message Read Rows as Primary Store (RS-006 A)

Rejected as system of record. Storage and sync cost O(messages × readers) is unacceptable in large groups. Permitted only as a derived/materialized view for recent UI windows.

### Per-Device Read Records (RS-006 C)

Rejected. Read is a user-attention concept. Device-scoped read fragments UX across linked devices and amplifies presence side channels.

### Event Sourcing as Primary Model (RS-006 D)

Deferred. Adds operational complexity without launch benefit. The projection design does not preclude adding an event log later.

---

## 5. Architecture Validation

| Property | Result |
|---|---|
| Message Immutability Preserved | Pass |
| Aggregate Ownership Preserved | Pass |
| Ordering Preserved | Pass |
| Synchronization Authoritative | Pass |
| Delivery/Read Independence | Pass |
| Multi-Device Determinism | Pass |
| Privacy Enforceability | Pass |
| Read State Reconstructible | Pass |
| Horizontal Scalability | Pass |

---

## 6. Required Changes to AD-044

1. Cite RS-006; adopt Alternative B (user watermark) with A-derived UI views.
2. Normative **ReadCursor** contract: monotonic max-merge; idempotent replay; unique `(UserId, ConversationId)`.
3. Normative **read emission conditions** (all required): local availability, intentional-view policy, authenticated owner, privacy gate for outbound.
4. **Resolve OQ-READ-03:** the watermark may only advance over Sequences the reporting device has locally applied (contiguous with its sync cursor); it must not assert reads for messages the user has never been able to view. Gaps pending on *other* devices do not block advancement — the watermark is user-scoped, and the reporting device's contiguous local state is the authority for what was viewable.
5. **Privacy epoch:** record the privacy setting in effect at emission; suppressed receipts are never emitted retroactively after a setting change.
6. R-INV-01..09 with enforcement layers; add invariant that outbound receipt visibility requires conversation membership at read time.
7. Event contract: `ReadCursorAdvanced` (metadata only); batched wire format allowed (OQ-READ-04 to protocol docs).
8. Explicit statement that read receipts introduce no new ADR-0018 exceptions — same idempotency discipline.
9. Defer notification-preview policy (OQ-READ-01) and default-on/off policy (OQ-READ-02) to Product/AD-020 without blocking approval.

---

## 7. Risks

No high-risk architectural issues identified.

| Concern | Disposition |
|---|---|
| Read timing side channel | Documented; mitigated by privacy opt-out and rate limits (AD-020) |
| Watermark regression bugs | Monotonic max-merge enforced at API + DB constraint |
| Group watermark fan-out chatter | Debounce/coalesce read deltas; batch sync |

---

## 8. Verdict

**Approved for Architecture Decision.**

The proposed Read Receipt architecture:

* Preserves the Messaging Core and Delivery baselines.
* Keeps Delivery and Read independent concepts.
* Scales to large conversations via watermarks.
* Provides deterministic multi-device convergence.
* Makes privacy enforceable at the emission boundary.

**Overall score:** 9.4

---

## 9. Sign-off

| Role | Result |
|---|---|
| Chief Software Architect | Approved for Architecture Decision |
| Distributed Systems Engineer | Approved (max-merge determinism) |
| Privacy & Security Architect | Approved (emission-time privacy gate) |
| Mobile Client Engineer | Approved (local-availability rule) |
| Messaging Specialist | Approved |

**Recommendation:** Proceed to Architecture Decision **AD-044**.
