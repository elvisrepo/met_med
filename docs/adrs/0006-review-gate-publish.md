# 0006. Review gate + idempotent publish

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

LLM output needs brand-safety control, and Instagram posts have no undo.
Token expiry mid-publish must not duplicate posts.

## Decision

Mandatory human review: `draft → in_review`, publish API requires `in_review`
(409 otherwise). Idempotency key = `generation_id`; 2-step container → publish;
code 190 parks the job as `token_expired` and resumes after refresh.

## Consequences

- No accidental or duplicate posts by construction.
- Human bottleneck is intentional for MVP; scheduler/auto-post stays Phase 2.
