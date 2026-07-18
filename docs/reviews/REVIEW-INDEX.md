# Review Index

> Entry point for reviewers. Start here, then follow the recommended review sequence. For the machine-readable dashboard, open `review-manifest.yaml`. The Git repository is the source of truth.

**Current Build:** #9 — 2026-07-18 (Messaging Core — Synchronization) | **Branch:** feature/messaging-core-ad-010
**Prior Builds:** #8 (AD-009), #7 (AD-008), #6..#1

> Build #9 completes Topic 4 Synchronization and the **Messaging Core Architecture Sprint** (AD-007..AD-010). **Await human approval before the next architecture phase.**

---

## Current Project Status

- **Phase:** Messaging Core Architecture Sprint — **complete** (pending human final confirmation).
- **Decision coverage:** 10 of 50 Approved; 0 Under Review in messaging spine; 40 Proposed.
- **AD-010:** Approved — ConversationSequence cursor delta sync + SignalR (RS-004 B).
- **Defining constraints upheld:** INV-01, INV-04, INV-05.

## Messaging Core Outcome

| Decision | Status |
|---|---|
| AD-007 Conversation Model | Approved |
| AD-008 Message Model | Approved |
| AD-009 Message Ordering | Approved |
| AD-010 Synchronization | Approved |

## Recommended Review Sequence

1. RS-004 → WS-010 → AR-010 → AD-010 → ADR-0016 → ADR-0011
2. DOC-024 §12 / overview §4.4 / glossary
3. CROSS-REFERENCE-REPORT Build #9
4. Human confirmation → next architecture phase

## Open Questions

| ID | Description | Owner |
|---|---|---|
| OQ-SYNC-01 | New-device backfill window | Product |
| OQ-SYNC-04 | Offline ciphertext retention TTL | Product + SRE |
