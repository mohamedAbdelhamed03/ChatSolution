# Domain Model Overview

| Field | Value |
|---|---|
| **Title** | Domain Model Overview |
| **Status** | Completed |
| **Owner** | Architecture |
| **Version** | 1.2.0 |
| **Last Updated** | 2026-07-18 |
| **Document ID** | DOC-024 |

**Dependencies:** `00-glossary-overview` (DOC-003), `12-functional-requirements` (DOC-010), `20-architecture-overview` (DOC-013), ADR-0031 (DOC-152), ADR-0032 (DOC-153).

**Related Documents:** `31-bounded-contexts-and-modules` (DOC-025), `32-aggregates-and-invariants` (DOC-026), `33-domain-events-catalog` (DOC-027), AD-007, AD-008, ADR-0031, ADR-0032, RS-001, RS-002.

---

## Purpose

This document defines the ubiquitous domain model for the messaging platform: core aggregates, identities, lifecycles, invariants, and relationships that all feature specs and sequences must respect. **Conversation** is ratified by AD-007 / ADR-0031. **Message** is ratified by AD-008 / ADR-0032. Ordering algorithms and sync protocol remain AD-009 / AD-010.

## Scope

**In scope:** Conversation aggregate; Message aggregate (ciphertext envelope, relations, attachment refs, persistence lifecycle); domain events; terminology.

**Out of scope (pending AD-009..AD-010):** Total-order algorithm details, sync cursor protocol, full delivery state machine implementation.

## Architecture Impact

Conversation + Message aggregates form the messaging engine core. Messages reference conversations by ID only. Immutable messages with relations uphold INV-01/02/12 and enable multi-device sync.

---

## 1. Bounded Context Sketch

```mermaid
flowchart LR
    Identity[Identity Context\n User, Device, Keys]
    Conversations[Conversations Context\n Conversation aggregate]
    Messaging[Messaging Context\n Message - AD-008]
    Identity -->|UserId| Conversations
    Conversations -->|ConversationId| Messaging
```

| Context | Owns | Key IDs |
|---|---|---|
| Identity | User, Device, public-key material | `UserId`, `DeviceId` |
| Conversations | Conversation aggregate (Membership, Metadata, Settings) | `ConversationId` |
| Messaging | Message aggregate, relations, attachment refs | `MessageId` |

No cross-module table access (INV-06). Messages are **not** inside the Conversation aggregate.

---

## 2. Aggregate Structure

**Root:** `Conversation`  
**Entities:** `Membership`  
**Value objects:** `Role`, `ConversationMetadata`, `ConversationSettings`, `CanonicalPairKey` (Direct)  
**External references:** `UserId`, `MessageId` (outside boundary)

```mermaid
classDiagram
    direction TB
    class Conversation {
        <<aggregate root>>
        +ConversationId id
        +ConversationType type
        +LifecycleState state
        +DateTime createdAt
    }
    class Membership {
        <<entity>>
        +UserId userId
        +MembershipState state
        +Role role
    }
    class Role {
        <<value object>>
        +RoleKind kind
    }
    class ConversationMetadata {
        <<value object>>
    }
    class ConversationSettings {
        <<value object>>
        +int maxMembers
    }
    Conversation "1" *-- "2..*" Membership
    Membership *-- Role
    Conversation *-- ConversationMetadata
    Conversation *-- ConversationSettings
```

Normative diagrams and field-level rules: AD-007 § Aggregate Structure.

---

## 3. Conversation Lifecycle

States: `Created` → `Active` ⇄ `Archived` / `Frozen` → `Deleted` (terminal).

```mermaid
stateDiagram-v2
    [*] --> Created: CreateConversation
    Created --> Active: Activate
    Active --> Archived: Archive
    Archived --> Active: Unarchive
    Active --> Frozen: Freeze
    Frozen --> Active: Unfreeze
    Active --> Deleted: SoftDelete
    Archived --> Deleted: SoftDelete
    Frozen --> Deleted: SoftDelete
    Deleted --> [*]
```

| State | Messaging | Notes |
|---|---|---|
| `Created` | No | Transient; launch activates in same transaction |
| `Active` | Yes | Subject to membership |
| `Archived` | No | Retained history |
| `Frozen` | No | Group/system; **not used for Direct at launch** |
| `Deleted` | No | Terminal soft-delete; id never reused |

**State ownership:** Conversations domain aggregate. Full transition table: AD-007 § Conversation Lifecycle.

---

## 4. Membership Lifecycle

States: `Invited` → `Pending` → `Active` → `Left` | `Removed` | `Blocked`.

```mermaid
stateDiagram-v2
    [*] --> Invited: InviteMember
    Invited --> Pending: AcceptInvite
    Invited --> Active: AcceptInviteDirect
    Pending --> Active: ConfirmMembership
    Active --> Left: Leave
    Active --> Removed: RemoveMember
    Active --> Blocked: BlockInConversation
    Left --> Invited: Reinvite
    Removed --> Invited: Reinvite
    Blocked --> Active: UnblockInConversation
```

| Type | Rules |
|---|---|
| Direct | Both members created `Active`; add/remove/`Left`/`Removed` **invalid** |
| Group | Full machine; launch may skip to `Active` on add |
| Channel | Future; subscription + publisher roles |

### Cross-cutting effects

| Concern | On membership change |
|---|---|
| Authorization | Only `Active` (+ role); cache invalidated via events |
| Notifications | Metadata-only fan-out to affected devices |
| Key rotation | Loss of `Active` triggers sender-key rotation before further group sends |
| Synchronization | Membership is control-plane sync data |
| Read models | Conversation list / membership-by-user updated from events |

---

## 5. Domain Invariants

Enforcement: **D** domain · **A** application · **DB** database.

### Direct

- Exactly two distinct participants with `Active` membership (D+DB).
- One conversation per canonical pair key (D+A+**DB UNIQUE**).
- No additional members; no owner/admin hierarchy (D).
- `Freeze` rejected at launch (D+A).

### Group

- ≥1 `Active` `owner` at all times (D).
- Last owner cannot leave/be removed without ownership transfer (D).
- `Active`+`Invited`+`Pending` count ≤ `maxMembers` (D+A).
- Roles ∈ {owner, admin, moderator, member} (D+DB).

### Channel (reserved)

- Not creatable until FS-03 (D+A).
- When enabled: only publishers create messages; subscribers read-only unless elevated (D+A).

Full invariant IDs (D-INV-*, G-INV-*, C-INV-*, X-INV-*): **AD-007**.

---

## 6. Ownership Model

| Role | Direct | Group |
|---|---|---|
| Owner | N/A (peers) | Ultimate authority; soft-delete; transfer |
| Admin | N/A | Manage members/settings |
| Moderator | N/A | Limited moderation (metadata actions) |
| Member | Both parties | Default participant |

**Ownership transfer (Group):** atomic command; former owner demoted; G-INV-01 held throughout.  
**Direct:** no ownership transfer; either peer may soft-delete per launch rule (OQ-CONV-06).

Authority matrix: AD-007 § Ownership Model.

---

## 7. Domain Events (Conversations)

| Event | When |
|---|---|
| `ConversationCreated` / `ConversationActivated` | Create/activate |
| `ConversationArchived` / `ConversationUnarchived` | Archive toggles |
| `ConversationFrozen` / `ConversationUnfrozen` | Freeze toggles |
| `ConversationDeleted` | Soft-delete |
| `MemberInvited` / `MemberJoined` | Invite / active |
| `MemberLeft` / `MemberRemoved` / `MemberBlocked` | Exit paths |
| `MemberRoleChanged` | Role / ownership transfer |
| `ConversationMetadataUpdated` | Non-content metadata |

Events are immutable (INV-03). Payloads: IDs, roles, states only — never plaintext content.

---

## 8. Sequences

### Create or Resolve Direct Conversation

```mermaid
sequenceDiagram
    participant C as Client Device
    participant API as Conversations API
    participant DB as Conversations Store
    C->>API: CreateDirect(peerUserId) [authn]
    API->>API: Authorize; canonicalize pair
    API->>DB: INSERT Conversation Active + 2 Membership Active OR fetch by pair key
    alt Exists
        DB-->>API: Existing ConversationId
    else Created
        API-->>API: ConversationCreated + Activated
    end
    API-->>C: ConversationId + membership snapshot
```

### Group Membership Change

```mermaid
sequenceDiagram
    participant Admin as Admin Device
    participant API as Conversations API
    participant DB as Conversations Store
    participant Bus as Outbox
    Admin->>API: AddMember(conversationId, userId)
    API->>API: Authorize admin/owner; reject if Direct; enforce maxMembers
    API->>DB: Membership Active or Invited
    API->>Bus: MemberJoined or MemberInvited
    Note over Bus: Authz invalidate; sync; future key rotation
    API-->>Admin: OK
```

---

## 9. Future Extensibility

| Concept | Strategy |
|---|---|
| Channels | Enable `type=Channel` + publish/subscribe roles |
| Communities | Wrapper aggregate → many `ConversationId`s |
| Threads / Topics | Message relations (AD-008), not new conversations |
| Business / AI / Support / Temporary | Additive settings + role packs |
| Broadcast Lists | Prefer Channel or client fan-out to Directs |

Never split the message pipeline by type. Details: AD-007 § Future Extensibility Strategy.

---

## 10. Message Aggregate (AD-008 / ADR-0032)

### 10.1 Structure

**Root:** `Message`  
**Owns:** ciphertext (current `editVersion`), envelope metadata, `AttachmentRef`s, reaction/pin relations, tombstone/expiry.  
**References:** `ConversationId`, `SenderUserId`, `SenderDeviceId`, blob ids.  
**Does not own:** Conversation, delivery/read receipts (projections), sync cursors, raw media bytes.

```mermaid
classDiagram
    class Message {
        <<aggregate root>>
        +MessageId id
        +ConversationId conversationId
        +long sequence
        +int editVersion
        +bytes ciphertext
        +bool tombstoned
    }
    class AttachmentRef
    class Reaction
    Message *-- AttachmentRef
    Message *-- Reaction
```

**Identity:** client-generated ULID `MessageId`. **`Sequence`:** per-conversation server order (algorithm: AD-009).

### 10.2 Persistence lifecycle

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Accepted: ServerAccept
    Pending --> Rejected
    Accepted --> Accepted: Edit
    Accepted --> Tombstoned: DeleteOrRecall
    Accepted --> Expired: TTL
```

Delivered/Read are **receipt projections**, not Message states.

### 10.3 Invariants (summary)

- Exactly one Conversation; one sender; immutable MessageId (M-INV-01..03).
- Ciphertext replaced only via `editVersion` (M-INV-04).
- Reply target same conversation (M-INV-05).
- Accept requires Active membership (M-INV-06).
- No plaintext content columns (M-INV-07).
- Tombstone clears ciphertext (M-INV-08).

Full table: AD-008.

### 10.4 Relations & mutations

| Feature | Model |
|---|---|
| Edit | New ciphertext + editVersion++; latest-only at launch |
| Recall/Delete | Tombstone |
| Reply | `replyToMessageId` |
| Reaction | Relation + encrypted emoji |
| Forward | New message + `isForwarded` |
| Attachment | Encrypted blob ref |

### 10.5 Sequence — Accept message

```mermaid
sequenceDiagram
    participant C as Sender Device
    participant API as Messaging API
    participant Conv as Conversations Authz
    participant DB as Message Store
    C->>API: Send(MessageId, ConversationId, ciphertext, envelope)
    API->>Conv: Assert Active membership
    Conv-->>API: OK
    API->>DB: Idempotent insert by MessageId; assign Sequence
    API-->>API: MessageAccepted
    API-->>C: Ack(Sequence, serverReceivedAt)
```

### 10.6 Message domain events

`MessageAccepted`, `MessageEdited`, `MessageRecalled`/`MessageTombstoned`, `MessageExpired`, `ReactionAdded`/`Removed`, `MessagePinned`/`Unpinned`, `AttachmentReferenced`.

---

## 11. Terminology

| Term | Meaning |
|---|---|
| Conversation | Aggregate root; typed Direct/Group/Channel; has lifecycle state |
| Membership | Entity binding `UserId` to Conversation with state and role |
| Role | owner / admin / moderator / member (Group); peers (Direct) |
| Message | Messaging aggregate root; immutable id; ciphertext + envelope |
| MessageId | Client-generated ULID; immutable global identity |
| Sequence | Per-conversation server ordering field (AD-009) |
| Tombstone | Soft-deleted message placeholder retaining id/sequence |
| AttachmentRef | Pointer to encrypted blob in object storage |
| Receipt projection | Delivered/Read state outside Message aggregate |
| Canonical Pair Key | Sorted `(UserId, UserId)` uniqueness for Direct |

Aligned with `00-glossary-overview`.

---

## Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Domain model drifts from AD-007/AD-008 | Inconsistent implementation | ADR-0031/0032 normative |
| Sequence treated as MessageId | Ordering bugs | Explicit field separation |
| Channel created early | Unsupported scale | Creation gated until FS-03 |

## Future Considerations

- DOC-026 / DOC-027 for invariants and event schemas.
- Expand sync aggregates when AD-010 approves.

## Open Questions

| ID | Question | Owner |
|---|---|---|
| OQ-CONV-01 | Max group size (`maxMembers`) | Product + Security |
| OQ-CONV-04 | Group title/avatar encryption | Security + Product |
| OQ-MSG-01 | Edit / delete-for-everyone windows | Product |
| OQ-MSG-05 | ULID vs UUIDv7 | Architecture |

## References

- AD-007 / ADR-0031 / RS-001
- AD-008 / ADR-0032 / RS-002
- AD-001, AD-003, AD-004, AD-006
- `20-architecture-overview` (DOC-013)
- `29.5-system-invariants` (DOC-023)
