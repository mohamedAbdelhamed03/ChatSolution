# AD-005 — Key Management (Decision Recommendation)

## Question

How are cryptographic keys generated, published, stored, distributed, and retired so that clients can establish E2EE sessions asynchronously while the backend only ever handles public key material?

---

## Background

**Business context:** E2EE is only as strong as its key management. Users add/remove devices, go offline, and occasionally lose devices; keys must support all of this without ever exposing private material to the server.

**Technical context:** Following AD-004 (Signal Protocol), each device has a long-term identity key, signed prekeys, and one-time prekeys. The server acts as a **blind key directory**: it stores and serves public keys/prekeys and never holds private keys.

**Constraints:** Backend never decrypts (INV-01); private keys never leave the device; asynchronous setup for offline recipients; multi-device (AD-006).

---

## Requirements

- **Functional:** Publish per-device public identity key + signed prekey + one-time prekeys; fetch a recipient's prekey bundle; replenish one-time prekeys; revoke device keys.
- **Non-Functional:** Prekey serving is high-throughput and low-latency; prekey pools do not exhaust.
- **Security:** Private keys generated and stored only on-device (secure enclave/keystore where available); signed prekeys authenticated by the identity key; safety-number verification supported.
- **Scalability:** Key directory scales to millions of devices; prekey replenishment is efficient.

---

## Alternatives

### Alternative A — Server as blind public-key directory (prekey bundles), private keys device-only
- **Pros:** Standard Signal model; enables asynchronous session setup; server never sees private keys; clean multi-device support.
- **Cons:** Prekey pool management (exhaustion, replenishment); server must authenticate publishers to prevent key injection.

### Alternative B — Client-to-client key exchange only (no server directory)
- **Pros:** Minimal server key role.
- **Cons:** Breaks asynchronous setup (both parties must be online); poor UX; impractical for mobile/offline.

### Alternative C — Server-escrowed keys (server holds/derives key material)
- **Pros:** Easy recovery.
- **Cons:** Violates INV-01 (server could decrypt); unacceptable.

---

## Industry Research

- **Signal/WhatsApp (documented):** Use prekey bundles served by the server for asynchronous session establishment; private keys remain on device; identity keys underpin safety numbers/security codes.
- **Documented pattern:** One-time prekeys are consumed per session setup and replenished; signed prekeys rotate periodically.
- **Encrypted backup (documented):** Signal and WhatsApp offer end-to-end encrypted backups protected by a user secret/recovery key, so the server still cannot read content — the informed pattern for reconciling recovery with E2EE.

---

## Recommendation

**Recommend Alternative A:** the backend is a **blind public-key directory** storing per-device **identity keys, signed prekeys, and one-time prekey pools**; **all private keys are generated and retained only on the device** (hardware-backed storage where available). Provide prekey replenishment, signed-prekey rotation, publisher authentication (to prevent key injection), and safety-number verification. Account recovery uses **client-side end-to-end encrypted backups** (user-held recovery secret), never server escrow.

**Why:** This is the only model that supports asynchronous, multi-device E2EE while strictly upholding INV-01. It matches the proven Signal/WhatsApp approach and cleanly separates public (server) from private (device) material.

**Trade-offs:** Prekey pool operations and rotation add server/client logic; recovery depends on a user-held secret (lost secret = lost history), which must be clearly communicated.

**Risks:** Prekey exhaustion (mitigated by replenishment + fallback signed prekey); key-injection/MITM (mitigated by publisher auth + safety numbers, see AD-006 device trust); backup-secret loss (UX and clear expectations).

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Asynchronous, multi-device E2EE; server never holds private keys; standard, auditable model.
- **Negative:** Prekey lifecycle management; recovery limited by design (no server content recovery).
- **Future Impact:** Enables AD-006 (Device Model), AD-024 (Key Rotation), OPS-07; constrains AD-048 (DR) to encrypted backups.

---

## Affected Documents

- `docs/40-security/43-key-management-and-devices.md`
- `docs/40-security/49-key-compromise-recovery.md`
- `docs/98-operations/OPS-07-key-rotation.md`
- `docs/50-data/57-retention-ttl-and-backup.md`

## Affected ADRs

- ADR-0004 (E2EE), ADR-0019 (Multi-Device)

## Affected Modules

- Identity, Messaging

## Open Questions

- One-time prekey pool size and replenishment thresholds?
- Signed-prekey rotation cadence?
- Encrypted-backup design and recovery-secret UX?

## Approval

- **Status:** Under Review
- **Owner:** Security
- **Review Date:** (pending)
- **Decision Date:** (pending)
