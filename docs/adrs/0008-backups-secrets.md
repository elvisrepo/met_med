# 0008. B2 backups + env secrets

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

Business data loss is unacceptable; no ops team and no budget for managed
secret stores. Target RPO <24h.

## Decision

Nightly `pg_dump | gzip | rclone → B2`, 30-day retention, monthly restore test.
Secrets in server `.env` (mode 600). No Vault/Secrets Manager in MVP.
LLM/IG/Firecrawl keys never reach frontend bundles or logs.

## Consequences

- Cheap durability with a tested restore path.
- Manual restore procedure; single-copy risk if B2 is misconfigured —
  mitigated by the monthly restore test, not by more tooling.
