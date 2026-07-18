# ADR-0031 — Unified Conversation Model

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-007 Conversation Model](../architecture-decisions/AD-007-conversation-model.md) |
| **Related Research** | [RS-001 Conversation Models](../research/RS-001-conversation-models.md) |
| **Workshop / Review** | [WS-007](../architecture-decisions/workshops/WS-007-conversation-model.md), [AR-007](../architecture-decisions/reviews/AR-007-conversation-model.md) |

## Context

The messaging engine needs a single definition of the container that owns membership, authorization context, and the message stream. The platform must support encrypted one-to-one and group chat at launch, remain channel-ready, and uphold INV-01 (backend never decrypts content). Research (RS-001) and Architecture Decision AD-007 selected a unified model after workshop and critical review.

## Decision

Adopt a **unified `Conversation` aggregate** with:

1. Opaque immutable `ConversationId`.
2. Type discriminator: `Direct` | `Group` | `Channel` (Channel not creatable until FS-03).
3. Separate **Membership** records keyed by `(ConversationId, UserId)` with roles for groups.
4. **Direct** conversations: exactly two members, immutable membership, identity via canonical sorted UserId pair with uniqueness and idempotent create.
5. **Group** conversations: dynamic membership; join/leave/role-change emit durable domain events for authz invalidation and group re-keying.
6. Membership is **per user**; devices inherit membership for authorization (device crypto fan-out remains per AD-006).
7. Every message references exactly one `ConversationId` (message representation: AD-008).

## Consequences

### Positive

- One message, sync, and storage path for all conversation types.
- Clear control plane for AD-003 authorization and future ADR-0020 group encryption.
- Deterministic 1:1 conversations under concurrent creation.
- Channels can be introduced as a type/permission change, not a new pipeline.

### Negative

- Type-specific invariants must be enforced in the Conversations module and tested.
- Channel semantics remain undefined until FS-03.
- Group display metadata privacy posture is a follow-up (OQ-CONV-04).

### Neutral

- Threads are not conversations; they are message relations (deferred to AD-008).
- Communities may later reference conversations without splitting the stream.

## Alternatives Considered

| Alternative | Reason not chosen |
|---|---|
| Separate Direct/Group/Channel aggregates and pipelines | Duplicates sync/storage/protocol; higher defect and ops cost (RS-001 Alternative A). |
| Generic space/room hierarchy | Premature complexity for launch FR scope; authz explosion (RS-001 Alternative C). |

## References

- AD-007 Conversation Model (Approved)
- RS-001 Conversation Models
- AD-001 User Identity, AD-003 Authorization, AD-004 E2EE Protocol, AD-006 Device Model
- `docs/30-domain/30-domain-model-overview.md`
- Future: ADR-0020 Group Encryption
