# RS-004 — Synchronization (Research)

| Field | Value |
|---|---|
| **Status** | Research — Evidence Only |
| **Owner** | Principal Software Architect |
| **Version** | 1.0.0 |
| **Last Updated** | 2026-07-15 |
| **Feeds Decision** | AD-010 Synchronization |
| **Respects Constraints** | AD-001..AD-006, and AD-009 ordering (per-conversation sequence) |

## Executive Summary

Multi-device synchronization in messaging is dominated by **incremental delta sync driven by a cursor/sync-token** over an ordered stream, backed by **server-side offline queues** and **client-side idempotent de-duplication**. Full snapshots are used only for cold start/backfill; full event sourcing on the client is rarely necessary. Given AD-009's per-conversation sequence, the evidence strongly favors **cursor/checkpoint delta sync (per-conversation cursor) + realtime push for liveness + bounded, paged backfill for new devices**, with conflict handling reduced to idempotent, deterministically-ordered application — exactly AD-010's proposal.

## Problem Statement

We must choose how a user's devices synchronize conversation state — catching up after disconnection, backfilling new devices, converging consistently — over slow/mobile networks and at scale, while the backend handles only ciphertext.

## Why This Decision Matters

Sync defines the multi-device promise (PER-03), reconnect latency (NFR-P-05), correctness under at-least-once delivery (INV-04), and battery/bandwidth cost on mobile. It is the mechanism that makes ordering (AD-009) and delivery observable to users consistently everywhere.

## Industry Research

- **Matrix (documented):** `/sync` with a `since` **sync token**; returns deltas (new events, state changes) since the token; long-poll for liveness; initial sync returns a bounded snapshot. A clear documented cursor/delta model.
- **Signal/WhatsApp (documented + pattern):** Messages for offline recipients are **queued server-side as ciphertext** and delivered on reconnect; multi-device companions receive their own encrypted copies and reconcile. Exact sync internals are not fully public (pattern).
- **Slack/Discord (documented + pattern):** Gateway/websocket for realtime; REST history endpoints with **cursor pagination** for backfill; clients reconcile via last-seen positions.
- **Telegram (documented):** Uses a `pts`/`qts`/`seq` update-state mechanism (documented in its API) — effectively sequence/version counters the client uses to fetch missed updates (a documented delta-sync design).
- **CRDTs / version vectors (documented, academic/industry):** Used where concurrent multi-writer conflict resolution is required (collaborative editing); generally heavier than needed for append-oriented chat.

## Publicly Documented Practices

- **Fact:** Matrix uses sync tokens for incremental delta sync with bounded initial sync.
- **Fact:** Telegram exposes `pts/qts/seq` update sequence counters for catch-up.
- **Fact:** Slack/Discord provide cursor-paginated history plus realtime gateways.
- **Fact:** Offline messages are queued server-side and delivered on reconnect across major messengers.

## Common Architectural Patterns

- **Pattern:** **Cursor/sync-token delta sync** — client persists a position; server returns changes after it.
- **Pattern:** **Realtime push + authoritative catch-up** — websocket for liveness; cursor sync as the source of truth for gap-filling.
- **Pattern:** **Bounded initial/backfill sync** — recent window first, older history on demand (cursor pagination).
- **Pattern:** **Idempotent apply + dedup by immutable ID** — safe under at-least-once/retries.
- **Pattern:** **Version vectors / CRDTs** — for concurrent conflict resolution (mostly unnecessary for append-only chat, relevant for editable settings/state).

## Alternative Designs

### Alternative A — Full snapshot sync on connect
Re-download conversation state each connect.

### Alternative B — Cursor / delta synchronization (per-conversation cursor over AD-009 sequence) + realtime push
Incremental deltas; realtime for liveness; paged backfill for cold start.

### Alternative C — Client-side event sourcing / global changefeed subscription
Per-device append-only event log the client folds; global change stream.

### Alternative D — CRDT / version-vector convergence
Conflict-free replicated state across devices.

## Advantages

- **A:** Simple; guaranteed completeness.
- **B:** Transfers only deltas; deterministic via AD-009; trivial gap detection; resumable; battery/bandwidth efficient; matches documented practice.
- **C:** Strong offline/real-time convergence; unified stream; good for many-conversation catch-up.
- **D:** Robust concurrent conflict resolution without central ordering.

## Disadvantages

- **A:** Prohibitively expensive at scale; slow reconnect; wasteful on mobile — unacceptable for large histories.
- **B:** Requires per-device/per-conversation cursor bookkeeping; large-history backfill must be paged/prioritized.
- **C:** More infrastructure (per-device changefeed); ordering still maps to per-conversation sequence; heavier to operate at launch.
- **D:** Overkill for append-only messaging; complex; higher storage/CPU; unnecessary given server-assigned ordering (AD-009).

## Trade-offs

The trade-off is **efficiency/determinism (B)** vs. **completeness cost (A)** vs. **infrastructure/complexity (C/D)**. Because AD-009 provides a per-conversation total order, conflict resolution collapses to idempotent ordered application — eliminating the need for CRDTs (D) in the message path. B leverages this directly and matches how Matrix/Telegram/Slack actually sync. A is only viable for cold start; C is a possible future enhancement for many-conversation clients.

## Security Considerations

- All synced payloads are ciphertext; new-device backfill never requires server decryption (INV-01).
- Sync tokens/cursors are metadata; they must not encode content and must be authorized to the requesting device (AD-002/AD-003).
- Backfill to a new device is gated by device trust (AD-006) and delivers only ciphertext the device's keys can decrypt.

## Scalability Considerations

- Delta sync minimizes server egress and DB reads vs. snapshots.
- Offline queues must be bounded and drained; very large backlogs are paged (cursor pagination, AD-009 sequence).
- Fan-out to many devices interacts with realtime backplane (Redis) and backpressure (future AD-016/DOC-064).

## Operational Considerations

- Gap detection via contiguous sequence enables self-healing and clear alerting.
- Cursor durability/replication is an operational concern (where cursors live, how they survive failover).
- Retry with idempotency (INV-04) requires dedup storage and backoff policy (feeds AD-023/AD-025 patterns).

## Mobile Considerations

- Delta sync + lazy backfill conserves battery and data; recent-first backfill improves perceived performance.
- Long-poll/websocket liveness with resumable cursors handles flaky networks and app suspension.
- Bounded payloads and pagination prevent memory spikes on constrained devices.

## Backend Considerations

- Sync endpoints/frames live in the Messaging/Sync slices; realtime via SignalR (future AD-015).
- Idempotent application keyed on immutable `MessageId` (AD-009) simplifies retries and duplicate suppression.

## Database Implications

- Per-device, per-conversation cursor state (durable, replicated); read patterns are range scans over (ConversationId, Sequence) — well served by the AD-009 index.
- Offline queue may be a table or Redis structure; retention/purge after delivery/ack.
- Backfill uses cursor pagination over the ordered message table.

## Future Evolution

- Add a per-device global changefeed (C) if many-conversation catch-up becomes a bottleneck.
- Multi-region sync must align with the ordering evolution (HLC, RS-003) to preserve convergence.
- Consider selective/partial sync (e.g., archived conversations on demand).

## Recommendation

*(Evidence-based recommendation; not an approval — AD-010 will decide.)* Adopt **Alternative B: cursor/checkpoint delta synchronization using per-conversation cursors over the AD-009 sequence, with realtime push for liveness, server-side offline queues, bounded paged backfill for new devices, and idempotent de-duplication by immutable `MessageId`.** CRDTs/version vectors (D) are unnecessary for the append-only message path given server-assigned ordering. This matches documented industry practice and corroborates AD-010.

## Open Questions

- New-device backfill policy: full history vs. recent window + on-demand older?
- Cursor storage/durability and failover behavior?
- Is a global per-device changefeed needed at launch or deferred?
- Retry/backoff and dedup-store specifics (coordinate with AD-023/AD-025 when reached)?

## References

- Matrix specification — `/sync`, sync tokens, incremental vs. initial sync.
- Telegram API documentation — `pts`/`qts`/`seq` update state and difference fetching.
- Slack / Discord API documentation — cursor pagination and realtime gateways.
- WhatsApp/Signal documentation — offline queueing and multi-device delivery.
- Shapiro et al., CRDTs; version vectors (industry pattern reference).
