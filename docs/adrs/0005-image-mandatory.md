# 0005. Source image mandatory

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

Every IG post needs a visual. The AI generates the final image, but editors
need a detected source image for context and category grounding.

## Decision

A scored source-image candidate (min 400px) is mandatory. Zero candidates →
`needs_image` 422 and generation is refused; manual paste requires an image URL.

## Consequences

- No imageless posts slip through.
- Text-only source pages hard-block (accepted as unlikely edge; revisit if hit).
