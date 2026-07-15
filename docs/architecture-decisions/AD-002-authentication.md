# AD-002 — Authentication (Decision Recommendation)

## Question

How do users and their devices prove their identity to the backend, and how are sessions established and maintained across multiple devices?

---

## Background

**Business context:** Users log in from several devices and expect fast, secure, low-friction access. Authentication is the gate for every request.

**Technical context:** Services are stateless and horizontally scaled; SignalR (WSS) connections must also be authenticated. Authentication must anchor to the internal `UserId` (AD-001) and to a per-device identity for E2EE.

**Constraints:** Every external request authenticated (INV-08); HTTPS/WSS only (INV-09); stateless nodes (NFR-S-02); backend never decrypts content (INV-01).

---

## Requirements

- **Functional:** Register/login/logout; establish per-device sessions; refresh without re-login; revoke a device.
- **Non-Functional:** Low-latency verification on every request without a central session lookup on the hot path.
- **Security:** Strong credential handling; token theft resistance; per-device revocation; protection against replay and session hijacking.
- **Scalability:** Verification must scale to millions of concurrent connections.

---

## Alternatives

### Alternative A — Stateful server sessions (session store in Redis)
- **Pros:** Instant revocation; simple mental model; small tokens.
- **Cons:** Central lookup on every request (hot-path dependency); scaling and latency cost; couples every node to Redis for auth.

### Alternative B — Stateless JWT access tokens + rotating refresh tokens
- **Pros:** No hot-path lookup (self-contained, signature-verified); scales statelessly; natural per-device tokens; works for REST and SignalR.
- **Cons:** Revocation is not instant (until access token expiry); requires careful refresh rotation and key management.

### Alternative C — Hybrid: short-lived JWT access tokens + refresh tokens + a lightweight revocation list in Redis
- **Pros:** Stateless hot path with near-instant revocation via a small deny-list checked cheaply; best balance of scale and control.
- **Cons:** Slightly more moving parts; deny-list must be replicated/low-latency.

---

## Industry Research

- **Documented pattern:** OAuth 2.0 / OIDC with short-lived access tokens and refresh-token rotation is the industry standard for scalable authentication.
- **Messaging apps (informed):** Consumer messengers typically bind a session to a specific device after an initial verification (SMS/code/QR link from an existing device) and then use long-lived device sessions with rotating tokens; exact token schemes are generally not publicly documented.
- **Signal/WhatsApp (documented):** Device linking uses a primary device to authorize secondary devices (QR-based), keeping cryptographic identity per device.

---

## Recommendation

**Recommend Alternative C:** short-lived **JWT access tokens** (per device) with **rotating refresh tokens**, plus a **Redis-backed revocation/deny-list** for immediate device/session revocation. Initial device verification uses a code (email/phone) or primary-device QR linking; each device holds a distinct session bound to its `DeviceId`.

**Why:** Keeps the request hot path stateless (JWT signature verification) to meet scale targets, while the small deny-list restores the fast revocation that a purely stateless design lacks. It naturally supports multi-device and SignalR authentication (token on connect).

**Trade-offs:** Access-token lifetime balances revocation latency vs. refresh frequency; signing-key management is required (rotation).

**Risks:** Refresh-token theft (mitigated by rotation + reuse detection); clock/expiry handling; deny-list availability (must be low-latency and replicated).

*(Not approved — recommendation only.)*

---

## Consequences

- **Positive:** Stateless, scalable verification; per-device sessions; immediate revocation when needed.
- **Negative:** Requires signing-key rotation and refresh-rotation logic; deny-list infrastructure.
- **Future Impact:** Sets session foundation for AD-006 (Device Model) and AD-019 (Session Management); enables SignalR auth in AD-039.

---

## Affected Documents

- `docs/40-security/42-authn-authz.md`
- `docs/80-api/82-signalr-hub-contract.md`
- `docs/30-domain/35-sequences/SQ-02-login.md`

## Affected ADRs

- ADR-0019 (Multi-Device)

## Affected Modules

- Identity

## Open Questions

- Access-token TTL and refresh-token lifetime targets?
- Initial verification method(s) at launch (SMS, email, QR device-linking)?
- Signing algorithm and key-rotation cadence?

## Review Outcome (2026-07-15)

**Reviewer:** Chief Software Architect · **Verdict:** Approve with Changes

**Required changes applied:**
- **Replay protection added:** access tokens carry a short expiry and a unique `jti`; the SignalR connect handshake requires a fresh token and rejects reused/expired tokens. All transport is WSS/HTTPS (INV-09), preventing token capture in transit.
- **Token binding added:** refresh tokens are bound to `DeviceId` and use rotation with reuse-detection (a replayed refresh token revokes the whole token family via the Redis deny-list).
- **Revocation latency bounded:** access-token TTL is capped (recommended ≤15 min) so the stateless hot path plus deny-list gives near-immediate effective revocation.
- **Clock-skew handling noted:** a small leeway is allowed on expiry validation to tolerate device clock drift.

**Residual open questions (owner follow-up, non-blocking):** exact access/refresh TTLs; launch verification methods (SMS/email/QR); signing algorithm and key-rotation cadence (coordinate with OPS-05).

**Quality scores** — Architecture 9 · Security 9 · Scalability 10 · Maintainability 9 · Documentation 9 · **Overall 9.3**

## Approval

- **Status:** Approved
- **Owner:** Security
- **Reviewed by:** Chief Software Architect (Architecture Decision Review Sprint)
- **Review Date:** 2026-07-15
- **Decision Date:** 2026-07-15
