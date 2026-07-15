# AD-006 — Device Model and Device Trust (Decision Recommendation)

## Question

How are multiple devices per user modeled, linked, trusted, and revoked, and how does each device participate in E2EE without the backend ever decrypting content?

---

## Background

**Business context:** Users expect to use phone, tablet, and desktop simultaneously with a consistent conversation view. Adding and removing devices must be secure and simple, and a lost device must be revocable.

**Technical context:** Per AD-004/AD-005, each device has its own cryptographic identity and prekeys. Messages are encrypted per recipient device (or via group sender-keys distributed per device). New devices must obtain history without the server decrypting it.

**Constraints:** Backend never decrypts (INV-01); per-device keys (AD-005); authenticated sessions (AD-002); deterministic sync (AD-010).

---

## Requirements

- **Functional:** Register/link a new device; list devices; revoke a device; re-sync history to a new device as ciphertext.
- **Non-Functional:** Efficient per-device fan-out; bounded device count per user.
- **Security:** New-device linking must be authenticated and resistant to unauthorized addition; revocation must promptly cut access; users must be able to verify device changes (safety numbers).
- **Scalability:** Fan-out cost grows with device count; must remain acceptable.

---

## Alternatives

### Alternative A — Primary-device-authorized linking (existing device approves new device)
- **Pros:** Strong trust anchor; user-visible approval; resistant to remote unauthorized additions; documented pattern (Signal/WhatsApp QR linking).
- **Cons:** Requires an existing active device; recovery when all devices lost depends on backup (AD-005).

### Alternative B — Independent per-device registration (each device authenticates to account independently)
- **Pros:** No dependency on an existing device; simplest onboarding for additional devices.
- **Cons:** Weaker trust; easier for an attacker with account credentials to silently add a device; needs strong notifications + verification.

### Alternative C — Hybrid: server-authenticated registration + user-visible device-change notifications + safety-number re-verification
- **Pros:** Works even without an existing device (recovery), while surfacing every device change to the user and enabling contacts to re-verify; balances usability and trust.
- **Cons:** Relies on users heeding notifications; requires robust device-change eventing.

---

## Industry Research

- **Signal/WhatsApp (documented):** Multi-device via primary-device QR linking; each linked device has its own keys; security-number/safety-number changes are surfaced when identity keys change.
- **WhatsApp multi-device (documented):** Companion devices operate with independent identity keys and receive their own encrypted copies; a device list is maintained per user.
- **Informed pattern:** Surfacing "a new device was added" and "safety number changed" to users is a key defense against silent malicious device addition.

---

## Recommendation

**Recommend Alternative C (with A as the preferred linking path):** model each device as a first-class entity with its own keys; **prefer primary-device-authorized linking (QR)** when an active device exists, and allow **server-authenticated registration for recovery**, but in all cases emit **user-visible device-change notifications** and support **safety-number re-verification**. Revocation removes the device's keys from the recipient set and adds its session to the deny-list (AD-002).

**Why:** It delivers seamless multi-device UX and account recovery while making unauthorized device addition visible and verifiable — the practical trust model used by leading E2EE messengers. It composes cleanly with AD-005 key management and AD-010 synchronization.

**Trade-offs:** Depends partly on user attention to notifications; per-device fan-out cost must be managed (AD-020 sender-keys, AD-013 storage).

**Risks:** Silent device addition if notifications are ignored (mitigated by mandatory prompts + optional trusted-contact verification); history backfill must remain ciphertext-only.

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Robust multi-device with recovery; visible, verifiable trust; clean revocation.
- **Negative:** Fan-out and device-list management overhead; UX depends on notification heed.
- **Future Impact:** Drives AD-010 (sync/backfill per device), AD-020 (group sender-key distribution per device), and AD-011 (presence per device).

---

## Affected Documents

- `docs/40-security/43-key-management-and-devices.md`
- `docs/30-domain/35-sequences/SQ-03-device-registration.md`
- `docs/30-domain/35-sequences/SQ-08-message-synchronization.md`

## Affected ADRs

- ADR-0019 (Multi-Device)

## Affected Modules

- Identity

## Open Questions

- Maximum number of devices per user?
- Recovery-path policy when no existing device is available?
- Mandatory vs. optional safety-number verification UX?

## Approval

- **Status:** Under Review
- **Owner:** Security
- **Review Date:** (pending)
- **Decision Date:** (pending)
