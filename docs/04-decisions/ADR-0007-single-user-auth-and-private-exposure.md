# ADR-0007: Single-user authentication and private network exposure

**Status:** Accepted · **Date:** 2026-10-05

## Context

Exactly one user; data is personal and career-sensitive; the host is self-managed; phone access matters. Full identity-provider integration is heavy for one person; no authentication at all is unacceptable even on a home network.

## Decision

- Local **single account** with password (Argon2id) and optional TOTP; opaque session tokens in HttpOnly, Secure, SameSite=Lax cookies, stored hashed in PostgreSQL; login throttling and lockout; `Origin` checks for CSRF.
- **Recommended exposure:** a private overlay network (Tailscale or WireGuard) with TLS at Caddy; public exposure is supported but requires TOTP enabled and a real domain with automatic TLS.
- Every query is scoped by `user_id` so a later sharing feature changes authorisation filters, not the data model.

## Alternatives considered

- **External identity provider (OIDC) or forward-auth proxy (Authelia, Authentik).** Robust, but another component to run; acceptable later as an optional mode in front of Caddy.
- **No auth, rely on the private network.** Rejected: one misconfiguration exposes everything; phones roam.
- **Passkeys only.** Appealing, but browser and device support friction for a single user; may be added as an alternative factor.

## Consequences

- Small, auditable auth code in `identity`; no third-party auth dependency.
- The operator must manage the overlay network; the deployment doc covers it.
- Multi-user requires adding roles and invitations later; not blocked by this decision.

## Revisit triggers

- Sharing with a mentor or manager (feature 018) requires a second account or guest links.
