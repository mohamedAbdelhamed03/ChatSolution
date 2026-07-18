# Review Index

> Entry point for reviewers. Start here, then follow the recommended review sequence. For the machine-readable dashboard, open `review-manifest.yaml`. The Git repository is the source of truth.

**Phase 3 Status:** **COMPLETE** — [DOC-155](../20-architecture/20.2-phase-3-messaging-core-completion.md)  
**Phase 4 Status:** **IN PROGRESS** — Sprint 1 (AD-042) — [DOC-156](../20-architecture/20.3-phase-4-messaging-services-plan.md)  
**Messaging Core entry point:** [DOC-154](../20-architecture/20.1-messaging-core-architecture.md)

---

## Current Project Status

- **Phase 3:** Closed. AD-007..AD-010 Approved. Baseline established.
- **Phase 4:** In progress. Sprint 1: AR-042 approved; AD-042 Under Review.
- **Decision gate:** Approve AD-042 → ratify ADR-0018 + realtime/protocol docs.
- **Decision coverage:** 10 of 54 Approved (AD-001..AD-010); AD-051..AD-054 newly catalogued Proposed.

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

## Sprint 1 Artifacts (AD-042)

| Artifact | Path | Status |
|---|---|---|
| RS-005 | [research/RS-005-delivery-and-acknowledgements.md](../research/RS-005-delivery-and-acknowledgements.md) | Complete |
| WS-042 | [workshops/WS-042-delivery-and-acknowledgements.md](../architecture-decisions/workshops/WS-042-delivery-and-acknowledgements.md) | Complete |
| AR-042 | [reviews/AR-042-delivery-and-acknowledgements.md](../architecture-decisions/reviews/AR-042-delivery-and-acknowledgements.md) | Approved |
| AD-042 | [AD-042-delivery-semantics.md](../architecture-decisions/AD-042-delivery-semantics.md) | Under Review |

## Recommended Reading

1. DOC-154 → DOC-155 → DOC-156
2. RS-005 → WS-042 → AR-042 → AD-042
3. Approve AD-042 to authorize ADR-0018 ratification
