# Brand Design System

Tokens, components, patterns and charting conventions **extracted** from two production FP&A products: **Forecast** (82 files, ~22.7k lines) and **RM Pro Forma**.

**Extracted, not authored.** Everything here documents what the code *does* today, with `file:line` evidence from the source application. Nothing in this repo is imported by the running app, and building it changed no application source. Where the app is inconsistent, the inconsistency is documented in `FINDINGS.md` rather than quietly smoothed over — 28 findings, with severity and evidence.

That is the point of the repo. A design system that only describes the intended state is a wish; this one describes the actual state and names the gap.

---

## Start here

| If you want to… | Read |
|---|---|
| See it rendered, in both themes | **`gallery.html`** — [how to view](#viewing-the-gallery) |
| Know what a token is called and how to consume it | **`FRAME.md`** |
| Know what to build and which rules to follow | **`DESIGN.md`** |
| Know what's already broken before you change something | **`FINDINGS.md`** |

---

## The layering

```
  TOKENS          49 custom properties                     FRAME.md
    │             shadcn semantic (20) · chart (13) · Brand surface (10) · component-scoped (6)
    ▼
  PRIMITIVES      brand-*  (29 selectors)                    COMPONENTS.md §B
    │             the de-facto SHARED layer — token-backed, used by both products
    │             components/ui/ (10 React exports, Radix + Tailwind)
    ▼
  COMPONENTS      forecast-*  (249 selectors)              COMPONENTS.md §A
    │             Forecast product vocabulary
    │             Tailwind utility compositions            COMPONENTS.md §C/§D
    │             RM Pro Forma product vocabulary
    ▼
  PATTERNS        10 composite layouts                     PATTERNS.md
                  panel blocks, dense matrices, stage flows, distribution tracks
```

**FRAME is the primitive layer; DESIGN is the application layer.** FRAME says what exists and what it is called. DESIGN says what to do with it — how to compose, when to reach for a shared primitive versus invent a product class, and what not to do.

**The load-bearing fact:** the two products barely share a vocabulary. Forecast uses 1,846 `forecast-*` classNames; RM Pro Forma uses **zero**. They meet only at the token layer and the `brand-*` primitives. That is not a bug to fix today, but it is the thing to know before touching either (`FINDINGS.md` F-09).

---

## The theme model

**There is no `.dark` class.** Themes are body-scoped:

| Selector | Role | Tokens |
|---|---|---|
| `:root` | Base scaffolding, light values. **No `--brand-*` family** | 33 |
| `body.brand-hub-theme-active` | Theme A / light | 43 |
| `body.brand-hub-theme-active[data-direction='B']` | Theme B / dark | 43 |

Two consequences that bite, both covered in detail in `FRAME.md`:

1. **Anything rendering outside `body.brand-hub-theme-active` loses every `--brand-*` token** — including `--brand-line`, the app's default border and second-most-used token overall (179 uses). Borders, panel fills, input backgrounds and shadows vanish simultaneously and silently.
2. **Colour tokens are bare HSL channels, not colours.** `hsl(var(--primary))` is correct; `var(--primary)` renders nothing. The `--brand-*` family is the exception — those are complete values, consumed bare.

---

## Contents

| File | What it is |
|---|---|
| `FRAME.md` | **The token layer.** 49 properties across four families, the three-block theme architecture, both consumption conventions, and the 11 phantom tokens marked do-not-use |
| `DESIGN.md` | **The application rules.** How to compose token → primitive → component → pattern; when to reuse vs invent; the 10 patterns; chart rules; a cited Do/Don't list |
| `COMPONENTS.md` | The catalog — 15 Forecast components, 7 shared `brand-*` primitives, the `ui/` layer, 5 RM Pro Forma components |
| `PATTERNS.md` | 10 composite patterns — how the components compose into screens |
| `CHARTS.md` | Recharts conventions, the `--chart-*` palette, the 8 chart components, a checklist for new charts |
| `FINDINGS.md` | 28 drift findings with severity and `file:line` evidence. **Read before changing anything** |
| `tokens.css` | The 49 custom properties, grouped and commented, both themes |
| `tokens.json` | Machine-readable token export with usage counts, theme values, and status flags |
| `gallery.html` | Standalone live specimen page — every token resolved via `getComputedStyle`, both themes |
| `styles/index.snapshot.css` | Vendored snapshot of the app stylesheet so the gallery works standalone — see below |

---

## Viewing the gallery

`gallery.html` links a relative stylesheet, and Chrome blocks that over `file://`. **Serve it:**

```bash
python3 -m http.server 8000
# then open http://localhost:8000/gallery.html
```

Toggle **A · Light** / **B · Dark** in the header to switch themes. Token swatches are read live from the browser via `getComputedStyle` — they are whatever the stylesheet actually computes, not values typed in twice.

Two things the page honestly cannot show, both flagged on it:

1. **Tailwind utilities do not resolve.** `index.css` contains `@tailwind` directives that only expand through the PostCSS build. Loaded raw, no utility class works — so **RM Pro Forma specimens render structurally but unstyled**. The Forecast and `brand-*` specimens are hand-written CSS and render exactly as in the app.
2. **Webfonts need network.** Fonts load from Google Fonts via `@import`; offline, everything falls back to system stacks.

---

## The stylesheet snapshot

`styles/index.snapshot.css` is a **byte copy** of the source application's `frontend/src/index.css`, taken **2026-07-20** from commit **`e0a75cb`** (branch `claude/close-intel`).

The source application is a private repo and is not vendored here. So:

> **This snapshot can drift from the running app, and nothing in this repo will warn you when it does.** Treat a gallery specimen as evidence about the snapshot, not a guarantee about production.

Refreshing is a copy **plus a rename pass.** This repo normalises the source app's `ace-`/`aa-` prefixes to
`brand-` (see [Naming provenance](#naming-provenance)), and `gallery.html` references the *renamed* identifiers.
A bare `cp` silently reverts the stylesheet to `ace-*` — the gallery still loads, but every specimen loses its
borders, panel fills, inputs and shadows at once, and nothing errors. Copy **and** rewrite, from the repo root:

```bash
cp /path/to/BankAnalysis/frontend/src/index.css styles/index.snapshot.css && \
perl -pi -e '
  s/(?<![\w-])ace-brand(?![\w])/brand/g;
  s/--aa-ink(?![\w-])/--brand-ink/g;
  s/--aa-accent(?![\w-])/--brand-accent/g;
  s/--ace-/--brand-/g;
  s/(?<![\w-])ace-/brand-/g;
' styles/index.snapshot.css
```

**The `(?<![\w-])` lookbehind is load-bearing.** A naive `sed 's/ace/brand/g'` corrupts every English word that
contains "ace" — `space`, `spacing`, `replace`, `interface`, `workspace`, `surface`, `placement`, `namespace` —
and the snapshot alone contains 286 of them. Rule order matters too: `--ace-` is rewritten before the general
prefix rule, and `ace-brand` before both, so `.ace-brand-mark` becomes `.brand-mark` rather than
`.brand-brand-mark`.

Verify any refresh with these two greps:

```bash
# 1. corruption check — must print nothing
grep -nE 'spbrand|spbranding|surfbrand|replbrand|interfbrand|workspbrand|plbrandment' styles/index.snapshot.css
# 2. leftover check — must print 0
grep -cE -- '--ace-|\.ace-|(^|[^A-Za-z])ace-' styles/index.snapshot.css
```

Same applies to `tokens.css` and `tokens.json` — they are mirrors. The app's `index.css` is canonical; editing anything here changes nothing in the app.

**After refreshing the snapshot,** re-run the checks in `FRAME.md` §12: re-extract token definitions, re-check that every `var(--x)` has a matching `--x:`, and open the gallery in both themes to confirm every specimen still renders.

---

## Naming provenance

**This repo renames the source application's brand prefixes; the source application does not.**

| Source app (`index.css`) | This repo |
|---|---|
| `--ace-*` | `--brand-*` |
| `.ace-*` (`ace-input`, `ace-tooltip`, `ace-scroll-shell`, …) | `.brand-*` |
| `body.ace-hub-theme-active` / `ace-hub-theme` | `body.brand-hub-theme-active` / `brand-hub-theme` |
| `.ace-brand`, `.ace-brand-mark`, `.ace-brand-title`, … | `.brand`, `.brand-mark`, `.brand-title`, … |
| `--aa-ink`, `--aa-accent` | `--brand-ink`, `--brand-accent` |

Nothing else was renamed: `forecast-*`, `--chart-*`, `--radius`, the shadcn semantic tokens and the product names
(BankAnalysis, Forecast, RM Pro Forma) are untouched.

**What this means for the citations.** Every `file:line` reference in `COMPONENTS.md`, `PATTERNS.md`, `CHARTS.md`
and `FINDINGS.md` points into the source application, where the identifier at that line still carries its original
`ace-`/`aa-` prefix. The line numbers are accurate; only the prefix differs. The citations have deliberately **not**
been rewritten — stating the mapping once here keeps them honest without pretending the source app uses `brand-`.

**One merge worth knowing about.** The source has two distinct names that both normalise to `--brand-ink`: the
phantom `--ace-ink` (referenced, never defined, always falls back to its inline hex) and the scoped alias `--aa-ink`
(defined on the brand mark). They do not collide in practice — `--brand-ink` is only *defined* inside `.brand-mark`,
the 22×33px logo, and its only consumer, `.forecast-notif-badge`, never renders inside it. That consumer still falls
back to `#14261c` exactly as it did before the rename.

---

## Scope & limitations

Read this before treating anything here as complete.

- **Extracted from two products only** — Forecast and RM Pro Forma. It is not a whole-application design system, and surfaces outside those two are not represented.
- **Deep markup analysis sampled 17 of 82 Forecast files.** Frequency rankings and component counts are repo-wide across the full 82-file directory; the markup snippets and structural claims come from the 17-file sample.
- **`ForecastWorkspaceShell` is a documented gap.** It provides the outer nav chrome for the workspace flow (imported at `ForecastProjectWorkspace.js:8`). Its source fell outside the sampled set and the classes it would own show zero occurrences in sampled files. **UNVERIFIED** — the nav chrome layer has no documented rules.
- **`modules/close-intel/closeIntel.css` (2,185 lines) is a third styling vocabulary and is not catalogued here.** It was treated as secondary reference only.
- **RM Pro Forma coverage is thinner than Forecast coverage**, proportionate to the source: two files of Tailwind composition against Forecast's 82 files and 249 CSS selectors.
- **Nothing in `FINDINGS.md` has been fixed.** It is a documentation deliverable; the application source is untouched. Recommendations are recommendations.
- **No license yet.** This repo does not carry a LICENSE file — one has not been chosen.

---

## Provenance

Extracted 2026-07-20 from branch `claude/close-intel`, commit `e0a75cb`. Source of every claim: `frontend/src/index.css` (5,581 lines), `frontend/tailwind.config.js`, `frontend/src/pages/forecasting/` (82 files), `frontend/src/pages/RMProForma_Dashboard.js`, `frontend/src/pages/RegionalForecastingModule.js`, `frontend/src/components/charts/` (8 files), `frontend/src/components/ui/` (3 files), and the root `RM_Pro_Forma_Color_Mapping.md` (reconciled in `FINDINGS.md` F-24).
