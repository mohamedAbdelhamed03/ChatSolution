# AD-009 — Message Ordering (Decision Recommendation)

## Question

How is a deterministic, total ordering of messages within a conversation established and communicated to clients, and what identifier scheme supports it at global scale?

---

## Background

**Business context:** Users expect messages to appear in a consistent, correct order on every device. Reordering or gaps are among the most damaging messaging bugs.

**Technical context:** Ordering must be deterministic per conversation, survive multi-device sync (AD-010), and work with at-least-once delivery + idempotency (AD-042). It underpins pagination (AD-035) and lifecycle (AD-027).

**Constraints:** Deterministic total order per conversation (INV-05); immutable Message ID (INV-02); backend never decrypts (INV-01, so ordering uses metadata only); millions of users, future multi-region.

---

## Requirements

- **Functional:** Assign each message a position; allow clients to sort deterministically and detect gaps.
- **Non-Functional:** Ordering assignment is fast and contention-tolerant; identifiers are compact and index-friendly.
- **Security:** Ordering metadata reveals no content.
- **Scalability:** Works within a single region now; extensible to multi-region later.

---

## Alternatives

### Alternative A — Client timestamps for ordering
- **Pros:** Simple; no server coordination.
- **Cons:** Clock skew and malicious clients break order; not deterministic; unacceptable for correctness.

### Alternative B — Per-conversation server-assigned monotonic sequence number (+ time-sortable ID like ULID for identity)
- **Pros:** Deterministic total order within a conversation; gap detection trivial (contiguous sequence); ULID gives globally unique, time-sortable identity; index-friendly.
- **Cons:** Requires a per-conversation sequence generator (contention per conversation, which is naturally sharded by conversation); cross-region needs coordination or HLC later.

### Alternative C — Global logical clock / Hybrid Logical Clocks (HLC) across regions
- **Pros:** Multi-region friendly; captures causality; future-proof for active-active.
- **Cons:** More complex; unnecessary for single-region launch; harder to reason about for a v1.

---

## Industry Research

- **Documented pattern:** Time-sortable unique IDs are widely used — Twitter Snowflake and ULID are documented schemes for globally unique, roughly time-ordered identifiers.
- **Discord (documented):** Uses Snowflake IDs (timestamp-based) for messages, giving global uniqueness and time ordering.
- **Informed pattern:** Per-conversation/room sequence numbers (stream positions) are a common way to guarantee gap detection and total order within a room (e.g., Matrix stream ordering). HLCs are the documented approach when multi-region causal ordering is required.

---

## Recommendation

**Recommend Alternative B for launch, with a documented path to C:** assign each message a **per-conversation server sequence number** for deterministic total ordering and gap detection, and a **ULID** as the immutable global `MessageId` (time-sortable, unique). Keep the design **HLC-ready** so multi-region active-active (AD-034 / DOC-103) can adopt hybrid logical clocks without changing the message identity contract.

**Why:** Per-conversation sequences give exactly the guarantee INV-05 requires (total order + gap detection) with contention naturally partitioned by conversation, while ULIDs provide clean, index-friendly global identity. It avoids the fragility of client timestamps and the premature complexity of HLC, but does not paint us into a corner for multi-region.

**Trade-offs:** A per-conversation sequence source is needed (DB sequence/append position); cross-region ordering is deferred to the HLC path.

**Risks:** Sequence-generation contention on very hot conversations (mitigated by conversation-level partitioning, AD-033); multi-region reordering if introduced prematurely without HLC (explicitly deferred).

*(Not approved — recommendation only. Resolves the ADR-0008 / ADR-0010 open decision.)*

---

## Consequences

- **Positive:** Deterministic, gap-detectable ordering; scalable identity; multi-region-ready.
- **Negative:** Per-conversation sequence machinery; multi-region ordering deferred.
- **Future Impact:** Enables AD-010 (sync cursors), AD-035 (cursor pagination), AD-027 (lifecycle), AD-042 (delivery/dedup).

---

## Affected Documents

- `docs/50-data/54-message-sync-and-storage.md`
- `docs/85-protocol/85.5-delivery-state-machine.md`
- `docs/85-protocol/85.9-synchronization-protocol.md`
- `docs/30-domain/36-message-lifecycle.md`

## Affected ADRs

- ADR-0008 (Message ID and Ordering Strategy), ADR-0010 (Message Ordering)

## Affected Modules

- Messaging

## Open Questions

- ULID vs. Snowflake for `MessageId` (both viable; ULID recommended)?
- Sequence source: database sequence, append position, or per-conversation counter?
- When is multi-region (HLC) adoption triggered?

## Approval

- **Status:** Under Review
- **Owner:** Architecture
- **Review Date:** (pending)
- **Decision Date:** (pending)
