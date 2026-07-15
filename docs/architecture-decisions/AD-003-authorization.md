# AD-003 — Authorization (Decision Recommendation)

## Question

How does the platform decide what an authenticated user/device is permitted to do (e.g., read a conversation, post to a group, delete a message), given the backend cannot read content?

---

## Background

**Business context:** Users must only access their own conversations and act within their role (member, admin). Group and future channel roles require differentiated permissions.

**Technical context:** Authorization is enforced server-side over metadata (membership, roles), never over content. It must be consistent across REST and SignalR and cheap to evaluate at scale.

**Constraints:** Deny-by-default (AP-11); backend never decrypts content (INV-01); every request authorized (INV-08); stateless nodes.

---

## Requirements

- **Functional:** Enforce conversation membership; group roles (member/admin/owner); message-level actions (edit/delete own; admin moderation on metadata); rate-limited actions.
- **Non-Functional:** Low-latency checks; centrally auditable policy.
- **Security:** No privilege escalation; membership changes take effect promptly.
- **Scalability:** Authorization data cacheable and shardable.

---

## Alternatives

### Alternative A — Role-Based Access Control (RBAC)
- **Pros:** Simple, well-understood; roles per conversation (member/admin/owner) cover most needs; easy to reason about and audit.
- **Cons:** Coarse-grained; awkward for fine-grained per-action rules; role explosion if over-extended.

### Alternative B — Attribute-Based Access Control (ABAC) / policy engine
- **Pros:** Fine-grained, expressive; adapts to complex future rules (channels, moderation).
- **Cons:** Heavier to implement/operate; harder to audit; risk of over-engineering for launch scope.

### Alternative C — RBAC core + a small set of relationship/resource checks (hybrid)
- **Pros:** RBAC for roles plus explicit membership/ownership checks (is-member, is-owner-of-message) covers messaging cleanly; low latency; extensible toward ABAC later.
- **Cons:** Requires disciplined policy placement in each vertical slice; two mechanisms to keep coherent.

---

## Industry Research

- **Documented pattern:** RBAC is the default for team/'group' permissions across collaboration tools; relationship-based checks (ReBAC, e.g., Google Zanzibar) are the documented approach for large-scale "who can access this resource" systems.
- **Discord (documented):** Rich per-role, per-channel permission bitfields — an RBAC-with-overrides model for large communities.
- **Informed pattern:** Consumer 1:1/group messengers typically rely on simple membership + role checks rather than a full policy engine, reserving richer models for channels/communities.

---

## Recommendation

**Recommend Alternative C:** an **RBAC core** (conversation roles: member, admin, owner) combined with **explicit relationship/resource checks** (is-member-of-conversation, is-author-of-message) enforced within each vertical slice and evaluated over metadata only. Design the policy layer so it can evolve toward ReBAC/ABAC when channels arrive.

**Why:** Matches the messaging domain precisely, keeps checks fast and auditable, honors deny-by-default, and never touches content. It is the least complex option that still scales and extends to future community/channel needs.

**Trade-offs:** Requires consistent enforcement discipline across slices; a future channels model may push toward ReBAC.

**Risks:** Missed check in a slice (mitigated by a shared authorization behavior + architecture/DoD gates); stale membership cache (bounded TTL + invalidation).

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Simple, fast, auditable; content-agnostic; extensible.
- **Negative:** Enforcement spread across slices; will need extension for channels.
- **Future Impact:** Shapes AD-007 (Conversation Model) roles and future channel permissions; feeds the cross-cutting authorization behavior.

---

## Affected Documents

- `docs/40-security/42-authn-authz.md`
- `docs/70-features/FS-02-group-chat.md`
- `docs/20-architecture/27-cross-cutting-concerns.md`

## Affected ADRs

- ADR-0019 (Multi-Device)

## Affected Modules

- Identity, Conversations, Messaging

## Open Questions

- Group role set and permission matrix for launch?
- Are message-level admin actions limited to metadata (pin, remove-for-group) given content is opaque?
- Membership-cache TTL and invalidation strategy?

## Approval

- **Status:** Under Review
- **Owner:** Security
- **Review Date:** (pending)
- **Decision Date:** (pending)
