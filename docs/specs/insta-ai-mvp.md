# Insta AI MVP — Spec (locked 2026-09-03)

Source: grilling R1 Q1-Q6 + R2 Q7-Q12, `docs/insta-ai-mvp-design.html`.
Stack: Django 5 + DRF monolith, React Vite SPA, Postgres 16, Redis + Celery, Docker Compose local = VPS prod, Caddy SSL.

## Problem Statement

Employees paste article links and manually craft Instagram posts — slow, inconsistent branding per category, no cost/ownership tracking. Need internal tool: URL → category-based caption + AI image → mandatory review → IG post + metrics.

## Solution

Invite-only web app for 2-4 employees. Paste URL → server fetches title/body/image/date → LLM generates caption/hashtags + 1080x1350 image → editor reviews/edits/regenerates per-field → posts to IG Business via Graph API → logs tokens/cost/status. Dashboard shows posts/day, per-employee, cost/post, failures.

## User Stories

1. As an editor, I want to paste an article URL so that title/date/body/main-image/source are auto-fetched with zero per-site config.
2. As an editor, I want auto-fetch to work on JS-heavy pages so that pasting the link is enough (no manual copy).
3. As an admin, I want Firecrawl as opt-in fallback so that hard-JS pages still auto-fetch when I enable it, with cost tracked.
4. As an editor, I want manual title/body fallback as dead-last resort so that hard paywall failures don't lose work.
5. As an editor, I want auto-detected category (sport/science/showbiz/politics/general) with manual override so that layout matches brand.
6. As an editor, I want generated caption + hashtags editable with live char count so that I control voice.
7. As an editor, I want per-field regenerate (caption-only/image-only) so that I save cost.
8. As an editor, I want mandatory review gate so that nothing auto-posts without approval.
9. As an editor, I want one-click Approve + Post so that publishing is idempotent, no duplicates.
10. As an editor, I want dashboard (posts/day, per-employee, avg cost, failures) so that I see output and spend.
11. As an admin, I want to invite/deactivate users via one-use 24h link so that no public signup exists.
12. As an admin, I want reconnect banner + alert on IG token expiry (code 190) so that publishing resumes without duplicates.
13. As an admin, I want nightly pg_dump to B2 + restore test so that data survives VPS loss.

## Implementation Decisions

Modules: `users, articles, generations, publishing, metrics` (single Django monolith, no microservices).
Seam: `generate_post(article) -> {caption, hashtags, image_key, tokens}` — adapters: LLM-text, LLM-image, IG-publisher, B2-storage. Fakes in tests.
Auth: admin-invite-only, no public register, no Google OAuth. JWT httpOnly + Secure + SameSite=Lax, CSRF, login rate-limit 5/min. All LLM/IG keys server-side.
Fetch (`fetch_article`, 3-tier, generic — no per-site selectors required):
Tier-1 free fast (default): normalize URL (strip UTM, sha256 url_hash, indexed NOT unique — same URL reusable), SSRF guard (http/https only, block private/169.254, 15s timeout, 5MB cap, 3 redirects), httpx → OG/Twitter/JSON-LD (`og:title`, `og:image`, `article:published_time`, `NewsArticle{headline,image,datePublished,articleBody}`) + trafilatura/readability body (text-only, 50k cap, min 300 chars) + image-score (srcset/data-src normalized, drop logo/avatar/ad/1px, score in-article/size/early/alt/aspect, min 400px) → title/body/main_image_url/published_at/source. Retry 2x.
Tier-2 free JS (auto when Tier-1 signals js_shell/short-body): Playwright Chromium headless, isolated `fetch_js` queue (concurrency 1-2), block fonts/ads, wait article+h1 (20s), 1 scroll, then same extractors on rendered DOM.
Tier-3 Firecrawl fallback (disabled by default: `FIRECRAWL_ENABLED=false`, `FIRECRAWL_API_KEY` server-only in .env, never browser): when enabled, `scrape {formats:[markdown,html], onlyMainContent:true}` → same extractors on returned HTML. Log `fetch_tier: free|playwright|firecrawl` + `fetch_cost_usd` per Generation. Enabling later is config-only, no rewrite.
Hard paywall (401/server-truncated): no bypass (ToS). Status `fetch_failed` + domain + reason. Source image mandatory: zero candidates → `needs_image` block, generation refused. Text-only pages: noted edge, unlikely per Dori — blocked for now, revisit if hit.
Generate (`generate_caption`): parallel text + image, log model_text/image_version + image_prompt + tokens_in/out + cost_usd even on fail. Append-only generations.
Publish (`publish_ig`): requires generation.status=in_review, creates Post draft → queued → posted. Idempotency generation_id, 2-step container→publish. `refresh_ig_token` beat daily (60-day token).
Categories: sport / science / showbiz / politics / general. Politics neutral-factual + source link, showbiz no rumors as fact, politics/showbiz images abstract no real faces, no text overlay, 3-5 hashtags, source credit mandatory.
Frontend: routes /login /new /review/:id /dashboard /users, React-Query polling, no Redux. Caddy serves SPA, proxies /api/*.
Deploy: one compose (caddy/web/gunicorn/worker/beat/db/redis), Caddy auto-SSL, migrate on deploy, .env 600 on host, provider firewall. No Terraform/K8s.
NFR: p95 <90s article→ready_for_review, <$0.10/post (alert at $0.07 avg, cap LLM_MONTHLY_CAP_USD, block image regen at 90%), OWASP basics + audits, no full-text republication (summary + link only).

```yaml
entities:
  User: {id: uuid PK, email: unique, role: admin|editor, active: bool}
  Article: {id: uuid, created_by: FK, url: indexed, url_hash: sha256 indexed, title, body: text, main_image_url, source, published_at, fetch_status, created_at}
  Generation: {id: uuid, article_id: FK, created_by: FK, category, caption: text, hashtags, image_url, image_storage_key, image_prompt, model_text_version, model_image_version, tokens_in: int, tokens_out: int, cost_usd: numeric, fetch_tier: free|playwright|firecrawl, fetch_cost_usd: numeric, status, created_at}
  Post: {id: uuid, generation_id: FK unique, posted_by: FK, ig_media_id: unique, status: draft|in_review|queued|posted|failed|token_expired, error: text, posted_at}
api:
  POST /api/articles {url} -> 202 {article_id}
  GET /api/articles/:id -> {url,title,body,main_image_url,published_at,source,fetch_status}
  POST /api/generations {article_id, category?} -> 202
  GET /api/generations/:id -> {category,caption,hashtags,image_url,tokens,cost_usd,model_versions,status}
  POST /api/generations/:id/regenerate {field} -> 202
  POST /api/posts {generation_id} # requires in_review
  POST /api/posts/:id/publish -> 202
  GET /api/dashboard?days=30 -> {posts_per_day, per_employee, avg_cost, failures}
```

## Testing Decisions

Good test = external behavior via seam, not internals. Prior art: harness smoke-checks only, so start fresh.
- Unit (pytest/vitest): URL normalize+hash, cost calc, prompt builder per category.
- Integration (pytest + fakes): article→generation→post happy path, 400 bad URL, 422 fetch_failed, needs_image block when zero image candidates, 409 publish without in_review, 190 token_expired park/resume, Firecrawl path mocked (disabled by default, enabled flag + cost logged).
- E2E (Playwright): login → paste → review → post → dashboard → invite. Manual 10-min exercise per session: open rendered app, verify counts/costs.
- Small-batch rule: never rewrite tests + impl same pass. Vibe Diff summary required on auth/publish diffs. CI blocks on pytest/eslint/pip-audit/npm audit/semgrep.

## Out of Scope

Scheduler, bulk URLs, auto-post, Facebook publisher, WordPress publisher (Phase 2 adapters), Google OAuth, Terraform/K8s, CDN, Elasticsearch, email integration, advanced analytics.

## Further Notes

Design doc: `docs/insta-ai-mvp-design.html` (HLD + sequence + ERD mermaid + 5 clickable wireframes). Next: `to-tickets` vertical slices (auth-invite → fetch → generate → review → publish → dashboard → deploy). Dori owes 1 example post per category for voice tuning after first 10 posts.
