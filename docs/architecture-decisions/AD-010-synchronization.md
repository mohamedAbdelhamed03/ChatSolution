# AD-010 — Synchronization (Decision Recommendation)

## Question

How do a user's multiple devices synchronize conversation state — receiving missed messages in order, converging on a consistent view, and backfilling history for new devices — while the backend only handles ciphertext?

---

## Background

**Business context:** Multi-device continuity is a core product promise (PER-03). A device that was offline, or a newly added device, must reach a correct, gap-free view quickly.

**Technical context:** Building on ordering (AD-009) and the device model (AD-006), synchronization uses per-device cursors over per-conversation sequences. All synced payloads are ciphertext; decryption happens on the device.

**Constraints:** Deterministic ordering (INV-05); idempotency/dedup (INV-04); backend never decrypts (INV-01); reconnect+sync latency target (NFR-P-05).

---

## Requirements

- **Functional:** Deliver missed messages after reconnect; backfill history to a new device; detect and fill gaps; converge across devices.
- **Non-Functional:** Fast catch-up for typical backlog; bounded memory/bandwidth; resumable.
- **Security:** All transferred content is ciphertext; new-device backfill never requires server decryption.
- **Scalability:** Sync scales with active devices and message volume.

---

## Alternatives

### Alternative A — Full state re-download on connect
- **Pros:** Simple; guarantees completeness.
- **Cons:** Prohibitively expensive at scale; slow; wasteful for small deltas.

### Alternative B — Cursor/checkpoint-based incremental sync (per-device cursor over per-conversation sequence)
- **Pros:** Transfers only deltas; deterministic via AD-009 sequences; trivial gap detection; resumable; efficient.
- **Cons:** Requires per-device cursor state and careful multi-conversation cursor management.

### Alternative C — Event-log / changefeed subscription (global per-device change stream)
- **Pros:** Real-time convergence; unified stream; good for many conversations.
- **Cons:** More infrastructure (per-device change stream); ordering across conversations must still map to per-conversation sequences; heavier to operate initially.

---

## Industry Research

- **Documented/informed pattern:** Incremental sync via cursors/tokens is standard — e.g., Matrix uses a `since` sync token to fetch deltas; many mobile clients persist a "last seen" position per room and request newer events.
- **Signal/WhatsApp (informed):** Offline messages are queued server-side (as ciphertext) and delivered on reconnect; multi-device companions receive their own encrypted copies and reconcile.
- **Fact vs. pattern:** Token/cursor-based delta sync is documented (Matrix); exact consumer-app sync internals are generally not public but follow the same delta/queue pattern.

---

## Recommendation

**Recommend Alternative B (with an eventing assist from C):** **cursor/checkpoint-based incremental synchronization**, where each device maintains a **per-conversation cursor** over the AD-009 sequence and requests deltas on reconnect; the server delivers queued ciphertext in order and the client de-duplicates via `MessageId` (INV-04). Use realtime events (SignalR, AD-039) as the **live push channel** and the cursor sync as the **authoritative catch-up/gap-fill mechanism**. New-device backfill streams historical ciphertext bounded by cursors, never decrypted server-side.

**Why:** Cursor sync gives efficient, deterministic, resumable convergence that directly leverages AD-009 ordering and satisfies INV-04/05, while realtime events provide low-latency delivery when online. This is the proven delta+queue pattern and meets the reconnect latency NFR.

**Trade-offs:** Per-device, per-conversation cursor bookkeeping; backfill volume for very large histories (bounded/paged, ties to AD-035 cursor pagination).

**Risks:** Cursor drift or loss (recover by re-reading from a known checkpoint); large-history backfill cost (paged + prioritized recent-first); dedup correctness (keyed on immutable `MessageId`).

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Efficient, deterministic multi-device convergence; fast reconnect; E2EE-preserving backfill.
- **Negative:** Cursor state management; backfill paging complexity.
- **Future Impact:** Defines AD-015 (realtime protocol sync frames), AD-041 (offline recovery), and `85.9` synchronization protocol; interacts with AD-035 pagination.

---

## Affected Documents

- `docs/85-protocol/85.9-synchronization-protocol.md`
- `docs/50-data/54-message-sync-and-storage.md`
- `docs/30-domain/35-sequences/SQ-08-message-synchronization.md`
- `docs/30-domain/35-sequences/SQ-15-offline-recovery.md`

## Affected ADRs

- ADR-0016 (Offline Synchronization), ADR-0011 (Cursor Pagination)

## Affected Modules

- Messaging, Sync

## Open Questions

- Backfill policy for new devices: full history vs. recent window + on-demand older?
- Where are per-device cursors stored, and what is their durability/replication?
- Do we need a global per-device changefeed (C) at launch or later?

## Approval

- **Status:** Under Review
- **Owner:** Architecture
- **Review Date:** (pending)
- **Decision Date:** (pending)
