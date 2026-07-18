# Changes

> Append-only. Each build adds a new section at the top listing files created or updated in that build.

---

## Build #5 — 2026-07-18 (Messaging Core — Conversation Model)

**Branch:** feature/messaging-core-ad-007

| File Path | Change | Reason | Related Documents |
|---|---|---|---|
| docs/architecture-decisions/workshops/WS-007-conversation-model.md | Created | Workshop preparing AD-007; no final decision. | RS-001, AD-007 |
| docs/architecture-decisions/reviews/AR-007-conversation-model.md | Created | Critical review; Approve with Changes. | WS-007, AD-007 |
| docs/architecture-decisions/AD-007-conversation-model.md | Updated | Cite RS-001; apply AR-007 changes; status Approved. | RS-001, ADR-0031 |
| docs/ADR/adr-template.md | Created | Formal ADR template (DOC-120). | — |
| docs/ADR/ADR-0031-unified-conversation-model.md | Created | Ratify unified Conversation model. | AD-007, DOC-024 |
| docs/30-domain/30-domain-model-overview.md | Created | Domain model with Conversation/Membership + sequences. | AD-007, ADR-0031 |
| docs/20-architecture/20-architecture-overview.md | Updated | Add §4.1 Conversation Model. | AD-007, DOC-024 |
| docs/00-glossary/00-glossary-overview.md | Updated | Conversation + Membership definitions. | DOC-024 |
| docs/architecture-decisions/decision-manifest.yaml | Updated | AD-007 Approved; ADR-0031 linked. | — |
| docs/architecture-decisions/README.md | Updated | Workshop/review paths; sprint progress. | — |
| docs/document-manifest.yaml | Updated | DOC-024/120/152 Completed; DOC-152 added. | — |
| docs/reviews/BUILD-REPORT.md | Updated | Append Build #5. | — |
| docs/reviews/CHANGES.md | Updated | Append Build #5. | — |
| docs/reviews/QUALITY-REPORT.md | Updated | AD-007 review scores. | — |
| docs/reviews/CROSS-REFERENCE-REPORT.md | Updated | Build #5 validation. | — |
| docs/reviews/REVIEW-CHECKLIST.md | Updated | Check off AD-007. | — |
| docs/reviews/REVIEW-INDEX.md | Updated | Entry point for Build #5. | — |
| docs/reviews/review-manifest.yaml | Updated | Regenerate dashboard for Build #5. | — |
| docs/reviews/ARCHITECTURE-DECISIONS-PENDING.md | Updated | Note ADR-0031 accepted. | — |

### Notes

- No code changes. AD-008..AD-010 untouched. Sprint pauses for human approval before Message Model.

---

## Build #4 — 2026-07-15 (Architecture Research Sprint)

**Branch:** feature/research-sprint-2

| File Path | Change | Reason | Related Documents |
|---|---|---|---|
| docs/research/README.md | Created | Establish the Research Repository and its relationship to decisions/ADRs/docs. | RS-001..RS-004 |
| docs/research/RS-001-conversation-models.md | Created | Evidence for conversation model decision. | AD-007 |
| docs/research/RS-002-message-models.md | Created | Evidence for message model decision. | AD-008 |
| docs/research/RS-003-message-ordering.md | Created | Evidence for ordering/ID decision. | AD-009 |
| docs/research/RS-004-synchronization.md | Created | Evidence for synchronization decision. | AD-010 |
| docs/architecture-decisions/decision-manifest.yaml | Updated | Add related_research links (AD-007..010 → RS-001..004). | RS-001..RS-004 |
| docs/reviews/review-manifest.yaml | Updated | Add research documents to next review scope; Build #4. | — |
| docs/reviews/BUILD-REPORT.md | Updated | Append Build #4 section. | — |
| docs/reviews/CHANGES.md | Updated | Append Build #4 section. | — |
| docs/reviews/REVIEW-INDEX.md | Updated | Refresh entry point to research sprint. | — |

### Notes

- No files removed; no Architecture Decisions authored or approved; no ADRs, documentation, or code generated.
- Research clearly distinguishes documented facts, industry patterns, and project recommendations, and respects constraints AD-001..AD-006.

---

## Build #3 — 2026-07-15 (Architecture Decision Review Sprint)

**Branch:** feature/architecture-decisions-wave-1

| File Path | Change | Reason | Related Documents |
|---|---|---|---|
| docs/architecture-decisions/AD-001-user-identity.md | Updated | Add Review Outcome; recovery model, launch posture, username lifecycle; status → Approved. | AD-005 |
| docs/architecture-decisions/AD-002-authentication.md | Updated | Add Review Outcome; replay protection, token binding, revocation bound; status → Approved. | AD-006, OPS-05 |
| docs/architecture-decisions/AD-003-authorization.md | Updated | Add Review Outcome; uniform enforcement, event-driven cache invalidation; status → Approved. | INV-07 |
| docs/architecture-decisions/AD-004-e2ee-protocol.md | Updated | Add Review Outcome; MLS trigger criteria, library spike, metadata caveat; status → Approved (direction). | ADR-0004, AD-021 |
| docs/architecture-decisions/AD-005-key-management.md | Updated | Add Review Outcome; prekey exhaustion fallback, key-injection defense, encrypted backup; status → Approved. | AD-006 |
| docs/architecture-decisions/AD-006-device-model.md | Updated | Add Review Outcome; all-devices-lost recovery, fan-out bound, mandatory notifications; status → Approved. | AD-002, AD-005 |
| docs/architecture-decisions/decision-manifest.yaml | Updated | Set AD-001..AD-006 status Approved with review/decision dates. | — |
| docs/reviews/BUILD-REPORT.md | Updated | Append Build #3 review sprint section. | — |
| docs/reviews/CHANGES.md | Updated | Append Build #3 section. | — |
| docs/reviews/QUALITY-REPORT.md | Updated | Add Build #3 review verdicts and scores. | — |
| docs/reviews/CROSS-REFERENCE-REPORT.md | Updated | Add Build #3 post-approval validation. | — |
| docs/reviews/REVIEW-CHECKLIST.md | Updated | Check off approved AD-001..AD-006. | — |
| docs/reviews/REVIEW-INDEX.md | Updated | Refresh entry point to review sprint outcome. | — |
| docs/reviews/review-manifest.yaml | Updated | Regenerate dashboard for Build #3. | — |

### Notes

- No files removed; no new decisions, ADRs, documentation, or code generated.
- All six decisions reached Approved via reviewed, justified changes; INV-01 upheld throughout.

---

## Build #2 — 2026-07-15

**Branch:** feature/architecture-decisions-wave-1 | **Base:** main (5305ba6)

| File Path | Change | Reason | Related Documents |
|---|---|---|---|
| docs/architecture-decisions/README.md | Created | Establish the ADRP: purpose, workflow, decision lifecycle, and relationship to ADRs and documentation. | decision-manifest.yaml |
| docs/architecture-decisions/decision-manifest.yaml | Created | Single source of truth cataloguing all 50 architectural decisions with dependency order and status. | All AD-* documents |
| docs/architecture-decisions/AD-001-user-identity.md | Created | Recommend canonical user identity model. | 42-authn-authz, 30-domain-model-overview |
| docs/architecture-decisions/AD-002-authentication.md | Created | Recommend authentication and session establishment approach. | 42-authn-authz, SQ-02 |
| docs/architecture-decisions/AD-003-authorization.md | Created | Recommend authorization model over metadata. | 42-authn-authz, FS-02 |
| docs/architecture-decisions/AD-004-e2ee-protocol.md | Created | Recommend E2EE protocol upholding backend-never-decrypts. | 41-e2ee-design, 85.4-encryption-envelope |
| docs/architecture-decisions/AD-005-key-management.md | Created | Recommend key lifecycle with blind public-key directory. | 43-key-management-and-devices |
| docs/architecture-decisions/AD-006-device-model.md | Created | Recommend multi-device model and device trust. | 43-key-management-and-devices, SQ-03 |
| docs/architecture-decisions/AD-007-conversation-model.md | Created | Recommend unified conversation model. | FS-01, FS-02, 31-bounded-contexts |
| docs/architecture-decisions/AD-008-message-model.md | Created | Recommend message representation (ciphertext + bounded metadata). | 36-message-lifecycle, 85.6-message-metadata |
| docs/architecture-decisions/AD-009-message-ordering.md | Created | Recommend deterministic ordering and ID scheme. | 54-message-sync-and-storage, ADR-0008 |
| docs/architecture-decisions/AD-010-synchronization.md | Created | Recommend cursor-based multi-device synchronization. | 85.9-synchronization-protocol |
| docs/reviews/BUILD-REPORT.md | Updated | Append Build #2 section. | — |
| docs/reviews/CHANGES.md | Updated | Append Build #2 section. | — |
| docs/reviews/REVIEW-CHECKLIST.md | Updated | Add ADRP review section. | — |
| docs/reviews/QUALITY-REPORT.md | Updated | Add Build #2 decision quality assessment. | — |
| docs/reviews/CROSS-REFERENCE-REPORT.md | Updated | Add Build #2 ADRP validation. | — |
| docs/reviews/REVIEW-INDEX.md | Updated | Refresh entry point to current build. | — |
| docs/reviews/review-manifest.yaml | Updated | Regenerate review dashboard for Build #2. | — |

### Notes

- No files were removed.
- No implementation code, ADRs, or documentation were generated in this build (Architecture Decision Sprint only).
- All decision recommendations uphold the invariant that the backend never decrypts message content (INV-01).

---

## Build #1 — 2026-07-14

| File Path | Change | Reason | Related Documents |
|---|---|---|---|
| docs/document-manifest.yaml | Created | Establish the machine-readable single source of truth for document identity, status, and dependency order. | All documents |
| docs/00-README.md | Created | Provide the documentation entry point, standards, folder taxonomy, and status model. | document-manifest.yaml, DOC-001 |
| docs/00-glossary/00-glossary-overview.md | Created | Establish the ubiquitous language index and core cross-cutting terms (plaintext, ciphertext, envelope, metadata). | DOC-004..008, DOC-024 |
| docs/10-product/10-vision-and-scope.md | Created (prior build, registered) | Root product intent, goals, scope, and constraints. | DOC-013, DOC-047 |
| docs/10-product/11-personas-and-use-cases.md | Created | Define personas and end-to-end use cases that requirements trace to. | DOC-010, sequences |
| docs/10-product/12-functional-requirements.md | Created | Enumerate testable functional requirements mapped to use cases and feature slices. | DOC-009, DOC-069 |
| docs/10-product/13-non-functional-requirements.md | Created | Define measurable quality attributes driving architecture, budgets, and SLOs. | DOC-021, DOC-106 |
| docs/20-architecture/20-architecture-overview.md | Created | Consolidate requirements and constraints into the solution strategy. | DOC-022, DOC-023, C4 docs |
| docs/20-architecture/29-architecture-principles.md | Created | Define the non-negotiable design principles and their enforcement. | DOC-023, DOC-046 |
| docs/20-architecture/29.5-system-invariants.md | Created | Define runtime invariants (incl. backend-never-decrypts) with enforcement and verification. | DOC-047, DOC-099, DOC-107 |

### Notes

- No files were removed.
- No implementation code was created or modified in this build.
- All content-handling documents were authored under the invariant that the backend never decrypts message content.
