# RS-005 — Delivery & Acknowledgements (Research)

| Field | Value |
|---|---|
| **Status** | Research — Evidence Only |
| **Owner** | Principal Software Architect |
| **Version** | 1.0.0 |
| **Last Updated** | 2026-07-18 |
| **Feeds Decision** | AD-042 Delivery Semantics |
| **Respects Constraints** | AD-001..AD-010 (Approved), INV-01, INV-04, INV-05 |

## Executive Summary

Production messengers separate **server accept**, **device delivery**, and **read** into distinct acknowledgement layers. Server accept means the message is durable and ordered; device delivery means a recipient device obtained (and typically decrypted) the ciphertext; read is a later user-attention signal. Industry practice favors **per-device offline queues**, **at-least-once transport with idempotent MessageId apply**, **best-effort realtime push**, and **authoritative catch-up** — aligning with AD-008 (receipts as projections), AD-009 (Sequence on accept), and AD-010 (hybrid push + sync). Evidence favors a **dual-ack model**: server AcceptAck plus per-device DeliveryAck projections, with read deferred to AD-044.

## Problem Statement

We must define how messages move from Accepted persistence to recipient devices, what acknowledgements mean, how multi-device delivery is tracked, and how offline/reconnect paths remain correct — without decrypting ciphertext on the server and without mutating the Message aggregate for delivery state.

## Why This Decision Matters

Delivery semantics determine sender UX (single vs double checkmarks), multi-device correctness, offline reliability, push wake-up design (AD-012), and the foundation for read receipts (AD-044). Ambiguity here causes duplicate UI ticks, lost offline messages, or incorrect “delivered to user” claims when only one device received the message.

## Industry Research

- **Signal (documented + community synthesis):** Per-device encryption fan-out; server holds ciphertext in a **per-device queue** until the device fetches it; retention windows are time-bounded; online devices pull over WebSocket/API after optional **data-only push wake-up**; delivery receipts may be issued after successful download/decrypt (sealed-sender path differs). Sesame specifies MessageRecords keyed by MessageID and delivery receipts that allow senders to drop pending records after successful decrypt.
- **WhatsApp (documented pattern + research):** Multi-device uses **independent per-device queues**; server ack vs device ack are distinct UI signals; offline messages retained until delivered or expired; at-least-once style delivery with client reconciliation.
- **Matrix (documented):** Federation + `/sync` make **event durability and sync** the recovery path; clients reconcile via sync tokens rather than relying solely on live push.
- **Academic (Careless Whisper, 2024):** Confirms that WhatsApp/Signal issue **independent per-device delivery receipts**; offline devices emit receipts on reconnect after fetch; distinguishes **server ack**, **device ack**, and **read receipt** layers; notes delivery receipts as a **presence/side-channel** risk.

## Publicly Documented Practices

- **Fact:** Signal Protocol / Sesame use MessageIDs for retry and delivery-receipt correlation across devices.
- **Fact:** Matrix relies on durable events + incremental sync for catch-up after missed live delivery.
- **Fact:** Major E2EE messengers treat offline storage as **ciphertext queues**, not plaintext mailboxes.
- **Industry pattern:** Server accept ≠ device delivery ≠ read.
- **Industry pattern:** At-least-once delivery with client-side dedup by immutable message identity.
- **Industry pattern:** Push notifications are wake-ups; content is fetched over an authenticated channel.

## Common Architectural Patterns

- **Pattern:** **Dual (or triple) ack ladder** — Accept → Delivered(device) → Read(user).
- **Pattern:** **Per-device mailbox/queue** — independent undelivered ciphertext per DeviceId.
- **Pattern:** **Hybrid live + catch-up** — websocket push for liveness; sync/queue drain for authority (already AD-010).
- **Pattern:** **Delivery receipt as first-class control message / projection** — does not rewrite message ciphertext.
- **Pattern:** **Idempotent ack upsert** — repeated DeliveryAck for same (MessageId, DeviceId) is a no-op.
- **Pattern:** **Retention-bounded offline store** — purge after all target devices delivered or TTL/policy (coordinates with AD-038).

## Alternative Designs

### Alternative A — Server-accept ack only
Sender learns only that the server Accepted + assigned Sequence. No device DeliveryAck to sender. Recipient devices still sync via AD-010.

### Alternative B — Dual ack: AcceptAck + per-device DeliveryAck projections
Keep AD-009 AcceptAck. Add durable **DeliveryReceipt** projections keyed by `(MessageId, RecipientDeviceId)` (or equivalent), advanced when a device obtains the message via live push apply or sync apply. Aggregate to user-level “delivered” only by explicit policy. Read remains AD-044.

### Alternative C — Per-user aggregated delivery (first device wins)
Track a single Delivered flag per `(MessageId, RecipientUserId)` when **any** device of the user applies the message.

### Alternative D — Exactly-once end-to-end delivery protocol
Attempt transactional exactly-once semantics across sender, server, and all recipient devices.

## Advantages

- **A:** Simplest server model; minimal metadata; fewer privacy side channels from device acks.
- **B:** Matches industry dual-checkmark UX; accurate multi-device; clear handoff to push/read; preserves Message immutability.
- **C:** Simpler UX aggregation; fewer receipt rows than per-device.
- **D:** Theoretically eliminates duplicates at the protocol layer.

## Disadvantages

- **A:** Sender cannot distinguish “queued on server” from “on a recipient device”; weak product parity; poor diagnostics.
- **B:** More projection storage/events; delivery acks are a known presence side channel if unconstrained; aggregation policy must be explicit.
- **C:** Hides which device has the message; misleading for linked devices; harder multi-device debugging; conflicts with per-device E2EE fan-out reality.
- **D:** Impractical under mobile disconnects; fights at-least-once networks; high complexity; not used as the primary guarantee by major messengers.

## Trade-offs

| Dimension | Lean |
|---|---|
| Accuracy vs simplicity | **B** over **C** for multi-device truth |
| Privacy vs UX | Device acks improve UX but need privacy controls (AD-020); **A** minimizes side channels at UX cost |
| Reliability model | At-least-once + idempotency (**B** with AD-010) over exactly-once (**D**) |
| Authority | Live push never sole authority; sync/queue drain remains authoritative (AD-010) |

## Security Considerations

- **INV-01:** Delivery path stores/routes ciphertext and metadata only; acks carry MessageId/Sequence/device identity — never plaintext.
- **INV-04:** Accept and DeliveryAck paths must be idempotent under retries.
- **AuthZ:** Only authorized recipient devices can advance their delivery projection; senders receive ack visibility only when conversation membership/policy allows.
- **Side channels:** DeliveryAck timing can reveal device online status (documented research risk). Mitigations belong with privacy settings and rate limits (AD-020 / moderation later), not by merging acks into Message ciphertext.
- **Push:** Wake-up payloads must remain data-only (feeds AD-012); no content in OS notifications by default for E2EE.

## Scalability Considerations

- Per-device receipt rows grow with devices × messages; require upsert semantics and pruning after retention.
- Fan-out of live pushes scales with online devices; offline load shifts to queue/sync drain.
- Ack aggregation for group senders must avoid N² chatty fan-in (batch/stack receipts — industry pattern).

## Operational Considerations

- Metrics: accept latency, push success, sync drain lag, undelivered queue depth, ack idempotent hit rate — metadata only (INV-11).
- Alert on growing undelivered queues and stuck devices.
- Poison/undecryptable messages need a client retry path (Sesame-style) without server decryption.

## Mobile Considerations

- Background restrictions require push wake + reconnect sync (AD-010/AD-012).
- Battery: coalesce acks where safe; avoid ack storms after long offline.
- Offline outbound remains Pending until AcceptAck (AD-009); inbound UI “delivered” waits for local apply + outbound DeliveryAck.

## Backend Considerations

- Delivery service consumes `MessageAccepted` (and sync apply signals) without owning Message aggregate writes for receipt state.
- SignalR frames are best-effort; durable truth is message store + delivery projections + device cursors.
- Clear separation: **Accept path** (Messaging Core) vs **Delivery projection path** (this service) vs **Read path** (AD-044).

## Database Implications

- Message store remains Sequence-ordered ciphertext (AD-009).
- Delivery projection table: unique `(MessageId, DeviceId)` with timestamps/status; optional user-level rollup materialized later.
- Offline undelivered view may be derived from device cursor lag vs max Sequence, or an explicit outbox — either must be consistent with AD-010 cursors.
- Indexes for: undelivered-by-device, receipts-by-message for sender sync.

## Future Evolution

- AD-044 Read Receipts build on DeliveryAck (read should not precede delivered for the same device under normal policy).
- AD-012 Push uses undelivered/online signals without carrying plaintext.
- Sealed-sender-style anonymous delivery receipts may be evaluated later; not required at launch.
- Multi-region: receipt upserts must be idempotent and causal with Sequence (HLC-ready from AD-009).

## Recommendation

*(Evidence-based recommendation; not an approval — AD-042 will decide.)* Adopt **Alternative B: dual acknowledgement model** —

1. **AcceptAck** — already defined by AD-009 (server Accepted + Sequence).
2. **DeliveryAck** — per-recipient-device projection when the device has obtained the message via live apply or authoritative sync apply; idempotent by `(MessageId, DeviceId)`.
3. Preserve AD-010 authority: push is liveness; sync/queue drain is recovery.
4. Do **not** mutate Message for delivery state (AD-008).
5. Reject exactly-once as the primary guarantee (**D**); reject first-device-wins as the sole tracking model (**C**) for the system of record (optional UX rollup may exist above per-device truth).
6. Defer read receipts to AD-044; coordinate offline retention TTL with AD-038 / OQ-SYNC-04.

## Open Questions

- User-level “delivered” rollup policy: all devices vs any trusted device vs primary device?
- Should DeliveryAck require successful client decrypt, or only durable local persist of ciphertext envelope?
- Offline queue TTL and max depth before drop/notify (product + SRE; ties AD-038)?
- Ack batching/stacking wire format and rate limits?
- Privacy default: delivery receipts enabled for Direct vs Group (AD-020)?

## References

- Signal Sesame specification — MessageRecords, retry requests, delivery receipts.
- Signal Wiki / public docs — per-device queues, push wake-up, delivery receipt behavior (community synthesis where noted).
- Matrix specification — sync-based catch-up after missed live delivery.
- Careless Whisper (arXiv:2411.11194) — server ack vs device ack vs read; per-device independent receipts.
- AD-008, AD-009, AD-010; RS-004; DOC-154; DOC-156.
