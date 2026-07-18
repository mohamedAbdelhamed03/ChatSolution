# Architecture Review Checklist

> Reviewer checklist. Build #2 adds the Architecture Decision Repository (ADRP) section below. Items for not-yet-generated documents remain unchecked until their build.

## Architecture Decisions (ADRP — Build #2, current review scope)

- [x] ADRP README complete (purpose, workflow, lifecycle)
- [x] decision-manifest.yaml consistent (50 decisions, valid dependency graph)
- [x] AD-001 User Identity — reviewed, improved, **Approved**
- [x] AD-002 Authentication — reviewed, improved, **Approved**
- [x] AD-003 Authorization — reviewed, improved, **Approved**
- [x] AD-004 E2EE Protocol — reviewed, improved, **Approved** (direction; library spike pending)
- [x] AD-005 Key Management — reviewed, improved, **Approved**
- [x] AD-006 Device Model and Device Trust — reviewed, improved, **Approved**
- [x] AD-007 Conversation Model — workshop WS-007, review AR-007, **Approved**, ADR-0031 Accepted, **finalization amendments** (lifecycle/invariants/ownership/extensibility/diagrams)
- [x] AD-008 Message Model — workshop WS-008, review AR-008, **Approved**, ADR-0032 Accepted
- [x] AD-009 Message Ordering — workshop WS-009, review AR-009, **Approved**, ADR-0008/0010 Accepted
- [x] AD-010 Synchronization — workshop WS-010, review AR-010, **Approved**, ADR-0016/0011 Accepted
- [ ] Pending decisions (AD-011..AD-050) catalogued with valid dependencies

## Product

- [ ] Vision complete (DOC-001)
- [ ] Personas complete (DOC-009)
- [ ] Functional requirements complete (DOC-010)
- [ ] NFR complete (DOC-011)
- [ ] Requirements traceability matrix (DOC-012) — *pending*

## Architecture

- [ ] Architecture Overview (DOC-013)
- [ ] Architecture Principles (DOC-022)
- [ ] System Invariants (DOC-023)
- [ ] C4 Context (DOC-014) — *pending*
- [ ] C4 Container (DOC-015) — *pending*
- [ ] Modular Monolith Blueprint (DOC-017) — *pending*
- [ ] Vertical Slice & CQRS (DOC-018) — *pending*
- [ ] Event-Driven Architecture (DOC-019) — *pending*

## Foundation

- [ ] README & reading order (DOC-002)
- [ ] Glossary overview (DOC-003)
- [ ] Glossary term files (DOC-004..008) — *pending*

## Domain

- [x] Domain model overview (DOC-024) — Conversation/Membership (Build #5); message/sync pending AD-008..010
- [ ] Bounded contexts & modules (DOC-025) — *pending*
- [ ] Domain events catalog (DOC-027) — *pending*
- [ ] Message lifecycle (DOC-045) — *pending*
- [ ] Sequences SQ-01..SQ-15 (DOC-030..044) — *pending*

## Security

- [ ] E2EE respected across all completed documents
- [ ] HTTPS only stated and enforced (INV-09)
- [ ] WSS only for realtime (INV-09)
- [ ] Backend never decrypts (INV-01)
- [ ] Metadata minimization acknowledged (NFR-SEC-05)
- [ ] No plaintext in telemetry (INV-11)
- [ ] Security Architecture (DOC-046) — *pending*
- [ ] E2EE Design (DOC-047) — *pending*
- [ ] Threat Model (DOC-050) — *pending*

## Data

- [ ] Data architecture (DOC-056) — *pending*
- [ ] Postgres schema & migrations (DOC-057) — *pending*
- [ ] Read model & Dapper (DOC-058) — *pending*
- [ ] Redis usage (DOC-059) — *pending*

## Realtime & Protocol

- [ ] Realtime architecture (DOC-064) — *pending*
- [ ] Protocol overview (DOC-086) — *pending*
- [ ] Encryption envelope (DOC-090) — *pending*

## Quality & Operations

- [ ] Testing strategy (DOC-104) — *pending*
- [ ] Performance budget (DOC-106) — *pending*
- [ ] Definition of Done (DOC-107) — *pending*
- [ ] SLO/SLI (DOC-110) — *pending*

## Consistency

- [ ] Cross references valid (see CROSS-REFERENCE-REPORT.md)
- [ ] No contradictions between completed documents
- [ ] Mermaid diagrams render
- [ ] Dependency graph respected (matches document-manifest.yaml)
- [ ] Naming conventions followed
- [ ] Every completed document has all mandatory sections

## Governance

- [ ] Pending ADRs reviewed (see ARCHITECTURE-DECISIONS-PENDING.md)
- [ ] Open questions triaged and assigned
- [ ] Quality scores acceptable (see QUALITY-REPORT.md)
