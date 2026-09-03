# ADRs — decision log

One file per significant decision: `NNNN-short-slug.md` (short titles only).

## Record format

```md
# NNNN. Short title

- Status: proposed | accepted | deprecated | superseded
- Version: v1 (bump on every change, note it under Changelog)
- Date: YYYY-MM-DD
- Supersedes: — (or `NNNN vX` when this version reverses one)

## Context
Why the decision was needed, constraints at the time.

## Decision
What was chosen, and the main alternative rejected.

## Consequences
Good / neutral / bad outcomes accepted with the choice.
```

## Rules

- New decision → new numbered file at v1. Changed decision → bump `Version`
  in the same file + `Changelog` entry. Full reversal → new file with `Supersedes`.
- Never silently rewrite an accepted ADR; a bump is a conscious act.
- Reference ADRs from code/PRs as `refs ADR-NNNN`.
- If implementation contradicts an accepted ADR, stop and re-anchor (same as spec drift).
