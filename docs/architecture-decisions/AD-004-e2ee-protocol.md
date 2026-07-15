# AD-004 — E2EE Protocol (Decision Recommendation)

## Question

Which end-to-end encryption protocol should the platform adopt for one-to-one and group messaging so that the backend never decrypts content while supporting multi-device and asynchronous delivery?

---

## Background

**Business context:** Confidentiality is the core product promise. The operator must be technically incapable of reading message content. This is the single most consequential architectural decision.

**Technical context:** Encryption happens only on clients; the backend stores/routes ciphertext + metadata. The protocol must support asynchronous (offline) message setup, forward secrecy, post-compromise security, multiple devices per user, and groups.

**Constraints:** Backend never decrypts (INV-01, absolute); metadata minimization (AD-021); multi-device (AD-006); millions of users. Do not invent custom cryptography.

---

## Requirements

- **Functional:** Encrypt/decrypt 1:1 and group messages on clients; asynchronous session setup (recipient offline); multi-device delivery.
- **Non-Functional:** Efficient at scale; small per-message overhead; well-defined envelope for the server to route (AD-021).
- **Security:** Forward secrecy; post-compromise security (healing); authenticated encryption; deniability where appropriate; verifiable identity (safety numbers).
- **Scalability:** Group encryption that does not grow cost linearly with membership at message-send time where avoidable.

---

## Alternatives

### Alternative A — Signal Protocol (X3DH + Double Ratchet), sender-keys for groups
- **Pros:** Battle-tested, audited, and documented; strong forward secrecy and post-compromise security; asynchronous setup via prekeys; widely deployed at billions-of-users scale; sender-keys make group fan-out efficient.
- **Cons:** Group membership changes require sender-key rotation handling; multi-device requires per-device sessions (pairwise) or sesame-style device management.

### Alternative B — MLS (Messaging Layer Security, RFC 9420)
- **Pros:** Standardized (IETF); designed for large groups with efficient membership changes (TreeKEM); strong security properties; future-proof.
- **Cons:** Younger ecosystem/tooling; operational complexity; overkill for 1:1; group-centric design.

### Alternative C — Custom protocol
- **Pros:** Tailored control.
- **Cons:** Extremely high risk; unaudited cryptography is a critical liability; violates "don't roll your own crypto." Not acceptable for a production privacy product.

---

## Industry Research

- **Signal (documented):** Created and uses the Signal Protocol (X3DH + Double Ratchet); open specifications and audits are public.
- **WhatsApp (documented):** Uses the Signal Protocol for E2EE across all messages; whitepaper is public.
- **Google Messages / others (documented):** Adopted the Signal Protocol for RCS E2EE.
- **MLS (documented):** RFC 9420; adopted/evaluated by several vendors for scalable group E2EE.
- **Telegram (documented):** Uses custom MTProto; cloud chats are not E2EE by default — a contrast we explicitly reject given our invariant.
- **Fact vs. pattern:** Signal Protocol adoption by Signal/WhatsApp is documented fact; using sender-keys for group scaling is a documented Signal design.

---

## Recommendation

**Recommend Alternative A:** adopt an **audited implementation of the Signal Protocol** (X3DH for asynchronous key agreement, Double Ratchet for message encryption) with **sender-keys for group messaging**, and treat **MLS as a future evaluation** for very large groups (ties to AD-020, AD-006).

**Why:** It is the most proven, publicly audited option that satisfies all requirements — asynchronous setup, forward secrecy, post-compromise security, multi-device, and efficient group fan-out — while never requiring server-side decryption. It avoids the unacceptable risk of custom cryptography and the ecosystem immaturity of MLS for 1:1 today.

**Trade-offs:** Multi-device and group membership changes add key-management complexity (addressed in AD-005/AD-006/AD-020). MLS may later be preferable for large groups; the envelope (AD-021) should not hard-code protocol assumptions.

**Risks:** Correct integration is critical (use a vetted library, conformance vectors per DOC-108); metadata still requires separate protection (AD-021); backup/restore of history is constrained by E2EE (encrypted backups only).

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Strong, proven confidentiality; satisfies INV-01 by construction; efficient group model.
- **Negative:** Key-management and multi-device complexity; server cannot help recover content.
- **Future Impact:** Defines AD-005 (Key Management), AD-020/AD-022/AD-023 (group/attachment/media encryption), the envelope (`85.4`), and search limits (AD-014).

---

## Affected Documents

- `docs/40-security/41-e2ee-design.md`
- `docs/85-protocol/85.4-encryption-envelope.md`
- `docs/80-api/84-client-sdk-crypto-contract.md`
- `docs/95-quality/98-crypto-test-vectors-and-conformance.md`

## Affected ADRs

- ADR-0004 (E2EE Client-Only Encryption), ADR-0020 (Group Encryption)

## Affected Modules

- Messaging, Identity, Media

## Open Questions

- Which specific vetted library/implementation and its current audit status?
- Sender-keys only, or MLS for large groups from the start?
- Deniability requirements and safety-number verification UX?

## Approval

- **Status:** Under Review
- **Owner:** Security
- **Review Date:** (pending)
- **Decision Date:** (pending)
