# Glossary — Insta AI MVP (canonical vocabulary)

Tiebreaker: on conflict, `docs/specs/insta-ai-mvp.md` wins, then ADRs, then this file.

## System in a paragraph

An editor pastes an article URL. The fetcher extracts title, date, body text,
and a mandatory source image (Tier-1 free → Tier-2 Playwright → Tier-3
Firecrawl if enabled). The generator auto-detects one of five categories and
produces a caption, hashtags, and an AI image, logging tokens and cost. Nothing
publishes until an editor reviews and approves; publishing to Instagram is
idempotent per generation. A dashboard tracks output and spend per employee.

## Terms

- **User** — employee account. Roles: `admin` (invite/deactivate, reconnect IG),
  `editor` (paste → review → post). Invite-only, no public signup (ADR-0003).
- **Article** — one fetched URL: normalized `url` + `url_hash`, `title`, `body`
  (text only), `main_image_url` (reference, not scraped for publishing),
  `source` (domain), `published_at` (nullable), `fetch_status` (`ok|fetch_failed`).
  Same URL is reusable — never unique-constrained.
- **Generation** — one AI pass over an article: `category`, `caption`, `hashtags`,
  `image_url` + `image_storage_key` + `image_prompt`, `model_text/image_version`,
  `tokens_in/out`, `cost_usd`, `fetch_tier` + `fetch_cost_usd`. Append-only; edits
  and regenerates create new rows. `status`:
  `ready|gen_failed|needs_image|in_review|approved|publish_failed`.
- **Post** — at most one per generation: `ig_media_id`, `posted_by`, `posted_at`,
  `error`. `status`: `draft|in_review|queued|posted|failed|token_expired`.
- **Category** — `sport|science|showbiz|politics|general`. Auto-detected,
  editor-confirmed. Drives tone, hashtags, and image style. Politics =
  neutral-factual; showbiz = no rumors as fact; both = abstract images, no faces.
- **fetch_tier** — which pipeline produced the article data:
  `free` (Tier-1) | `playwright` (Tier-2 JS render) | `firecrawl` (Tier-3 fallback)
  | `manual` (dead-last hand paste). Logged per generation (ADR-0004).
- **needs_image** — blocking state: zero image candidates ≥400px, or manual paste
  without an image URL (422). Generation is refused until resolved (ADR-0005).
- **in_review** — the mandatory gate state. Publish API requires it (409 otherwise).
  No auto-post path exists in MVP (ADR-0006).
- **Idempotency key** — the `generation_id`. Reposting the same generation returns
  the existing post instead of duplicating (duplicate-blocked, not an error).
- **token_expired** — IG long-lived token hit code 190. Publish parks, admin is
  alerted, job resumes after `refresh_ig_token` with no duplicate.
- **cost_usd / fetch_cost_usd** — LLM tokens converted to dollars per generation,
  plus Firecrawl spend when Tier-3 runs. KPI: avg <$0.07/post, hard cap $0.10.
- **Invite** — one-use 24h link created by an admin; redeemed at
  `/accept-invite?token=...` by setting a password. Deactivation keeps history.
- **Publisher adapter** — Phase-2 seam for new outputs (scheduler, WordPress
  reusing the existing flow, Facebook). Instagram is the only MVP adapter.
- **fetch_js queue** — isolated Celery queue for Playwright renders
  (concurrency 1–2, ~300MB), so slow JS pages never block the fast Tier-1 path.
