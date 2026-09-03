# Spec: c4-diff skill (module-level visual PR diffs)

Refs: ADR-0009 (proposed). Design reference: `jovaneyck/skills`
`skills/engineering/c4-diff` (MIT) — this spec adopts its workflow and adapts
it to our Django modules. Status: spec, unimplemented — blocked until Ticket 1
lands real code.

## Problem Statement

As a reviewer, I open a PR and get a wall of line diffs with no sense of
structural impact, so I either read everything line-by-line or gloss over it
(comprehension debt). I want before/after/merged-diff diagrams that tell me
where to look.

## Solution

A CI-backed skill following the reference workflow: resolve base/head
(merge-base for PRs, never the branch tip), extract the Django-app dependency
graph at both ends deterministically, render three Mermaid `C4Component`
diagrams (`before`, `after`, merged `diff` with red/green/amber overlay), and
post them as a PR comment GitHub renders natively — each with commit metadata
in the title, an Evidence section, and a summary answering what's new, what's
gone, what relationships changed.

## User Stories

1. As a reviewer, I want before/after/diff diagrams on every backend PR, so
   that I see structural impact before lines.
2. As a reviewer, I want added/removed/changed elements in green/red/amber
   with `[+]/[-]/[~]` label prefixes, so that the diagram survives color-blind
   and plain-text viewing.
3. As a reviewer, I want every changed node/edge traced to file path + symbol
   in an Evidence section, so that the diagram is reviewable, not decorative.
4. As a reviewer, I want renames shown as changed (not add+remove) with stable
   node IDs across the rename, so that context doesn't disappear.
5. As a maintainer, I want boundary violations (`infra/arch-boundaries.yaml`)
   to fail the build, so that the map can't lie.
6. As a maintainer, I want the extractor to be deterministic code (stdlib `ast`
   walk), so that no LLM guesswork enters the map.
7. As a junior, I want to sketch my expected diagram before a big change and
   compare it against the generated diff, so that I build a theory of the system.
8. As a contributor, I want an explicit "no structural change" result for
   formatting/config-only PRs, so that small slices stay cheap to review.

## Implementation Decisions

- Modules = Django apps (`users, articles, generations, publishing, metrics`).
  Allowed-dependency rules live in `infra/arch-boundaries.yaml` (new file).
- Extractor seam: `extract_graph(tree) -> {nodes, edges, violations}` — pure
  function over a directory tree (stdlib `ast`, no new deps). Read each file at
  both commits via `git show <ref>:<path>` without touching the working tree.
- Scope: impacted subgraph + one hop of neighbors computed over the **union**
  of before/after graphs (removed nodes keep their context). Cap ~20 elements.
- Base for PRs = `git merge-base main feature`, or unrelated base-branch
  commits leak into the diff and misattribute changes.
- Render: Mermaid `C4Component` (native GitHub rendering beats Archify HTML
  here — HTML can't embed in a PR comment). Overlay codes, copied verbatim:
  added `#e6ffed/#22863a`, removed `#ffeef0/#cb2431`, changed
  `#fff5b1/#b08800/#735c0f`. Legend + prefixes live only in `diff`.
- Artifacts per PR: `before.component.md`, `after.component.md`,
  `diff.component.md` — each with title metadata (short SHA, subject, author,
  date), diagram, Legend (diff only), Evidence, Summary.
- Trigger: GitHub Actions `pull_request` on `backend/**`; one idempotent
  comment per PR; "no structural change" note when graphs are identical.

## Testing Decisions

- Good test = extractor output on fixture trees, never on the live repo.
- Fixtures: clean tree, added-module tree, removed-edge tree, renamed-file
  tree (asserts changed-not-add+remove), boundary-violation tree (nonzero exit
  + named violation).
- Integration: golden triple-artifact for a fixture PR pair; byte-compare after
  canonical serialization.
- Honesty rules (from reference, enforced in review, not code): structure, not
  sequence — call-order-only changes are not structural; never infer
  relationship types the code doesn't show; small honest diagram over
  exhaustive one.

## Out of Scope

- Frontend route diffs (Phase 2, same pattern).
- Line-level analysis — existing diff + scanners keep that job.
- Auto-approval or merge gating on "diagram looks small" — human judges.
- LLM-summarized diffs in the comment (untrusted-content risk, skip).

## Further Notes

- Kill-switch (ADR-0009): if extraction ever becomes LLM-guessed, delete this.
- Run order with `code-review`: c4-diff first (where to look), standards/spec
  review on flagged modules second, 10-minute exercise third (trace the
  spiciest module through the stack — system walk).
- Iterative fit: every ticket's delta should be small enough that its diff
  diagram is boring. A spicy diagram on a "small" ticket means the slice was
  too big — re-scope, don't push on.
