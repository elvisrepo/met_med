# Guardrails — user harness for Insta AI MVP

Model: Böckeler/Fowler, *Harness engineering for coding agent users* (Apr 2026).
Vocabulary used exactly: **guides** (feedforward, steer before acting) vs
**sensors** (feedback, observe after acting) × **computational** (deterministic,
CPU, cheap) vs **inferential** (LLM-judged, GPU, expensive). Goal of the
harness: raise first-try correctness *and* self-correct before human eyes, so
human input goes where it matters most.

Status: mostly specified, barely installed — no code, CI, or hooks exist yet.
Install order is staged by ticket below. Rule of thumb from the article:
computational everywhere it's possible, inferential only where semantic
judgment is required — and never ask the model to remember what a machine can
check.

## 1. Regulation categories (what we govern)

### Maintainability — easiest, mostly computational

- Guides: `AGENTS.md` style rules, Clean tokens (`design-tokens.md`).
- Sensors (pre-commit/PR): ruff + black, eslint + prettier, `mypy --strict` on
  the backend service layer (untyped Django is where comprehension debt breeds),
  Semgrep (generic + custom rules for *our* sins: secrets in frontend bundles,
  raw article HTML rendered, publish path missing the `in_review` check),
  dead-code detection (`vulture`), coverage *quality* (not just %).
- Verdict on the TDD question: Fowler's warning stands — don't over-trust
  AI-generated tests. We test at the `generate_post` seam with hand-written
  cases plus **approved fixtures** (golden article → caption/image-prompt pairs
  per category); prompt changes must keep fixtures green or justify the diff.
- Model fixtures double as the model benchmark (ADR-0010): the fixture set must
  contain real Albanian articles, Standard plus one Gheg sample. Swapping the
  text/image model id = config change + full fixture run + weekly-sampling
  watch for two weeks. Pin reasoning/thinking off for summarize/translate calls
  (reasoning tokens are pure tax here).

### Architecture fitness — fitness functions, not vibes

- Guides: ADRs (accepted = law), module map (5 Django apps), OpenAPI contract.
- Sensors: `infra/arch-boundaries.yaml` + import check in CI (ArchUnit
  equivalent — fails the build on violation), c4-diff diagram on backend PRs
  (where to look), p95 generation-time assertion (<90s) as a performance
  fitness function, `openapi-diff` on contract PRs.

### Behaviour — the elephant; weakest link by default

- Guides (feedforward): the spec + change specs + OpenAPI input/output/error
  contracts with examples. Spec quality caps everything — no sensor rescues a
  vague spec.
- Sensors: seam tests with fake LLM/IG adapters, Playwright for the 5-route
  flow, approved caption fixtures per category (selective, where it fits —
  not wholesale), and the human 10-minute exercise as the main sensor.
- LLM-output quality (we *have* LLM integration, so this applies): sample
  production generations weekly against a rubric (tone per category, no faces
  rule, source credit present) — inferential sensor, sampled, never per-commit.

## 2. Timing — keep quality left

| Stage | Runs | Contents |
|---|---|---|
| Pre-commit (seconds) | every commit | format (black/prettier), fast pytest subset (`-k` seam), boundaries check |
| Agent self-correction loop | during work | same fast sensors re-run by the agent until green before handoff |
| Human review | per ticket | Vibe Diff (plain language) + c4-diff spicy modules first, lines second |
| PR pipeline (minutes) | per PR | full pytest + Playwright, eslint, `pip-audit`/`npm audit`, Semgrep, coverage, `openapi-diff`, c4-diff comment, stale-`openapi.yaml` failure |
| Expensive pipeline (later) | nightly/main | mutation testing (publishing only), `/detailed-review` agent pass |
| Continuous drift (post-launch) | always-on | Dependabot, dead code, Sentry error rate, dashboard SLOs (p95, $/post, failure %) as runtime sensors, weekly LLM-output sampling |

Slow/flaky checks never live in hooks — a bypassed hook is worse than none.
What must not be skipped lives in CI.

## 3. Steering loop (operating procedure, not a document)

When the agent repeats a mistake (the "no, not like that" signal): encode it
once, at the cheapest level that holds — AGENTS.md rule → skill/recipe →
structural test → custom Semgrep rule. Use the agent to draft the control
(draft rules from observed patterns, scaffold the linter), human approves it.
Each encoding should make that mistake class less probable next time; that is
the loop working.

## 4. Harnessability (bake it in — we're greenfield)

Choices that keep this codebase governable: typed service layer, 5 explicit
Django apps with banned-import rules, OpenAPI-first routes, thin handlers,
tokens as the only styling source. Our scaffold *is* the harness template
(topology: Django monolith + Vite SPA + Compose) — future services copy it and
inherit the guides and sensors. Legacy gets the harness it deserves; we don't
have legacy yet, keep it that way.

## 5. Human role (what machines don't get)

No sensor reliably catches misdiagnosed issues, overengineering, or
misunderstood instructions — and correctness is outside every sensor's remit
if the spec was vague. Humans keep: spec authorship, Vibe Diffs, 10-minute
exercises, system walks (trace one post end-to-end, explain each layer), and
the final approve. The harness doesn't eliminate human input; it directs it at
the surprises.

## 6. Install plan (tied to tickets)

- Ticket 1 (scaffold): pre-commit (format+fast tests+boundaries file),
  CI skeleton (pytest, eslint), `openapi.yaml` generation.
- Tickets 2–6: Semgrep (+2 custom rules), audits, coverage gate on
  `publishing/`, c4-diff job, Playwright flow.
- Post-launch: mutation testing, dead-code + Dependabot, SLO dashboards as
  sensors, weekly LLM-output sampling, first system walk.

## Open questions (review quarterly)

- If sensors never fire: high quality or inadequate detection? (Add a
  canary violation test per sensor that must fail.)
- Are guides and sensors contradicting each other anywhere? (Audit on drift.)
- Which inferential sensor earns its cost, and which should become
  computational? (Review token spend vs catches.)
