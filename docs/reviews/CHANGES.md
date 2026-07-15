# Changes

> Append-only. Each build adds a new section at the top listing files created or updated in that build.

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
