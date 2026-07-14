# Build Report

> Append-only. Each documentation build adds a new section at the top. History is never overwritten.

---

## Build #1 — 2026-07-14

| Field | Value |
|---|---|
| **Build ID** | BUILD-0001 |
| **Build Timestamp** | 2026-07-14T16:59:00+03:00 |
| **Branch** | main |
| **Documented Commit** | cb5c3a4 |
| **Current Phase** | Phase-1 (Foundations) — complete; entering Phase-2 |
| **Build Duration** | ~20 minutes (single build pass) |

### Documents Completed (this build)

| DOC | Title | Path |
|---|---|---|
| DOC-001 | Vision and Scope | docs/10-product/10-vision-and-scope.md |
| DOC-002 | Documentation README and Reading Order | docs/00-README.md |
| DOC-003 | Glossary Overview | docs/00-glossary/00-glossary-overview.md |
| DOC-009 | Personas and Use Cases | docs/10-product/11-personas-and-use-cases.md |
| DOC-010 | Functional Requirements | docs/10-product/12-functional-requirements.md |
| DOC-011 | Non-Functional Requirements | docs/10-product/13-non-functional-requirements.md |
| DOC-013 | Architecture Overview | docs/20-architecture/20-architecture-overview.md |
| DOC-022 | Architecture Principles | docs/20-architecture/29-architecture-principles.md |
| DOC-023 | System Invariants | docs/20-architecture/29.5-system-invariants.md |

### Documents Remaining

- **142 Pending** of 151 catalogued.
- **Next eligible** (all dependencies Completed): DOC-004, DOC-005, DOC-006, DOC-007, DOC-008, DOC-012, DOC-014, DOC-021, DOC-046, DOC-120.

### Manifest Statistics

| Metric | Value |
|---|---|
| Total documents | 151 |
| Completed | 9 |
| Pending | 142 |
| Blocked | 0 |
| Completion | 6.0% |

By category (completed / total):

| Category | Completed | Total |
|---|---|---|
| Foundation | 2 | 7 |
| Product | 3 | 4 |
| Architecture | 3 | 11 |
| Domain | 0 | 22 |
| Security | 0 | 10 |
| Data | 0 | 8 |
| Realtime | 0 | 5 |
| Features | 0 | 12 |
| API | 0 | 5 |
| Protocol | 0 | 10 |
| Infrastructure | 0 | 8 |
| Quality | 0 | 5 |
| Operations | 0 | 11 |
| ADR | 0 | 31 |
| Migration | 0 | 1 |

### Dependency Graph Summary

- Roots (no dependencies): DOC-001, DOC-002, DOC-003, DOC-120.
- Longest resolved chain this build: DOC-001 → DOC-009 → DOC-010 → DOC-013.
- No circular dependencies detected among completed documents.
- All completed-document dependencies are themselves Completed (graph is internally consistent).

### Blocking Documents

- None. No completed document is waiting on a missing dependency.

### Blocking Architectural Decisions

- None blocking generation. Advisory decisions to finalize before dependent docs reach "Approved": ADR-0004 (E2EE protocol), ADR-0008 / ADR-0010 (message ID & ordering). See `ARCHITECTURE-DECISIONS-PENDING.md`.
