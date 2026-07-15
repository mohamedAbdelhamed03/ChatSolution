# AD-007 — Conversation Model (Decision Recommendation)

## Question

How should conversations (one-to-one, group, and future channels) be modeled so that membership, roles, and message association are consistent while content remains E2EE?

---

## Background

**Business context:** Conversations are the container for all messaging. The model must support 1:1 and groups now and channels later, with clear membership and roles for authorization (AD-003).

**Technical context:** The backend manages conversation metadata (participants, roles, timestamps) but never content. Group encryption (AD-020) depends on stable membership; sync (AD-010) depends on a stable conversation identity.

**Constraints:** Backend never decrypts (INV-01); metadata minimization (AD-021); authorization over membership (AD-003); extractable modules (AP-09).

---

## Requirements

- **Functional:** Create 1:1 and group conversations; manage membership and roles; associate messages with a conversation; support future channels.
- **Non-Functional:** Stable conversation identity; efficient membership lookups; scalable to large groups.
- **Security:** Membership changes drive key/sender-key updates; only members can access conversation metadata.
- **Scalability:** Membership storage and fan-out scale with group size.

---

## Alternatives

### Alternative A — Unified Conversation aggregate with a `type` (Direct | Group | Channel)
- **Pros:** One consistent model and code path; messages always attach to a `ConversationId`; easy to add channel type; simplifies sync and storage.
- **Cons:** Some type-specific rules (1:1 immutable membership vs. group dynamic) need conditional handling.

### Alternative B — Separate models per type (DirectChat, Group, Channel)
- **Pros:** Each type optimized independently; explicit invariants per type.
- **Cons:** Duplicated logic across messaging paths; harder sync/storage; more surface to keep consistent.

### Alternative C — Unified conversation + separate membership/role sub-model, with 1:1 as a special-cased group
- **Pros:** Single message path; clean membership/role handling; 1:1 is a 2-member conversation with fixed membership; extensible to channels (broadcast membership).
- **Cons:** Must special-case 1:1 semantics (no adding members) to avoid confusing UX.

---

## Industry Research

- **Documented/informed pattern:** Many messaging systems treat 1:1 as a special case of a conversation/room with exactly two participants (e.g., Matrix rooms), which unifies the message pipeline.
- **Discord (documented):** Distinguishes DMs, group DMs, and guild channels, but all are message containers with membership/permissions.
- **Slack/Teams (informed):** Channels and DMs share an underlying "conversation" concept with differing membership/permission rules.

---

## Recommendation

**Recommend Alternative C:** a **unified `Conversation` aggregate** with a `type` (Direct | Group | Channel-future) and a **separate Membership/Role sub-model**, where **1:1 is a fixed-membership two-party conversation**. Messages always reference a stable `ConversationId`. Membership changes emit domain events that drive group sender-key updates (AD-020) and authorization cache invalidation (AD-003).

**Why:** A single conversation/message pipeline minimizes complexity and keeps sync, storage, and protocol uniform, while the membership sub-model cleanly expresses roles and future channels. Modeling 1:1 as a constrained conversation avoids duplicate code paths.

**Trade-offs:** Requires explicit constraints for Direct type (immutable membership); channel broadcast semantics deferred but reserved.

**Risks:** Over-generalization could blur type-specific rules (mitigated by explicit invariants per type in AD-026).

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** One consistent pipeline; clean roles; channel-ready; simpler sync/storage.
- **Negative:** Type conditionals; channel semantics still to be designed.
- **Future Impact:** Anchors AD-008 (Message Model), AD-020 (group encryption), AD-025 (bounded contexts), and future channels (FS-03).

---

## Affected Documents

- `docs/70-features/FS-01-one-to-one-chat.md`
- `docs/70-features/FS-02-group-chat.md`
- `docs/30-domain/31-bounded-contexts-and-modules.md`

## Affected ADRs

- ADR-0020 (Group Encryption)

## Affected Modules

- Conversations

## Open Questions

- Maximum group size for launch (drives fan-out and sender-key strategy)?
- Are group-DMs distinct from named groups at launch?
- Channel membership/broadcast semantics (future) — reserve which fields now?

## Approval

- **Status:** Under Review
- **Owner:** Architecture
- **Review Date:** (pending)
- **Decision Date:** (pending)
