# 0004. Three-tier fetch

- Status: accepted
- Version: v1
- Date: 2026-09-03
- Supersedes: —

## Context

Paste-any-URL must auto-fetch title/date/body/image with zero per-site
selectors, including JS-heavy pages, while holding <$0.10/post.

## Decision

Tier-1 free fast (httpx + OG/JSON-LD + trafilatura) → Tier-2 free Playwright
(isolated `fetch_js` queue) → Tier-3 Firecrawl opt-in fallback, disabled by
default (`FIRECRAWL_ENABLED=false`). Log `fetch_tier` + `fetch_cost_usd` per
generation. Hard paywalls are never bypassed (ToS).

## Consequences

- ~90–95% of pages free and automatic; vendor-independent by default.
- Playwright costs RAM/complexity; Firecrawl adds per-call cost when enabled.
