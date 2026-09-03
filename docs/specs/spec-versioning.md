# Spec versioning (audit convention)

All specs live with the code and version with it via git. Three layers:

## 1. Standing specs (already done)

- `docs/specs/insta-ai-mvp.md` — the product contract (Problem/Solution/Stories/
  YAML entities+API/Tests). Edited in place; history = `git log`.
- `docs/specs/glossary.md` — vocabulary; same rule.
- `docs/adrs/` — decisions with explicit `Version: v1…` + Changelog (stronger
  than git alone: the version is *in the file*).

## 2. Change specs (the delta pattern — adopt from the next ticket on)

Each feature/fix gets a folder; the agent takes current code + delta → next code:

```
docs/specs/changes/NNN-short-slug/
  proposal.md  — what + why, linked ticket, refs ADR (1 page max)
  api-diff.md  — endpoint/schema changes, or "none"
  tests.md     — seam, cases, status codes asserted
```

Rules: one folder per ticket, merged-with-ticket (same PR), never edited after
merge — follow-ups get a new folder. `proposal.md` past tense OK after merge;
it is the audit trail of *what changed and why*, ADRs stay the trail of *why
this way over alternatives*.

## 3. API layer spec (machine-enforced contract — highest value)

- Source of truth for routes: `POST /api/docs` (drf-spectacular) in code.
- Commit the generated artifact: `backend/docs/openapi.yaml`, regenerated in CI;
  PR fails if the artifact is stale (deterministic check, not a reminder).
- Breaking changes require a versioned path (`/api/v2/…`) + `api-diff.md`.
- Later: `openapi-diff` in CI posts the contract delta on the PR — same idea as
  the c4-diff skill, but for the API surface. Cheap once the artifact exists.

## What this buys

Audit = `git log docs/specs/changes/` + ADRs. Agent workflow per ticket:
read standing spec → read this ticket's `changes/NNN/` delta → implement →
regenerate `openapi.yaml` → run gates. No chat archaeology.
