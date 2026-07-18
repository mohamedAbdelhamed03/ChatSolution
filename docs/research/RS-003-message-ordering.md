# RS-003 — Message Ordering (Research)

| Field | Value |
|---|---|
| **Status** | Research — Evidence Only |
| **Owner** | Principal Software Architect |
| **Version** | 1.0.0 |
| **Last Updated** | 2026-07-15 |
| **Feeds Decision** | AD-009 Message Ordering |
| **Respects Constraints** | AD-001..AD-006 |

## Executive Summary

Deterministic ordering requires two separable concerns: **message identity** (globally unique, ideally time-sortable) and **ordering position** (a total order within a conversation). Industry practice pairs a time-sortable unique ID (Snowflake, ULID, or UUIDv7) with either implicit ID-time ordering or an explicit per-conversation sequence/stream position. Client timestamps alone are universally avoided due to clock skew and tampering. For a single-region launch with a clear multi-region path, the evidence favors **per-conversation server sequence numbers for total order + a time-sortable ULID/UUIDv7 for identity**, keeping the design **HLC-ready** for future active-active — precisely AD-009's proposal.

## Problem Statement

We must choose the identifier scheme and ordering mechanism that give a deterministic, gap-detectable total order of messages within a conversation, across offline and multi-device scenarios, at global scale, without the backend reading content.

## Why This Decision Matters

Ordering is a correctness invariant (INV-05). Reordering, gaps, or duplicates are the most damaging messaging bugs and directly harm trust. The scheme also affects database index locality, pagination (cursor), sync (AD-010), and multi-region evolution.

## Industry Research

- **Discord (documented):** Uses **Snowflake IDs** — 64-bit, timestamp-based, globally unique, roughly time-ordered; message ordering derives from the timestamp-embedded ID.
- **Twitter (documented):** Originated **Snowflake** for time-sortable distributed IDs.
- **ULID / UUIDv7 (documented):** ULID (spec) and UUIDv7 (RFC 9562) are lexicographically/time sortable unique identifiers designed for index locality and distributed generation.
- **Matrix (documented):** Uses server-assigned **stream ordering** (per-homeserver stream positions) plus topological ordering (DAG) for events; clients sort by stream order.
- **Signal/WhatsApp (industry pattern):** Exact internal ordering is not fully public; behavior indicates server-assigned ordering with client reconciliation; client timestamps are not authoritative.
- **Lamport / Hybrid Logical Clocks (documented, academic/industry):** Lamport clocks provide causal ordering without wall-clock; HLCs (documented, used by CockroachDB and others) combine physical + logical time for causal, roughly-real-time ordering in distributed/multi-region systems.

## Publicly Documented Practices

- **Fact:** Snowflake (Twitter/Discord) embeds a timestamp for time-sortable global IDs.
- **Fact:** ULID and UUIDv7 are specified as time-sortable unique identifiers.
- **Fact:** Matrix uses server stream positions for ordering.
- **Fact:** HLCs are used in distributed databases (e.g., CockroachDB) for causal ordering across nodes/regions.

## Common Architectural Patterns

- **Pattern:** **Separate identity from order** — a unique ID for identity, a monotonic sequence/stream position for total order within a scope (conversation/room).
- **Pattern:** **Time-sortable IDs** for index locality and coarse global ordering without coordination.
- **Pattern:** **Per-scope sequence** for exact, gap-detectable total order within a conversation.
- **Pattern:** **HLCs** when causal ordering across regions is required.
- **Anti-pattern:** **Client-timestamp ordering** — rejected due to skew and tampering.

## Alternative Designs

### Alternative A — Client timestamps
Order by device-provided time.

### Alternative B — Time-sortable global ID only (Snowflake / ULID / UUIDv7), order by ID
Single ID provides identity and approximate order.

### Alternative C — Per-conversation server sequence for order + time-sortable ULID/UUIDv7 for identity
Explicit total order per conversation; unique time-sortable identity; HLC-ready for multi-region.

### Alternative D — Hybrid Logical Clocks globally
Causal ordering via HLC across regions from day one.

## Advantages

- **A:** Trivial; no coordination.
- **B:** Simple; one value; good index locality; decent global ordering; distributed generation (ULID/UUIDv7 client- or server-side).
- **C:** Exact total order + trivial gap detection per conversation; clean pagination; robust to skew; multi-region-ready via HLC later.
- **D:** Correct causal ordering across regions; future-proof for active-active.

## Disadvantages

- **A:** Non-deterministic; breaks under clock skew and malicious clients; violates INV-05. Unacceptable.
- **B:** Approximate order only; concurrent messages within the same millisecond or across nodes can tie/reorder; gap detection not inherent.
- **C:** Requires a per-conversation sequence source (contention per hot conversation, naturally sharded by conversation); cross-region needs HLC later.
- **D:** Highest complexity now; unnecessary for single-region launch; harder to reason about and operate for v1.

## Trade-offs

The key trade-off is **exactness/gap-detection (C/D)** vs. **simplicity (B)** vs. **naivety (A, rejected)**. B is attractive but its lack of an inherent total order and gap detection undermines INV-05 and reliable sync. D solves multi-region but is premature. C delivers the exact guarantee INV-05 requires with contention partitioned by conversation, and preserves a clean upgrade path to HLC — the best balance for launch.

## Security Considerations

- Ordering metadata (sequence, timestamp, ID) must reveal no content (INV-01); IDs must not encode content-derived data.
- Server-assigned sequence prevents malicious clients from forging order.
- Time-sortable IDs leak coarse creation time — acceptable and already implied by message metadata.

## Scalability Considerations

- Per-conversation sequence contention is bounded because it is partitioned by conversation; hot conversations may need an efficient counter (DB sequence, append position, or atomic increment).
- Time-sortable IDs improve B-tree index locality, reducing write amplification for append-heavy message tables.
- Multi-region introduces cross-region ordering needs → HLC path.

## Operational Considerations

- Gap detection (contiguous sequence) enables self-healing sync and clear monitoring (missing-sequence alerts).
- A per-conversation counter is simple to operate within a single region; multi-region requires careful design (HLC) to avoid regressions.

## Mobile Considerations

- Deterministic per-conversation order lets clients sort locally and detect gaps to trigger catch-up sync (AD-010).
- Compact IDs reduce storage and payload size on device.

## Backend Considerations

- Sequence assignment happens at persistence in the Messaging slice; identity (ULID/UUIDv7) can be generated at creation.
- Design must keep the message-identity contract stable so multi-region (HLC) does not require changing IDs.

## Database Implications

- Time-sortable primary/clustering key improves insert locality (PostgreSQL); ULID/UUIDv7 recommended over random UUIDv4 to avoid index fragmentation.
- Per-conversation sequence via a dedicated sequence/counter or a monotonic append position; unique index on (ConversationId, Sequence) enforces total order and gap detection.
- Partition by conversation + time (feeds AD-033).

## Future Evolution

- Adopt **HLC** for message positioning when multi-region active-active is introduced (DOC-103), without changing the `MessageId` contract.
- Consider UUIDv7 vs. ULID based on library/DB support at implementation time.

## Recommendation

*(Evidence-based recommendation; not an approval — AD-009 will decide.)* Adopt **Alternative C: per-conversation server sequence numbers for deterministic total order and gap detection, combined with a time-sortable ULID (or UUIDv7) as the immutable `MessageId`, kept HLC-ready for multi-region.** This satisfies INV-05, provides clean pagination and sync, avoids client-clock fragility, and preserves a non-disruptive multi-region path — corroborating AD-009.

## Open Questions

- ULID vs. UUIDv7 for identity (both viable; choose per DB/library support)?
- Sequence source: PostgreSQL sequence, per-conversation counter row, or append/stream position?
- Precise trigger and design for HLC adoption in multi-region?

## References

- Twitter Snowflake (engineering blog) — time-sortable distributed IDs.
- Discord engineering — Snowflake IDs and message storage.
- ULID specification; RFC 9562 (UUIDv7).
- Matrix specification — stream and topological ordering.
- Kulkarni et al., "Logical Physical Clocks" (HLC); CockroachDB documentation (HLC usage).
