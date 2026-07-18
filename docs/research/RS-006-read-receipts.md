# RS-006 — Read Receipts (Research)

| Field | Value |
|---|---|
| **Status** | Research — Evidence Only |
| **Owner** | Principal Software Architect |
| **Version** | 1.0.0 |
| **Last Updated** | 2026-07-18 |
| **Feeds Decision** | AD-044 Read Receipts |
| **Respects Constraints** | AD-001..AD-010, AD-042 (Approved), INV-01, INV-04, INV-05 |

## Executive Summary

Read receipts in E2EE messengers represent **user attention**, distinct from **device delivery** (AD-042). Industry practice combines **optional privacy controls**, **encrypted receipt/control messages**, and either **per-message read markers** or **read-position watermarks** (highest-read sequence). For large conversations, watermarks dominate on bandwidth and storage. Multi-device products either **synchronize read position across a user's devices** (convergent UX) or emit **independent device reads** (more side-channel surface). Evidence favors **user-scoped read watermarks** (`readUpToSequence` per `(UserId, ConversationId)`) as the system of record, synced via AD-010 delta types, with **per-message precision derived** for UI; read remains a **projection** outside the Message aggregate (AD-008).

## Problem Statement

Senders need to know when recipients have **intentionally viewed** messages, without conflating that with device delivery, without mutating Message or Sequence, and without breaking E2EE or multi-device convergence. The design must scale to large group histories and support privacy opt-out.

## Why This Decision Matters

Read receipts affect UX trust, privacy (stalking/side channels), group coordination, and downstream analytics. Incorrect coupling to delivery causes false "read" states; per-message storage at scale is expensive; disabling read receipts must be enforceable without redesigning the core.

## Industry Research

- **WhatsApp (documented + pattern):** Optional read receipts ("blue ticks"). Delivery (double tick) and read (blue) are separate. Read occurs when recipient **opens/views** the conversation/message. Multi-device: delivery receipts are per-device; read behavior is user-visible when enabled. Receipts travel as encrypted control traffic; server routes metadata/ciphertext only.
- **Signal (documented):** Read receipts are **optional** in settings. Distinct from delivery. Encrypted messenger pattern; exact internal sync not fully public.
- **iMessage (documented):** Optional read receipts per sender preference; "Delivered" vs "Read" are separate UI states.
- **Slack (documented):** Channel/conversation **read cursor** / last-read position; clients sync read state; efficient for high-volume channels (watermark pattern).
- **Microsoft Teams (documented + pattern):** Read receipts in supported chat modes; often aggregated read indicators in groups rather than per-message storage for entire history.
- **Threema (research):** May synchronize delivery receipts across devices; stricter privacy posture on probing (Careless Whisper, 2024).
- **Academic (Careless Whisper, 2024):** Read receipts are a distinct message type returned when user accesses message; disable supported on WhatsApp/Signal/Threema; read timing is a **presence/side-channel** risk.

## Publicly Documented Practices

- **Fact:** Major consumer E2EE messengers separate delivery and read indicators.
- **Fact:** Read receipts are commonly **user-configurable** (global or per-chat).
- **Fact:** Slack-style products use **read cursors / last-read timestamps** for channel scale.
- **Industry pattern:** Read receipts as **encrypted control messages** or **metadata projections** — not plaintext on server.
- **Industry pattern:** Read state sync across a user's own devices for consistent UX.
- **Industry pattern:** Idempotent, monotonic read advancement (never regress read position without explicit reset).

## Common Architectural Patterns

- **Pattern:** **Ack ladder** — Accept → Delivered (device) → Read (user attention) — AD-042 + AD-044.
- **Pattern:** **Read watermark** — store `readUpToSequence` per user per conversation; implies all messages ≤ N are read.
- **Pattern:** **Per-message read row** — `(MessageId, UserId)` timestamp; precise; costly at scale.
- **Pattern:** **Hybrid** — watermark authoritative; materialize per-message view for recent window or group member list.
- **Pattern:** **Privacy gate** — if recipient disabled read receipts, suppress outbound ReadAck to senders (may still track locally).
- **Pattern:** **Sync delta type** — read cursor updates replicate via same hybrid push + authoritative sync as messages (AD-010).

## Alternative Designs

### Alternative A — Per-message ReadReceipt rows `(MessageId, UserId)`
Each read emits one durable row per message per user.

### Alternative B — User read watermark `readUpToSequence` per `(UserId, ConversationId)`
Monotonic Sequence cursor representing highest intentionally viewed message; derived "read up to here" for all prior messages.

### Alternative C — Per-device read (mirror DeliveryAck)
Each device emits independent ReadAck `(MessageId, DeviceId)`.

### Alternative D — Read event sourcing only
Append-only read events; projections rebuilt by fold — no direct watermark table.

## Advantages

- **A:** Maximum precision; simple mental model per message; easy group member lists for recent messages.
- **B:** O(1) storage per user per conversation; efficient sync; natural fit with AD-009 Sequence; scales to large histories.
- **C:** Aligns with per-device delivery; granular device audit.
- **D:** Full audit trail; flexible projections; good for analytics pipelines.

## Disadvantages

- **A:** Storage and sync volume O(messages × readers); expensive in large groups; ack storms on bulk mark-read.
- **B:** Requires monotonicity rules; "read" for middle messages inferred; group UI may need derived per-recipient watermarks.
- **C:** Conflicts with "user viewed" semantics; multi-device UX fragmentation; amplified side channels (Careless Whisper).
- **D:** Higher operational complexity; still need idempotent fold and retention policy; overkill at launch.

## Trade-offs

| Dimension | Lean |
|---|---|
| Precision vs scale | **B** watermark primary; derive UI for groups |
| User vs device semantics | **User-level read** (charter) over **C** |
| Privacy vs UX | Optional outbound receipts; local read tracking may continue |
| Authority | AD-010 sync for read deltas; push best-effort |
| Independence from delivery | Read projections separate; ReadAck requires message available locally (post-delivery policy) |

## Security Considerations

- **INV-01:** Read metadata (UserId, ConversationId, Sequence, timestamps) only — no plaintext content on server.
- **Side channels:** Read receipt timing reveals user attention patterns; mitigated by opt-out (AD-020) and rate limits.
- **AuthZ:** Only the user (via authenticated device) may advance their read watermark; senders see others' read state only when permitted by membership and privacy settings.
- **INV-04:** Read advancement idempotent; monotonic — replay of same or lower Sequence is no-op.

## Scalability Considerations

- Watermark: one row per active (UserId, ConversationId) vs millions of per-message rows.
- Group fan-out: sender syncs read watermarks for members, not per-message fan-in for full history.
- Bulk mark-read on open conversation: single watermark update, not N receipts.

## Operational Considerations

- Monotonic watermark violations indicate client bugs or replay attacks — alert.
- Privacy setting changes must not retroactively forge read states.
- Rebuild: watermarks reconstructible from read event log if event sourcing hybrid used later.

## Mobile Considerations

- Background read on notification open must define policy (advance watermark or not until in-app view).
- Offline read: queue watermark update; sync on reconnect; idempotent merge (max Sequence wins).
- Battery: coalesce read updates; debounce rapid scroll mark-read.

## Backend Considerations

- Read service consumes local client read reports; stores projections; publishes `ReadStateAdvanced` deltas.
- Does not mutate Message aggregate or ConversationSequence allocation.
- Integrates with AD-042: read path separate from DeliveryReceipt table.

## Database Implications

- Primary: `ReadCursor (UserId, ConversationId, readUpToSequence, updatedAt)` unique per pair.
- Optional: recent per-message materialization for group UI (cache/rollup).
- Indexes: by conversation for group sender views; monotonic compare-and-max upsert.

## Future Evolution

- Sealed-sender read receipts; read receipts for reactions/edits; channel-specific policies; analytics on metadata with consent.

## Recommendation

*(Evidence-based recommendation; not an approval — AD-044 will decide.)* Adopt **Alternative B as primary** with **A-derived UI where needed**:

1. **User-scoped read watermark** `readUpToSequence` per `(UserId, ConversationId)` — monotonic, idempotent.
2. **Read independent from Delivery** — separate projection; ReadAck emission gated on intentional view + local message availability (post-delivery).
3. **Multi-device:** synchronize read watermark across user's devices via AD-010 read deltas; convergent UX.
4. **Privacy:** outbound read receipts suppressed when disabled; coordinate AD-020.
5. **Groups:** per-member watermarks; sender aggregates "read up to N" per recipient.
6. Reject **C** as system of record (device-scoped read); reject **D** at launch (defer event-sourcing audit log).

## Open Questions

- Notification preview open: advance read or not?
- Partial read in long message / thread boundary?
- Default privacy: read receipts on for Direct, off for Groups?
- Watermark conflict when user reads on two devices offline — max Sequence merge rule sufficient?

## References

- AD-042, AD-008, AD-010, AD-009, RS-005
- Careless Whisper (arXiv:2411.11194) — read vs delivery; privacy
- Slack API — read cursors / channel read state (documented pattern)
- WhatsApp / Signal / iMessage — optional read receipts (documented product behavior)
- DOC-156 Phase 4 Sprint 2 charter
