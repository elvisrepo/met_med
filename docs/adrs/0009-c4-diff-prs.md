# 0009. C4 diff on PRs

- Status: proposed
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

Line-level git diffs don't show structural impact. For comprehension
(anti-comprehension-debt) the reviewer needs a module-level before/after view
to direct attention at surprises before reading lines. Reference design:
`jovaneyck/skills` `c4-diff` (before/after/merged-diff, red/green/amber).

## Decision (proposed)

On every PR touching `backend/`: diff against the **merge-base** (not the
branch tip), extract the Django-app dependency graph deterministically
(AST/import walk, not LLM) at both ends, and render three Mermaid
`C4Component` diagrams — `before`, `after`, and a merged `diff` with
red/green/amber overlay + `[+]/[-]/[~]` label prefixes. GitHub renders Mermaid
natively, so the diff embeds directly in the PR comment with an Evidence
section (file path + symbol per changed node/edge) and a summary answering
what's new, what's gone, what relationships changed. Boundary violations fail
the build so the map can't lie.

## Consequences

- Reviews start at system level, drill to lines only where the diagram surprises.
- Requires real code + enforced module boundaries first — cannot function before
  Ticket 1 lands. Costs one CI job per PR.
- If the extractor is ever LLM-guessed instead of deterministic, kill this ADR:
  a lying map is worse than none.
