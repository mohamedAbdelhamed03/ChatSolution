# AD-001 — User Identity (Decision Recommendation)

## Question

How should a user be identified within the platform, and what is the canonical identity that authentication, cryptographic identity, and all domain data are anchored to?

---

## Background

**Business context:** A global messaging platform must let people find and message each other while honoring privacy. Identity is the anchor for accounts, devices, keys, and every conversation. The choice affects onboarding friction, privacy (contact discovery), and account recovery.

**Technical context:** Identity must support multi-device, E2EE (each identity owns published public keys), and stateless horizontally scaled services. The backend never sees message content, so identity is one of the few pieces of first-class server-side data.

**Constraints:** Backend never decrypts content (INV-01); every external request authenticated (INV-08); metadata minimization (NFR-SEC-05); millions of users (NFR-S-01).

---

## Requirements

- **Functional:** Uniquely identify a user; allow login from multiple devices; support a human-facing handle for discovery; allow account recovery.
- **Non-Functional:** Globally unique, stable, and immutable internal identifier; scalable lookup.
- **Security:** Identifier must not leak personal data; contact discovery must be privacy-preserving; identity anchors public-key material.
- **Scalability:** Constant-time identity resolution at millions of users.

---

## Alternatives

### Alternative A — Phone number as primary identity
- **Pros:** Frictionless discovery (address book); proven onboarding (WhatsApp/Signal); natural spam resistance (SIM cost).
- **Cons:** Privacy exposure of phone numbers; SIM-swap risk; excludes users without phones; ties identity to a mutable external asset.

### Alternative B — Username / handle as primary identity
- **Pros:** No PII required; discovery by handle; user-controlled; works without a phone.
- **Cons:** Weaker default discovery from contacts; handle squatting; needs separate anti-abuse.

### Alternative C — Opaque internal UserId (UUID/ULID) with pluggable identifiers (phone and/or username) layered on top
- **Pros:** Internal identity is stable, immutable, PII-free, and privacy-safe; external identifiers (phone, username, email) are attributes that can change without breaking references; supports multiple discovery strategies; best fit for E2EE anchoring and metadata minimization.
- **Cons:** More moving parts; requires an identifier-to-UserId resolution layer and privacy-preserving discovery design.

---

## Industry Research

- **WhatsApp (documented):** Uses phone number as the primary identifier and the Signal Protocol for E2EE.
- **Signal (documented):** Historically phone-number based; has publicly introduced usernames/phone-number privacy to reduce phone exposure.
- **Telegram (documented):** Phone-number registration with optional public usernames; cloud chats are not E2EE by default (secret chats are).
- **Discord (documented):** Username/handle-based identity with no phone requirement to message; not E2EE for messages.
- **Informed pattern:** Mature systems separate a stable **internal ID** from **external identifiers** (phone/username/email) to allow the latter to change and to support privacy-preserving discovery. This is an architectural pattern, not a single vendor's documented internal schema.

---

## Recommendation

**Recommend Alternative C:** an opaque, immutable internal `UserId` (ULID/UUID) as the canonical identity, with **username as the primary user-facing identifier** and **optional verified phone/email** as attributes usable for discovery.

**Why:** It anchors E2EE public keys and all domain references to a stable, PII-free identifier (privacy by design, metadata minimization), while keeping onboarding and discovery flexible. External identifiers can change (number port, username change) without breaking cryptographic identity or message history.

**Trade-offs:** Requires a resolution/discovery layer and explicit anti-abuse for username-based signup (mitigated by AD-002 authentication choices and later abuse decisions). Slightly higher initial complexity than phone-only.

**Risks:** Discovery UX weaker than address-book phone matching; username squatting; recovery flows must not weaken E2EE guarantees.

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Stable identity for keys and data; privacy-preserving; flexible discovery; clean multi-device foundation.
- **Negative:** Extra resolution layer; discovery and anti-abuse need dedicated design.
- **Future Impact:** Enables privacy-preserving contact discovery (FS-11), painless number/username changes, and clean microservice extraction of the Identity context.

---

## Affected Documents

- `docs/40-security/42-authn-authz.md`
- `docs/40-security/43-key-management-and-devices.md`
- `docs/30-domain/30-domain-model-overview.md`
- `docs/70-features/FS-11-contact-discovery-and-blocking.md`

## Affected ADRs

- ADR-0019 (Multi-Device)

## Affected Modules

- Identity

## Open Questions

- Is phone verification required at launch, or optional for discovery/anti-abuse?
- What is the account-recovery model, and how does it interact with E2EE key loss?
- What are username policy and squatting-prevention rules?

## Review Outcome (2026-07-15)

**Reviewer:** Chief Software Architect · **Verdict:** Approve with Changes

**Required changes applied:**
- **Account recovery model added:** recovery relies on client-side end-to-end encrypted backup with a user-held recovery secret (see AD-005). Loss of the secret loses message history; the server cannot recover content (INV-01). This must be communicated clearly in onboarding.
- **Launch posture clarified:** username is the primary user-facing identity; phone/email are optional verified attributes used solely for discovery and anti-abuse, never as the cryptographic anchor.
- **Username lifecycle rule added:** a released username enters a cool-down period before it can be reused, to reduce impersonation and takeover of prior conversations.

**Residual open questions (owner follow-up, non-blocking):** whether phone verification is mandatory at launch (Product); detailed squatting/anti-abuse policy (Trust & Safety).

**Quality scores** — Architecture 9 · Security 9 · Scalability 9 · Maintainability 9 · Documentation 9 · **Overall 9.2**

## Approval

- **Status:** Approved
- **Owner:** Architecture
- **Reviewed by:** Chief Software Architect (Architecture Decision Review Sprint)
- **Review Date:** 2026-07-15
- **Decision Date:** 2026-07-15
