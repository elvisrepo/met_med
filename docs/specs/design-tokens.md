# Design tokens (frozen from OpenDesign export 2026-09-03)

Source: identical `:root` block in all 5 files under
`docs/Insta-AI-MVP-System-Design/screens/`. Extract before writing React
components — no framework-default colors/typography.

## Color

| Token | Value | Use |
|---|---|---|
| `--bg` | `#ffffff` | Page background |
| `--surface` | `#f7f7f7` | Panels, sidebar |
| `--surface-warm` | `#eeeeee` | Borders-soft fill |
| `--fg` | `#111111` | Text, accent (mono black system) |
| `--fg-2` | `#3a3a3a` | Secondary text |
| `--muted` | `#707070` | Hints, eyebrows, counts |
| `--meta` | `#111111` | Meta text |
| `--border` | `#d9d9d9` | Card/panel borders |
| `--border-soft` | `#eeeeee` | Table row dividers |
| `--accent` | `#111111` | Primary buttons, active nav |
| `--accent-on` | `#ffffff` | Text on accent |
| `--success` | `#168a46` | posted / ok pills, done dots |
| `--warn` | `#b7791f` | token_expired / invited pills |
| `--danger` | `#c53030` | failed pills, errors |
| `--accent-hover` | `color-mix(accent, black 8%)` | Button hover (modern browsers only) |
| `--accent-active` | `color-mix(accent, black 14%)` | Button pressed |
| `--focus-ring` | `0 0 0 3px rgba(17,17,17,.18)` | Visible focus on all inputs/buttons |

## Type

Display + body: `Inter, system-ui, sans-serif`. Mono: `"SF Mono", ui-monospace, Menlo, monospace`
(pills, eyebrows, table headers, numbers, costs).
Scale: xs 12 · sm 14 · base 16 · lg 18 · xl 24 · 2xl 36 · 3xl 54 · 4xl 76.
Body leading 1.52, tight 1.06, display tracking -0.025em.

## Spacing / radius / elevation / motion

Space: 1:4 · 2:8 · 3:12 · 4:16 · 5:20 · 6:24 · 8:32 · 12:48.
Section-y: desktop 96 · tablet 68 · phone 48. Container max 1180px,
gutters 36/24/16.
Radius: sm 4 · md 8 · lg 12 · pill 9999.
Elevation: flat none · ring `0 0 0 1px border` · raised `0 16px 40px rgba(0,0,0,.10)`.
Motion: fast 150ms · base 240ms · `cubic-bezier(0.2,0,0,1)`.

## Layout contract

Sidebar 232px → top-bar under 900–980px. Min touch target 44px
(36px table row buttons). Review grid `1fr 380px` → 1fr under 980px.
Stats 4-col → 2-col mobile. No horizontal scroll at 360–1920px.
