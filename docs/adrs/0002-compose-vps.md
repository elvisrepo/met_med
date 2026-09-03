# 0002. Compose on cheap VPS

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

VPS budget, no ops team. Must run local = prod with minimal moving parts.

## Decision

One `docker-compose.yml` (caddy + web + worker + beat + db + redis) is the
deploy unit locally and on the VPS. No Terraform, K8s, ALB, or managed DB in MVP.

## Consequences

- Reproducible and cheap; Compose file doubles as IaC.
- Single host = downtime on host failure, vertical scaling only.
  Revisit with managed Postgres + second host when uptime demands it.
