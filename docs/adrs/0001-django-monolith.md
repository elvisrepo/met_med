# 0001. Django monolith

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

2–4 internal users, ~100 posts/month, highly relational data
(User → Article → Generation → Post), one team, cheap single VPS.

## Decision

Single Django 5 + DRF monolith with 5 modules
(`users, articles, generations, publishing, metrics`).
No microservices, no GraphQL in MVP.

## Consequences

- Simple deploy, test, and TDD seam (`generate_post`); DB transactions across the flow.
- Must keep modules decoupled; revisit at >10k posts/month or a second team.
