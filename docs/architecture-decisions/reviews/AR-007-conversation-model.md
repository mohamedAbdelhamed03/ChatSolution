# AR-007 — Architecture Review: Conversation Model

| Field | Value |
|---|---|
| **Review ID** | AR-007 |
| **Topic** | Conversation Model |
| **Status** | Completed |
| **Reviewer** | Chief Software Architect (Architecture Review Board) |
| **Reviewed Artifact** | [WS-007](../workshops/WS-007-conversation-model.md) |
| **Evidence** | [RS-001](../../research/RS-001-conversation-models.md) |
| **Decision Under Review** | [AD-007](../AD-007-conversation-model.md) |
| **Date** | 2026-07-18 |
| **Verdict** | Approve with Changes |

> Assumption for this review: every workshop recommendation is wrong until it survives critical challenge.

---

## 1. Review Scope

Challenge WS-007's recommendation of Alternative B (unified Conversation + Membership/Role; Direct as fixed two-party conversation). Validate against AD-001..AD-006, INV-01, FR-004/005/006, and distributed-systems realities of multi-device E2EE messaging.

---

## 2. Challenge Findings

### 2.1 Hidden assumptions

| Assumption | Challenge | Outcome |
|---|---|---|
| "Unified type discriminator is enough" | Type-specific invariants can rot into scattered `if (type)` without a documented invariant table | **Required change:** AD-007 must list per-type invariants (Direct/Group/Channel-reserved) as normative |
| "1:1 = two-member group" | UX and crypto differ: no admin role, no leave-and-rejoin, deterministic pair identity | **Required change:** Direct is a constrained conversation type, not "just a group of 2"; document pair-canonicalization |
| "Channel can be reserved with zero schema cost" | Reserved enum without product rules invites accidental Channel creates | **Required change:** Channel not creatable until FS-03; reject unknown/future types in API |
| "Membership events always reach crypto layer" | Outbox lag or consumer failure leaves stale sender-keys | **Accepted risk** with mitigation: membership change → durable outbox → re-key before further group sends (ties AD-020); non-blocking for model choice |

### 2.2 Security weaknesses

| Issue | Severity | Disposition |
|---|---|---|
| Former member retains sender-key after remove | High (crypto, not model) | Model must emit ordered membership events; crypto rotation owned by AD-020 — **call out explicitly in AD-007** |
| Plaintext group titles on server | Medium | Leave as open question OQ-CONV-04; default for launch: **minimal plaintext display name allowed**, encrypted topic optional later |
| Enumeration of conversation membership by non-members | High | Authz: only members (or authorized system roles) read membership — already AD-003; **restate in AD-007** |
| Direct conversation IDOR via guessed ConversationId | Medium | Opaque ConversationId (ULID) + membership check on every access |

### 2.3 Distributed systems issues

| Issue | Disposition |
|---|---|
| Concurrent Direct create race | **Required:** unique constraint on canonical user-pair key + idempotent create command |
| Multi-device membership view lag | Eventual consistency acceptable if authz cache invalidation is event-driven (AD-003) |
| Split-brain on conversation settings | Conversation aggregate is single-writer per conversation (module owns writes); no CRDT needed for membership at launch |
| Cross-region sequence vs. membership | Out of scope for AD-007; ordering is AD-009 |

### 2.4 Performance bottlenecks

| Issue | Disposition |
|---|---|
| Large group membership fan-out | Bound via OQ-CONV-01; model supports bound without change |
| Hot membership checks | Cache by ConversationId; invalidate on membership events |
| Listing all conversations for a user | Secondary index / projection `Membership by UserId` — document as required read model, not optional |

### 2.5 Operational complexity

Alternative B wins. Alternative A doubles runbooks. Alternative C multiplies permission debugging. **No change** to recommendation.

### 2.6 Recovery / offline / multi-device

| Issue | Disposition |
|---|---|
| Offline client with stale membership sends to group | Server rejects if not member at receive time; client must sync membership — **document** |
| Device linked after join sees history | Gated by AD-006 device trust + AD-010 backfill; conversation model provides ConversationId only |
| User leaves on one device | Membership is user-scoped, not device-scoped — **clarify in AD-007** (devices inherit user's membership) |

### 2.7 Future migration risks

| Risk | Disposition |
|---|---|
| Need communities later | Can layer a Community aggregate referencing ConversationIds without splitting message stream — acceptable |
| Need threads as containers | Prefer message relations (RS-002) over nested conversations — note in Future Evolution |
| Wrong choice of separate Direct tables | Would force dual-write migration — confirms rejecting A |

---

## 3. Alternative Re-evaluation

| Alternative | Survives review? | Notes |
|---|---|---|
| A Separate models | No | Sync/storage/protocol duplication unacceptable under INV-05/AD-010 trajectory |
| B Unified + Membership | **Yes, with changes** | Align lettering with RS-001; add invariants, pair key, Channel gate |
| C Hierarchy | No | Violates launch simplicity; conflicts with E2EE ops model |

---

## 4. Required Changes to AD-007

1. Cite **RS-001** explicitly (Related Research); normalize alternatives to RS-001 lettering (A/B/C).
2. State **Decision** with per-type invariants table.
3. Mandate **canonical Direct pair key** and uniqueness.
4. Clarify membership is **per UserId** (devices inherit).
5. Gate **Channel** creation until FS-03.
6. Require **membership domain events** for authz invalidation and future sender-key rotation.
7. Require **user-conversation list projection** via membership-by-user.
8. Expand sections: Decision Drivers, Risks, Mitigations, Future Evolution, Related Research/ADRs/Documents.
9. Resolve lettering inconsistency with draft (old "Alternative C" → RS-001 **Alternative B**).

---

## 5. Non-Blocking Follow-ups

- OQ-CONV-01 max group size
- OQ-CONV-02 group-DM vs named group
- OQ-CONV-03 Channel schema reservation depth
- OQ-CONV-04 display-name encryption posture
- OQ-CONV-05 membership retention for abuse

---

## 6. Verdict

**Approve with Changes.**

Alternative B is the correct foundation for the messaging engine. The workshop recommendation stands after applying the required changes above. AD-007 may proceed to ratification once those changes are applied in the decision document.

**Quality scores** — Architecture 9 · Security 9 · Scalability 9 · Maintainability 9 · Documentation 9 · **Overall 9.2**

---

## 7. Sign-off

| Role | Name / Function | Result |
|---|---|---|
| Principal Software Architect | Architecture Review Board | Approve with Changes |
| Security Architect | Architecture Review Board | Approve with Changes (membership events mandatory) |
| Distributed Systems Engineer | Architecture Review Board | Approve with Changes (pair uniqueness + projections) |
| Product Architect | Architecture Review Board | Approve with Changes (Channel gated) |
