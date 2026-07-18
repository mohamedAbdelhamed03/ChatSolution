# Review Index

> Entry point for reviewers. Start here, then follow the recommended review sequence. For the machine-readable dashboard, open `review-manifest.yaml`. The Git repository is the source of truth.

**Current Build:** #8 — 2026-07-18 (Messaging Core — Message Ordering) | **Branch:** feature/messaging-core-ad-009
**Prior Builds:** #7 (AD-008), #6 (AD-007 amendments), #5, #4, #3, #2, #1

> Build #8 completes Topic 3 Message Ordering (WS-009 → AR-009 → AD-009 Approved → ADR-0008/0010). **Await human approval before Topic 4 (AD-010 Synchronization).**

---

## Current Project Status

- **Phase:** Messaging Core — Topics 1–3 complete; Topic 4 paused for human approval.
- **Decision coverage:** 9 of 50 Approved; 1 Under Review (AD-010); 40 Proposed.
- **AD-009:** Approved — per-conversation server Sequence + ULID MessageId (RS-003 C).
- **Defining constraint upheld:** Backend never decrypts; INV-05 ordering satisfied.

## Sprint Outcome (Build #8)

| Decision | Verdict | Status |
|---|---|---|
| AD-009 Message Ordering | Approve with Changes | Approved |

## Recommended Review Sequence

1. RS-003 → WS-009 → AR-009 → AD-009 → ADR-0008 → ADR-0010
2. DOC-024 §11 / overview §4.3 / glossary
3. CROSS-REFERENCE-REPORT Build #8
4. Human approval → AD-010 Synchronization

## Open Questions

| ID | Description | Owner |
|---|---|---|
| OQ-ORD-01 | Counter implementation pattern | Backend |
| OQ-ORD-03 | HLC adoption triggers | Architecture + SRE |
