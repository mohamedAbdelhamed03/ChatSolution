# Architecture Review Checklist

> Reviewer checklist for Build #1. Items for not-yet-generated documents are listed as pending and remain unchecked until their build.

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

- [ ] Domain model overview (DOC-024) — *pending*
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
