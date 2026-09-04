# 0010. Model stack (text + image)

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

Pipeline needs, per post: category detect + summary + caption + hashtags with
translation between Albanian and source languages, plus one abstract 1080×1350
image (no faces, no text overlay). Albanian is low-resource: comprehension is
stronger than generation, Standard (Tosk) beats Gheg, idioms drift. Budget cap
is $0.10/post all-in.

## Decision

- **Text default: Gemini Flash tier** ($0.75/$3.75 per 1M in/out, 1M context,
  thinking pinned off — reasoning tokens are pure tax on this workload).
  Shortlist to benchmark before lock-in: GPT mini-tier, Claude Sonnet-tier.
- **Image default: Imagen Fast tier** (~$0.01/image) — same vendor/key/bill as
  text. Fallback: Flux Pro via fal/Replicate (~$0.03–0.05). No typography king
  needed (our images carry no text by design).
- **Provider abstraction, not provider marriage**: one `LLMText` + one
  `LLMImage` adapter behind the `generate_post` seam (env-switchable model
  ids, versions logged per generation). Switching = config + fixture run.
- **Benchmark before trust**: approved caption fixtures must include real
  Albanian articles (Standard + Gheg sample); a model change requires fixtures
  green or a justified diff. Mandatory human review stays the final guardrail
  for Albanian nuance — no model publishes unchecked.

## Consequences

- Math at 100 posts/mo (~2k in / 400 out tokens + 1–2 images with regens):
  text ≈ $0.30, images ≈ $3–9. Total ≈ $0.03–0.09/post vs $0.10 cap.
  Even frontier text models would cost <$3/mo here — so this ADR optimizes for
  Albanian quality and one-bill practicality, not token pennies.
- Single-vendor concentration (Google) for both modalities; mitigated by the
  adapter seam + logged versions making a switch a one-ticket job.
- Revisit on: fixture regressions, >3× price moves, or volume crossing
  5k posts/mo (then re-run the benchmark, bulk/batch discounts apply).
