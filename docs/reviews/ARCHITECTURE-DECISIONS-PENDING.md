# Pending Architecture Decisions

> Decisions not yet finalized. Each is catalogued as an ADR document in `document-manifest.yaml` (status Pending). None currently blocks documentation generation, but the flagged ones must be resolved before their dependent documents can be promoted from Draft to Approved.

### Recently Accepted

| ADR | Title | Date | Ratifies |
|---|---|---|---|
| ADR-0031 | Unified Conversation Model | 2026-07-18 | AD-007 |
| ADR-0032 | Immutable Message Model | 2026-07-18 | AD-008 |
| ADR-0008 | Message ID and Ordering Strategy | 2026-07-18 | AD-009 (+ AD-008 identity) |
| ADR-0010 | Per-Conversation Message Ordering | 2026-07-18 | AD-009 |
| ADR-0011 | Cursor Pagination | 2026-07-18 | AD-010 |
| ADR-0016 | Offline Synchronization | 2026-07-18 | AD-010 |

---

## DEC-01 — E2EE Protocol / Library (ADR-0004)

| Field | Value |
|---|---|
| **Decision ID** | ADR-0004 |
| **Why it matters** | Defines the cryptographic foundation that realizes the core promise (backend never decrypts). Determines key exchange, message encryption, group encryption, and multi-device behavior. |
| **Dependent documents** | DOC-047 (E2EE Design), DOC-049 (Key Management), DOC-090 (Encryption Envelope), DOC-085 (Client SDK Crypto Contract), DOC-108 (Crypto Conformance), DOC-140 (Group Encryption). |
| **Current assumptions** | A double-ratchet-style protocol with X3DH-style asynchronous key agreement and sender-keys for groups; server acts as a blind key directory. |
| **Possible alternatives** | (a) Signal Protocol library; (b) MLS (Messaging Layer Security) for groups; (c) custom protocol (not recommended). |
| **Recommendation** | Adopt an audited implementation of the Signal Protocol for 1:1 and evaluate MLS for large-group scaling; avoid custom cryptography. |

## DEC-02 — Global Message ID & Ordering (ADR-0008, ADR-0010) — **RESOLVED 2026-07-18**

| Field | Value |
|---|---|
| **Decision ID** | ADR-0008 / ADR-0010 |
| **Status** | **Accepted** via AD-009 |
| **Decision** | ULID `MessageId` + per-conversation server `Sequence`; HLC-ready for multi-region; no client-timestamp order. |
| **Residual** | OQ-ORD-01 counter implementation; OQ-ORD-03 HLC adoption triggers. |

## DEC-03 — Group Encryption Model (ADR-0020)

| Field | Value |
|---|---|
| **Decision ID** | ADR-0020 |
| **Why it matters** | Determines how group messages scale while remaining E2EE, affecting fan-out cost. |
| **Dependent documents** | DOC-071 (Group Chat), DOC-035 (Group Message sequence), DOC-047, DOC-090. |
| **Current assumptions** | Sender-keys per group with per-device key distribution. |
| **Possible alternatives** | (a) Sender-keys; (b) pairwise encryption per member; (c) MLS group. |
| **Recommendation** | Sender-keys for medium groups; evaluate MLS for very large groups (ties to DEC-01). |

## DEC-04 — Search Engine for Metadata (ADR-0009, ADR-0028)

| Field | Value |
|---|---|
| **Decision ID** | ADR-0009 / ADR-0028 |
| **Why it matters** | Content is not server-searchable (E2EE); server search is metadata-only. The engine choice affects operability and cost. |
| **Dependent documents** | DOC-061 (Search Architecture), DOC-079 (FS-10 Search). |
| **Current assumptions** | PostgreSQL full-text search over non-encrypted metadata initially; client-side search for content. |
| **Possible alternatives** | (a) PostgreSQL FTS; (b) OpenSearch/Elasticsearch for metadata scale. |
| **Recommendation** | Start with PostgreSQL FTS for metadata; revisit a dedicated engine only if metadata-search scale demands it. |

## DEC-05 — Multi-Region & Data Residency (relates to ADR-0016, DOC-103)

| Field | Value |
|---|---|
| **Decision ID** | (architecture decision feeding DOC-103) |
| **Why it matters** | Determines global latency, availability, ordering strategy, and legal residency compliance. |
| **Dependent documents** | DOC-103 (Multi-Region & Residency), DOC-062 (Partitioning & Sharding), DOC-051 (Privacy & Compliance). |
| **Current assumptions** | Single-region launch; multi-region deferred. |
| **Possible alternatives** | (a) Single region at launch; (b) multi-region active-active from day one. |
| **Recommendation** | Launch single-region with a documented extraction/expansion path; defer active-active until scale/residency requires it. |

---

## Decision Status Summary

| Decision | ADR(s) | Blocks generation? | Must resolve before |
|---|---|---|---|
| DEC-01 E2EE Protocol | ADR-0004 | No | DOC-047 Approved |
| DEC-02 Message ID/Ordering | ADR-0008/0010 | **Resolved** | Accepted 2026-07-18 |
| DEC-03 Group Encryption | ADR-0020 | No | DOC-071 Approved |
| DEC-04 Search Engine | ADR-0009/0028 | No | DOC-061 Approved |
| DEC-05 Multi-Region | DOC-103 | No | DOC-103 Approved |
