# RS-001 — Conversation Models (Research)

| Field | Value |
|---|---|
| **Status** | Research — Evidence Only |
| **Owner** | Principal Software Architect |
| **Version** | 1.0.0 |
| **Last Updated** | 2026-07-15 |
| **Feeds Decision** | AD-007 Conversation Model |
| **Respects Constraints** | AD-001..AD-006 (identity, auth, authz, E2EE, keys, devices) |

## Executive Summary

Modern messaging platforms converge on a small set of conversation primitives — direct (1:1), group, and broadcast (channel) — but differ sharply in how much structure they layer on top (threads, topics, communities) and in their membership/permission models. Consumer E2EE messengers (Signal, WhatsApp) keep the model deliberately simple to preserve encryption guarantees and privacy; collaboration platforms (Slack, Teams, Discord) add rich structure (channels, threads, roles) at the cost of complexity and, typically, without message-content E2EE. For this project — E2EE-first, backend-never-decrypts — the evidence favors a **unified conversation aggregate with a type discriminator (Direct | Group | Channel-future) and a separate membership/role model**, which is exactly the direction to validate in AD-007.

## Problem Statement

We must choose how conversations are modeled so that 1:1 and group messaging work today, channels are possible later, and membership/roles support authorization (AD-003) — all while message content stays E2EE and the backend operates only on metadata.

## Why This Decision Matters

The conversation model is the container for every message, membership change, and key-distribution event. It determines the uniformity of the message pipeline, the cost of group encryption (AD-004 sender-keys), sync (AD-010), storage/partitioning, and whether channels/communities can be added without a rewrite. A poor choice forces duplicated pipelines or an authorization/encryption redesign later.

## Industry Research

- **Signal (documented):** Direct and group chats; groups use a private group system (Signal's "Groups v2") with server-blind membership where feasible; no channel/community broadcast product comparable to Telegram. Emphasis on minimal metadata.
- **WhatsApp (documented):** 1:1 and groups (bounded size, raised over time), Broadcast lists (sender-side fan-out to individual chats), Communities (a documented feature grouping related groups with an announcement group), and Channels (a documented one-to-many broadcast product). All personal chats are E2EE (Signal Protocol).
- **Telegram (documented):** 1:1, groups, supergroups, and channels (broadcast); cloud chats are not E2EE by default (secret chats, which are 1:1 only, are E2EE). Large supergroups and channels are a core differentiator.
- **Discord (documented):** Guilds (servers) containing text/voice channels, threads, categories, and a rich role/permission system; built for communities; messages are not E2EE.
- **Slack (documented):** Workspaces containing channels (public/private), DMs, and threads; enterprise features; not E2EE by default.
- **Microsoft Teams (documented):** Teams containing channels, threaded conversations, chats; recently added E2EE for 1:1 calls (documented), but channel messages are generally not E2EE.

## Publicly Documented Practices

- **Fact:** WhatsApp and Signal use the Signal Protocol for E2EE personal/group messaging; group scaling uses sender-key style distribution (documented in the WhatsApp/Signal whitepapers).
- **Fact:** Telegram secret chats are 1:1 only and E2EE; group/channel content is server-readable.
- **Fact:** Discord/Slack/Teams model communities as containers of channels with role-based permissions; threads hang off a parent message/channel.
- **Fact:** WhatsApp Communities and Channels are documented broadcast/organization features layered on the existing conversation/E2EE foundation.

## Common Architectural Patterns

- **Pattern:** Treat 1:1 as a special case of a room/conversation with exactly two members (e.g., Matrix rooms) to unify the message pipeline.
- **Pattern:** Separate the *conversation* (identity, type, settings) from *membership* (who, what role) and from *messages* (the stream).
- **Pattern:** Model threads/topics as messages that relate to a parent, rather than as separate containers, to avoid pipeline duplication.
- **Pattern:** Broadcast/channel = a conversation where write permission is restricted to publishers and membership is "subscription."

## Alternative Designs

### Alternative A — Separate models per type (DirectChat, Group, Channel)
Distinct aggregates and pipelines per conversation type.

### Alternative B — Unified conversation aggregate with a `type` discriminator + separate membership/role model
One conversation/message pipeline; 1:1 is a fixed-membership two-party conversation; channels are a restricted-write conversation.

### Alternative C — Fully generic "space/room" hierarchy (communities → sub-rooms → threads)
Highly flexible tree of spaces and rooms (Matrix-like spaces, Discord guilds).

## Advantages

- **A:** Type-specific invariants explicit; each pipeline optimizable independently.
- **B:** Single pipeline (simpler sync, storage, protocol); channel-ready; minimal duplication; clean fit with sender-key group encryption.
- **C:** Maximum flexibility for communities/threads; future-proof for rich social features.

## Disadvantages

- **A:** Duplicated messaging logic; harder to keep sync/storage consistent; more surface for bugs.
- **B:** Requires special-casing Direct (immutable membership); channel broadcast semantics still need design.
- **C:** Significant complexity now for features not in scope; heavier authorization; risk of over-engineering an E2EE product.

## Trade-offs

The central trade-off is **uniformity/simplicity (B)** vs. **per-type optimization (A)** vs. **maximal flexibility (C)**. For an E2EE-first product where the priority is a correct, private message pipeline and future channels (not communities/threads at launch), simplicity and channel-readiness dominate. Rich hierarchy (C) is valuable for community platforms (Discord/Slack) but their trade-off — server-readable content and heavy permissioning — conflicts with our INV-01 constraint.

## Security Considerations

- Membership must drive group key/sender-key updates (AD-004/AD-005); the model must emit membership-change events for re-keying.
- Backend stores conversation metadata only (participants, roles, timestamps), never content (INV-01); metadata minimization applies (AD-021 upcoming).
- Channels (broadcast) change the threat model (large audiences, moderation without content access) and should be reserved, not built now.

## Scalability Considerations

- Group fan-out cost grows with membership; a unified model with sender-keys keeps per-message send cost bounded.
- Very large channels/supergroups (Telegram-scale) are a different scaling regime (broadcast fan-out, read-heavy) — deferring channels avoids premature complexity.
- Membership lookups must be cacheable and shardable by conversation.

## Operational Considerations

- Fewer pipelines (B) means fewer operational surfaces to monitor and fewer failure modes.
- Moderation/abuse for E2EE conversations cannot rely on content inspection; operational tooling works on metadata and user reports.

## Mobile Considerations

- A uniform conversation/message model simplifies client caching, sync, and offline behavior across all conversation types.
- Bounded group sizes reduce per-device key and fan-out overhead on constrained devices.

## Backend Considerations

- A single conversation aggregate simplifies command/query slices (CQRS), event handling, and extraction to a Conversations service later.
- Distinct membership sub-model isolates role/permission logic for AD-003 enforcement.

## Database Implications

- `Conversation` table keyed by `ConversationId` with `type`; `Membership` table (ConversationId, UserId, role, joined_at); messages reference `ConversationId` (partitionable by conversation/time — feeds AD-009/AD-033).
- 1:1 lookups benefit from a canonical two-party membership index to find/create the direct conversation deterministically.

## Future Evolution

- Add `Channel` broadcast type (restricted write, subscription membership) when channels graduate from future scope.
- Consider communities (grouping of conversations) and threads (parent-referencing messages) only if the product direction requires them; both fit as extensions of the unified model.

## Recommendation

*(Evidence-based recommendation; not an approval — AD-007 will decide.)* Adopt **Alternative B: a unified `Conversation` aggregate with a `type` discriminator and a separate Membership/Role model, with 1:1 as a fixed-membership two-party conversation and channels reserved as a future restricted-write type.** It best matches the E2EE-first constraints, unifies the pipeline, keeps group encryption efficient, and remains channel-ready without the over-engineering of a full space/room hierarchy. This corroborates the direction proposed in AD-007.

## Open Questions

- Maximum group size at launch (drives sender-key vs. MLS per AD-004)?
- Are broadcast/announcement semantics needed before full channels?
- Do we reserve schema fields now for channels/communities to avoid later migrations?

## References

- WhatsApp Encryption Overview (whitepaper) — documented E2EE and group model.
- Signal Technical documentation / blog — Groups, sender keys, metadata minimization.
- Telegram documentation — chats, supergroups, channels, secret chats.
- Discord / Slack / Microsoft Teams product documentation — channels, threads, roles.
- Matrix specification — rooms and spaces (industry pattern reference).
