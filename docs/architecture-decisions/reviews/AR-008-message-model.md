# AR-008 — Architecture Review: Message Model

| Field | Value |
|---|---|
| **Review ID** | AR-008 |
| **Topic** | Message Model |
| **Status** | Completed |
| **Reviewer** | Chief Software Architect (Architecture Review Board) |
| **Reviewed Artifact** | [WS-008](../workshops/WS-008-message-model.md) |
| **Evidence** | [RS-002](../../research/RS-002-message-models.md) |
| **Decision Under Review** | AD-008 |
| **Date** | 2026-07-18 |
| **Verdict** | Approve with Changes |

> Every recommendation is assumed wrong until it survives critical challenge.

---

## 1. Review Scope

Challenge WS-008’s recommendation of RS-002 Alternative B (immutable Message aggregate + bounded envelope + relations) against AD-001..AD-007, INV-01/02/04/12, multi-device sync, and storage/ops realities.

---

## 2. Challenge Findings

### 2.1 Hidden assumptions

| Assumption | Challenge | Outcome |
|---|---|---|
| "`Edited` is a lifecycle state" | Mixing delivery UX states with persistence states confuses clients | **Required:** Separate **persistence states** (`Accepted`, `Tombstoned`, `Expired`) from **version facet** (`editVersion`) and **delivery projections** (`Delivered`/`Read`) |
| "ULID alone orders messages" | Clock skew / offline breaks total order | **Required:** MessageId ≠ total order; `Sequence` mandatory field reserved for AD-009 |
| "Reactions as either messages or rows" left open | Ambiguity blocks schema | **Required for AD-008:** Launch = **relation rows** with **encrypted reaction payload**; not cleartext emoji metadata |
| "ForwardedFrom id is optional" | Privacy leak if stored carelessly | **Required:** Default **flag-only** `isForwarded`; attribution ids deferred (OQ-MSG-04) |

### 2.2 Distributed systems / sync

| Issue | Disposition |
|---|---|
| Concurrent edits from two devices | Last-accept-wins by server `editVersion` / server time; clients reconcile via sync events — document in AD-008 |
| Pending offline duplicate | Idempotent upsert on `MessageId` |
| Tombstone vs late edit | Reject edits when tombstoned |
| Gap detection | Relies on AD-009 Sequence — model must not invent a second total-order field |

### 2.3 Security

| Issue | Disposition |
|---|---|
| Reply preview / reaction emoji in metadata | Forbidden plaintext — ciphertext only |
| Attachment content-type leakage | Store coarse or encrypted; size may be metadata (AD-021) — bound fields in AD-008 |
| System messages | Still ciphertext or well-known opaque type codes without content |
| Delete-for-everyone false sense of security | Document E2EE limitation explicitly |

### 2.4 Storage / performance

| Issue | Disposition |
|---|---|
| Edit versions unbounded growth | Launch: retain **latest ciphertext + editVersion + edited flag**; older versions purgeable (OQ-MSG-03 default: latest-only) |
| Receipts on Message row | **Forbidden** — separate projection (confirm in AD-008) |
| Reaction storms | Separate table; no lock on Message ciphertext row |

### 2.5 Operational complexity

Retention/purge must preserve Sequence gaps policy (tombstones remain until purge horizon). **Accept** with documented purge rules pointer to future retention AD.

### 2.6 Multi-device edge cases

| Case | Rule |
|---|---|
| Device A edits, Device B offline | Sync applies `MessageEdited` by MessageId + editVersion |
| Sender revoked mid-Pending | Accept rejects if membership/device invalid |
| Link device history | Backfill ciphertext per AD-006/AD-010; model unchanged |

### 2.7 Migration risks

Avoid JSON-only schemaless server types without `MessageType` enum — breaks indexing. Additive enum + encrypted payload version is enough.

### 2.8 Aggregate boundary vs Conversation

WS-008 correctly keeps Message outside Conversation. **Confirm** Attachment bytes outside Messaging DB (object storage). Relation records inside Messaging module.

---

## 3. Alternative Re-evaluation

| Alternative | Survives? |
|---|---|
| RS-002 A Mutable | **No** — INV breach |
| RS-002 B Immutable + relations | **Yes, with changes** |
| RS-002 C Full event sourcing | **No for launch** — revisit if audit/compliance mandates |

Envelope alternatives (draft AD-008 A/C): pure-blob insufficient for receipts/relations; prefer explicit bounded envelope (workshop lean confirmed).

---

## 4. Required Changes to AD-008

1. Cite **RS-002**; align alternatives to RS-002 A/B/C (plus envelope clarification).
2. Message = **Aggregate Root** in Messaging; owns relations/attachment refs; references Conversation/User/Device/blobs.
3. **Persistence model:** Accepted message immutable ciphertext; `editVersion`; tombstone/expired flags — do not overload Delivered/Read as Message states.
4. **MessageId** = ULID or UUIDv7 (pick one default: **ULID** at launch, OQ-MSG-05 allows switch to UUIDv7 if stdlib preference); **Sequence** field for AD-009.
5. Reactions = encrypted relation rows at launch.
6. Forward = new message + `isForwarded` only by default.
7. Edit history = latest-only at launch.
8. Invariants table with D/A/DB enforcement.
9. Domain events list; lifecycle Mermaid; aggregate diagram.
10. Explicit non-ownership of ordering algorithm and sync protocol.
11. Alignment with AD-007 membership checks at accept.

---

## 5. Non-Blocking Follow-ups

OQ-MSG-01..05; AD-021 metadata minimization contract; retention ADR; ADR-0008/0010 for ordering algorithm detail.

---

## 6. Verdict

**Approve with Changes.**

RS-002 Alternative B is correct. Apply required changes in AD-008 before ratification.

**Quality scores** — Architecture 9 · Security 10 · Scalability 9 · Maintainability 9 · Documentation 9 · **Overall 9.3**

---

## 7. Sign-off

| Role | Result |
|---|---|
| Principal Software Architect | Approve with Changes |
| Security Architect | Approve with Changes (no plaintext sub-content; forward flag-only default) |
| Distributed Systems Engineer | Approve with Changes (Sequence vs MessageId; receipt projections) |
| DDD / Messaging Specialist | Approve with Changes (Message aggregate root; persistence vs delivery states) |
