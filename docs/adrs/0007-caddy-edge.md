# 0007. Caddy at the edge

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

Single host needs SSL, static SPA serving, and `/api/*` proxying with no ops overhead.

## Decision

Caddy as the edge: auto-SSL, serves the SPA, proxies `/api/*` + `/admin/*`,
rate-limits login at the edge. No API Gateway, LB, or CDN in MVP.

## Consequences

- Zero-config TLS and one fewer service to run.
- Single point at the edge; revisit CDN when media bandwidth matters.
