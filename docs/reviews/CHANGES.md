# Changes

> Append-only. Each build adds a new section at the top listing files created or updated in that build.

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
