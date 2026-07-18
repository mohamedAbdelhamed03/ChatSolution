# ADR-0031 — Unified Conversation Model

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-07-18 |
| **Amended** | 2026-07-18 |
| **Deciders** | Architecture |
| **Related ADRP** | [AD-007 Conversation Model](../architecture-decisions/AD-007-conversation-model.md) |
| **Related Research** | [RS-001 Conversation Models](../research/RS-001-conversation-models.md) |
| **Workshop / Review** | [WS-007](../architecture-decisions/workshops/WS-007-conversation-model.md), [AR-007](../architecture-decisions/reviews/AR-007-conversation-model.md) |

## Context

The messaging engine needs a single definition of the container that owns membership, authorization context, and the message stream. The platform must support encrypted one-to-one and group chat at launch, remain channel-ready, and uphold INV-01 (backend never decrypts content). Research (RS-001) and Architecture Decision AD-007 selected a unified model after workshop and critical review. Finalization amendments make lifecycles, invariants, ownership, and extensibility explicit without changing the design.

## Decision

Adopt a **unified `Conversation` aggregate** with:

1. Opaque immutable `ConversationId`.
2. Type discriminator: `Direct` | `Group` | `Channel` (Channel not creatable until FS-03).
3. **Membership** entities inside the aggregate boundary, keyed by `(ConversationId, UserId)`, with `MembershipState` and `Role`.
4. Value objects: `ConversationMetadata`, `ConversationSettings`, and Direct `CanonicalPairKey`.
5. **Direct:** exactly two `Active` members; immutable membership set; unique canonical pair; no owner/admin hierarchy.
6. **Group:** dynamic membership; roles `owner` | `admin` | `moderator` | `member`; ≥1 active owner; last owner cannot leave without transfer; membership/lifecycle events are durable.
7. Membership is **per user**; devices inherit authorization (AD-006).
8. Every message references exactly one `ConversationId` (AD-008); messages are **outside** this aggregate.
9. Conversation lifecycle: `Created` → `Active` ⇄ `Archived` / `Frozen` → `Deleted` (terminal). Normative rules in AD-007.
10. Membership lifecycle: `Invited` / `Pending` / `Active` / `Left` / `Removed` / `Blocked` with effects on authz, notifications, key rotation, sync, and read models (AD-007).
11. Future types/features extend via additive `type`/settings/role packs or wrapper aggregates — never a second message pipeline.

Normative detail for state machines, invariants (with D/A/DB enforcement), and ownership matrices: **AD-007** and **DOC-024**.

## Consequences

### Positive

- One message, sync, and storage path for all conversation types.
- Clear control plane for AD-003 authorization and future ADR-0020 group encryption.
- Deterministic 1:1 conversations under concurrent creation.
- Channels and other future concepts can be introduced without redesigning the messaging core.
- Implementers have explicit lifecycles and invariants.

### Negative

- Type-specific invariants and dual lifecycles must be tested continuously.
- Channel product rules remain deferred until FS-03.
- Invite/`Pending` states exist even if launch collapses invite UX.

### Neutral

- Threads are message relations (AD-008), not nested conversations.
- Communities are wrapper aggregates referencing `ConversationId`s.

## Alternatives Considered

| Alternative | Reason not chosen |
|---|---|
| Separate Direct/Group/Channel aggregates and pipelines | Duplicates sync/storage/protocol; higher defect and ops cost (RS-001 Alternative A). |
| Generic space/room hierarchy | Premature complexity for launch FR scope; authz explosion (RS-001 Alternative C). |

## References

- AD-007 Conversation Model (Approved, v2.1 amendments)
- RS-001 Conversation Models
- AD-001 User Identity, AD-003 Authorization, AD-004 E2EE Protocol, AD-006 Device Model
- `docs/30-domain/30-domain-model-overview.md`
- Future: ADR-0020 Group Encryption
