# Review Index

> Entry point for reviewers. Start here, then follow the recommended review sequence. For the machine-readable dashboard, open `review-manifest.yaml`. The Git repository is the source of truth.

**Phase 3 Status:** **COMPLETE** — [DOC-155](../20-architecture/20.2-phase-3-messaging-core-completion.md)  
**Phase 4 Status:** **IN PROGRESS** — Sprint 1 (AD-042) — [DOC-156](../20-architecture/20.3-phase-4-messaging-services-plan.md)  
**Messaging Core entry point:** [DOC-154](../20-architecture/20.1-messaging-core-architecture.md)

---

## Current Project Status

- **Phase 3:** Closed. AD-007..AD-010 Approved. Baseline established.
- **Phase 4:** In progress. Sprint 1 complete (AD-042). Sprint 2: RS-006 + WS-044 complete.
- **Decision gate:** Approve WS-044 lean → proceed to AR-044 / AD-044.
- **Decision coverage:** 11 of 54 Approved (AD-001..AD-010, AD-042).

## Messaging Core (baseline)

| Decision | Status |
|---|---|
| AD-007..AD-010 | Approved |

## Phase 4 Sprint Order (authoritative AD IDs)

| Sprint | AD | Topic |
|---|---|---|
| 1 | AD-042 | Delivery & Acknowledgements |
| 2 | AD-044 | Read Receipts |
| 3 | AD-011 / AD-043 | Presence & Typing |
| 4 | AD-051 | Reactions & Interactions |
| 5 | AD-054 | Media & Attachments |
| 6 | AD-012 | Push Notifications |
| 7 | AD-014 | Search & Indexing |
| 8 | AD-052 | Moderation & Safety |
| 9 | AD-038 | Retention & Compliance |
| 10 | AD-053 | Voice & Video Signaling |

Do **not** use retired placeholder labels that conflict with catalog AD-011..AD-020 (see DOC-156).

## Sprint 2 Artifacts (AD-044)

| Artifact | Path | Status |
|---|---|---|
| RS-006 | [research/RS-006-read-receipts.md](../research/RS-006-read-receipts.md) | Complete |
| WS-044 | [workshops/WS-044-read-receipts.md](../architecture-decisions/workshops/WS-044-read-receipts.md) | Awaiting Decision |
| AR-044 / AD-044 | — | Not started |

## Recommended Reading

1. AD-042 + ADR-0018 (Sprint 1 baseline)
2. RS-006 → WS-044
3. Approve workshop lean to authorize AR-044 / AD-044
