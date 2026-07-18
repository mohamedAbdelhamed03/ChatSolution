# Review Index

> Entry point for reviewers. Start here, then follow the recommended review sequence. For the machine-readable dashboard, open `review-manifest.yaml`. The Git repository is the source of truth.

**Current Build:** #6 — 2026-07-18 (AD-007 Finalization Amendments) | **Branch:** feature/messaging-core-ad-007
**Prior Builds:** #5 (AD-007 Approved), #4 (Research), #3 (AD-001..006), #2, #1

> Build #6 strengthens AD-007 documentation (lifecycles, invariants, ownership, extensibility, aggregate diagrams) **without changing the approved design**. Confirm finalization, then approve starting Topic 2 (AD-008).

---

## Current Project Status

- **Phase:** Messaging Core — AD-007 finalization amendments complete; awaiting human confirmation before AD-008.
- **Decision coverage:** 7 of 50 Approved; 3 Under Review (AD-008..AD-010); 40 Proposed.
- **AD-007:** Approved + finalization amendments (v2.1); ADR-0031 Accepted.
- **Defining constraint upheld:** Backend never decrypts message content (INV-01).

## Sprint Outcome (Build #6)

| Item | Result |
|---|---|
| Architectural direction | Unchanged (Alternative B) |
| Mandatory amendments 1–6 | Complete |
| Consistency validation | PASS |
| New Architecture Decision | None |

## Review Scope (this build)

Documentation-only amendments to AD-007 and related artifacts listed in `CHANGES.md` Build #6.

## Recommended Review Sequence

1. `AD-007` (v2.1) — lifecycle, membership, invariants, ownership, extensibility, diagrams
2. `ADR-0031` / `DOC-024` / overview §4.1 / glossary
3. `AR-007` §8 verification table
4. `CROSS-REFERENCE-REPORT.md` Build #6
5. Human confirmation → proceed to AD-008 Message Model

## Critical Documents

| Document | Why critical |
|---|---|
| AD-007 v2.1 | Normative conversation architecture including amendments |
| ADR-0031 | Ratified ADR |
| DOC-024 | Domain model lifecycles and invariants |
| RS-001 | Evidence base |

## Open Questions (non-blocking)

| ID | Description | Owner |
|---|---|---|
| OQ-CONV-01 | Max group size | Product + Security |
| OQ-CONV-04 | Title/avatar encryption | Security + Product |
| OQ-CONV-06 | Direct delete vs per-user hide | Product |
| OQ-CONV-07 | Launch invite UX | Product |
