# OpenDesign reconciliation (2026-09-03)

Export: `docs/Insta-AI-MVP-System-Design/screens/` (login, new, review, dashboard, users).
Spec: `docs/specs/insta-ai-mvp.md`. Export is a visual contract — NOT edited.
Resolutions below are locked into the spec.

## Must-fix (resolved in spec)

1. **Missing accept-invite view** — export has invite (users) + sign-in (login) but no
   set-password target for the one-use link.
   Resolution: new route `/accept-invite?token=...` (token validity check, new-password
   + confirm, min 8 chars, one-use, 24h expiry). Added as Story 14 + Frontend routes.
   Reference screen (same Clean tokens, export left untouched):
   `docs/screens/accept-invite.html` — states: valid form, 410 missing/expired token,
   422 short/mismatch, success → login.
2. **Manual fallback missing date + image URL** — export `new.html` manual panel had
   title/source/body only; backend requires `published_at` + `main_image_url`
   (image is mandatory).
   Resolution: applied directly to `screens/new.html` manual panel — added
   `mDate` (date, optional) + `mImg` (url, required, `needs_image` 422 if empty),
   wired into the prototype `useManual` handler (stores `published` + `img`).
3. **Status naming drift** — dashboard seed uses `gen_failed`; Post enum has only
   `failed`. Failure 6.8% seed exceeds the <5% KPI (demo data, not a contract).
   Resolution: stage-scoped enums documented in spec —
   `Article.fetch_status: ok|fetch_failed`,
   `Generation.status: ready|gen_failed|needs_image|in_review|approved|publish_failed`,
   `Post.status: draft|in_review|queued|posted|failed|token_expired`.
   Dashboard `gen_failed`/`fetch_failed` rows are generation/article-stage failures,
   valid. Seed numbers are placeholders; KPI (<5% publish failure) measured on Post.

## Nits (accepted, tracked)

4. Review cost pill shows tokens + $ only — `fetch_tier`/`fetch_cost_usd` surface in
   dashboard row detail + generation GET response, not in the pill. No UI change.
5. `needs_image` 422 lives as a note on New only — accepted; blocked-generate state
   reuses the fetch-steps error list, no new empty-state design.
6. Login demo accepts any email + 8 chars — prototype-only (`localStorage` session).
   Backend enforces real credentials + 429 + 401. Must not survive into code;
   asserted in auth ticket tests.
