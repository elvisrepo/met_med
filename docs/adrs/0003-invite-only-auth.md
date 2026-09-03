# 0003. Invite-only auth

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

Internal tool; random signups must be impossible. Small trusted user base.

## Decision

Admin-invite-only: one-use 24h invite/reset links, JWT in httpOnly + Secure +
SameSite=Lax, CSRF, login rate-limit 5/min. No public signup, no Google OAuth
in MVP. Route `/accept-invite?token=...` activates accounts.

## Consequences

- Smallest attack surface, no OAuth integration cost.
- Admin bottleneck for onboarding; no SSO. Revisit SSO if the org outgrows invites.
