# Review Index

> Entry point for reviewers. Start here, then follow the recommended review sequence. For the machine-readable dashboard, open `review-manifest.yaml`. The Git repository is the source of truth.

**Current Build:** #7 — 2026-07-18 (Messaging Core — Message Model) | **Branch:** feature/messaging-core-ad-008
**Prior Builds:** #6 (AD-007 amendments), #5 (AD-007), #4 (Research), #3, #2, #1

> Build #7 completes Topic 2 Message Model (WS-008 → AR-008 → AD-008 Approved → ADR-0032). **Await human approval before Topic 3 (AD-009 Message Ordering).**

---

## Current Project Status

- **Phase:** Messaging Core — Topics 1–2 complete; Topics 3–4 paused for human approval.
- **Decision coverage:** 8 of 50 Approved; 2 Under Review (AD-009, AD-010); 40 Proposed.
- **AD-008:** Approved — immutable Message + envelope + relations (RS-002 B).
- **Defining constraint upheld:** Backend never decrypts message content (INV-01).

## Sprint Outcome (Build #7)

| Decision | Verdict | Status |
|---|---|---|
| AD-008 Message Model | Approve with Changes | Approved |

Workshop: WS-008 · Review: AR-008 · ADR: ADR-0032 Accepted.

## Recommended Review Sequence

1. RS-002 → WS-008 → AR-008 → AD-008 → ADR-0032
2. DOC-024 §10 / overview §4.2 / glossary
3. CROSS-REFERENCE-REPORT Build #7
4. Human approval → AD-009 Message Ordering

## Critical Documents

| Document | Why critical |
|---|---|
| AD-008 / ADR-0032 | Normative Message Model |
| RS-002 | Evidence |
| AD-007 | Conversation boundary constraint |
| DOC-023 | INV-01/02/12 |

## Open Questions

| ID | Description | Owner |
|---|---|---|
| OQ-MSG-01 | Edit / delete-for-everyone windows | Product |
| OQ-MSG-05 | ULID vs UUIDv7 | Architecture |
| OQ-INV-02 | Ordering scheme detail (AD-009) | Architecture |
