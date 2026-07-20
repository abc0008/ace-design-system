# DESIGN — the application rules

**FRAME says what exists. DESIGN says what to do with it.** This file is prescriptive: how to compose a screen, when to reach for a shared primitive versus invent a product class, which of the ten patterns applies, and the rules for charts. Where the app is already inconsistent, this file picks a side and says so.

Read `FRAME.md` first — the theme rule in FRAME §2.1 and the two consumption conventions in FRAME §3 are assumed here.

---

## 1. The layering

```
  TOKENS          49 custom properties                     FRAME.md
    │             the vocabulary of values
    ▼
  PRIMITIVES      brand-*  (29 selectors)                    COMPONENTS.md §B
    │             the de-facto SHARED layer — both products, token-backed
    │             components/ui/ (10 React exports, Radix + Tailwind)
    ▼
  COMPONENTS      forecast-*  (249 selectors)              COMPONENTS.md §A
    │             Forecast product vocabulary — hand-applied CSS classes
    │             Tailwind utility compositions            COMPONENTS.md §C/§D
    │             RM Pro Forma product vocabulary
    ▼
  PATTERNS        10 composite layouts                     PATTERNS.md
                  panel blocks, dense matrices, stage flows, distribution tracks
```

Each layer may consume the layer above it. **A pattern must not reach past the primitive layer to a raw hex**, and a product component must not redefine something the primitive layer already provides.

### 1.1 The load-bearing fact (F-09)

Forecast and RM Pro Forma **barely share a vocabulary.**

| | Forecast | RM Pro Forma |
|---|---|---|
| `forecast-*` classNames | **1,846** | **0** |
| Distinct `forecast-*` selectors | 249 | — |
| `brand-*` classNames | 24 | 16 |
| Styling approach | hand-written CSS classes | `components/ui/` + Tailwind utilities |
| Inline `style={{}}` | 194 | 8 |

They share exactly three things: **the token layer, the `brand-*` primitives, and `card-wrapper`.** Everything else is parallel invention.

This is not a bug to fix this quarter, but it is the thing to know before touching either product. It also sets the direction: **`brand-*` is where consolidation goes.** All 29 of its selectors are token-backed, none contains a hard-coded colour, and it is the only layer both products already speak.

---

## 2. Composing a screen

### 2.1 Which vocabulary am I in?

Decide this first; it determines everything downstream.

**Building in Forecast** (`pages/forecasting/`) — you are writing **CSS classes, applied by hand**. There is no React component library for the `forecast-*` vocabulary; the classes live in `index.css` and you type them into `className`. Start from pattern **P1** and pull atoms from `COMPONENTS.md` §A.

**Building in RM Pro Forma** — you are **composing `ui/` primitives with Tailwind semantic utilities** (`text-muted-foreground`, `bg-card/75`), not writing CSS. Start from pattern **P10** and `COMPONENTS.md` §C/§D.

Do not mix the two in one screen. A Forecast panel built from Tailwind utilities, or an RMPF card built from `forecast-*` classes, is how the third vocabulary gets born.

### 2.2 When to reach for `brand-*` vs invent a product class

**Reach for an existing `brand-*` primitive when the thing you need is a surface, a control chrome, or a scroll/overlay behaviour.** That is what the layer covers:

| Need | Use | Not |
|---|---|---|
| Scrolling container | `.brand-scroll-shell` | a bespoke `::-webkit-scrollbar` block |
| Text input, select trigger | `.brand-input` | a Tailwind input recipe |
| Dropdown / portal menu | `.brand-floating-surface` + `.brand-floating-item` | a new popover surface |
| Chart or table tooltip | `.brand-tooltip` | an inline Recharts `contentStyle` object |
| Table header chrome | `.brand-table-header` | restyling `th` per table |
| Chart legend text | `.brand-chart-legend-text` | inline legend styles |
| Pill button | `.brand-button-primary` / `-secondary` / `-danger` | a new button class |

`.brand-floating-surface` is the one primitive genuinely shared by both products today — Radix `SelectContent`/`SelectItem` and RMPF's portal menu both land on it. Treat it as the model for what a good shared primitive looks like.

**Invent a product class only when all three are true:**

1. It is genuinely product-specific layout or semantics — not a surface, control, or overlay.
2. It will appear in **≥2 files**, or it is load-bearing structure. (That is the inclusion bar `COMPONENTS.md` already enforces, and it has a "considered and excluded" list at the bottom to keep it honest.)
3. **You have checked `FINDINGS.md` for a duplicate.** This is not optional politeness — the duplication is severe and well-mapped.

**The default answer to "should I add a new component?" is no**, because one of these is usually already it:

- **Four** implementations of the labelled-distribution track (F-21) — `consol-bridge`, `pulse-card`, `forecast-bullet-row`, `forecast-peer-distribution-row`. All four are `label → track → markers positioned by left:%`. None share a class.
- **Three** incompatible table implementations (F-10).
- **Two** stat-strip implementations sharing half their CSS (F-22).
- **Two** accordion mechanisms.
- A notification tray that reimplements the panel head from scratch (F-23).

If you are about to build a fifth distribution track, the correct move is to generalise one of the four, not to add to the pile.

### 2.3 Never invent a second vocabulary per product

Three vocabularies already exist: `forecast-*`, RMPF's Tailwind compositions, and `modules/close-intel/closeIntel.css` (2,185 lines, not catalogued here). A fourth is not free — it doubles the surface a designer has to hold in their head and guarantees that the next shared primitive gets written three times.

New product surface goes into the `brand-*` layer if it is reusable, or into an existing product vocabulary if it is not. It does not get its own namespace.

---

## 3. The ten patterns and when each applies

Full markup and evidence in `PATTERNS.md`. This is the routing table.

| Pattern | Reach for it when | Parts |
|---|---|---|
| **P1** Panel + head + body | **Always — the default.** Every content block in the product | `.forecast-panel` › `.forecast-panel-head` › (`.forecast-section-code` + `h2`) › body |
| **P2** Action bar + note | A panel accepts input and needs a decision footer | `.forecast-submit-chips`/`-actions` › buttons › `.forecast-submit-note` |
| **P3** Read table | Narrow tabular data — roster, log, summary — read-only, needs an empty state | `.forecast-home-table-wrap` › `table.forecast-home-table` |
| **P4** Dense matrix | Wide **editable** numeric grid; where forecasting actually happens | `.forecast-table-shell.brand-scroll-shell` › `table.forecast-table` |
| **P5** Workspace stage flow | A linear multi-step model builder, one stage at a time | stage nav › `.workspace-stage-panel` › `.forecast-accordion` › `.workspace-action-row` |
| **P6** Labelled distribution track | A label, a horizontal track, markers positioned by `left:%` | **four existing implementations — pick one, don't add** |
| **P7** Stat strip | A row of label/value pairs above the fold | `.forecast-status-strip` (span+strong) or `.forecast-kpi-grid` (span+strong+small) |
| **P8** Notification / audit list | Unread-aware item list in a popover, or its audit-trail table twin | `.forecast-notif-panel.forecast-panel` › `.forecast-notif-list` |
| **P9** AI prompt / narrative box | Plain-English input that drafts a change; four explicit states | `.forecast-prompt-box` › `.forecast-prompt-input` |
| **P10** RM Pro Forma entry workspace | Anything in RMPF | `.brand-rm-overview` hero › entry tab pills › `Card.card-wrapper` |

### 3.1 P1 is the spine — and it has a hard rule

Every screen is a stack or a grid of P1 panels (100 panel / 91 head / 112 section-code occurrences across 15 files).

> **A panel body is exactly one of four things:** a table (P3 or P4), a chart, a chip row (P2), or a stack of subpanels.

Nothing in the codebase does anything else. If your panel body is a fifth thing, you are either building a new pattern deliberately — document it — or you have the wrong container.

Variants: `.forecast-panel-lite`, `.forecast-panel-tall`, `.forecast-subpanel` (nested, `h3`), `.forecast-exec-pair` (two-up). When a panel sits under a section id like `#forecast-macro`, `--section-color` tints the section code and left border (`index.css:3760-3799`).

### 3.2 P3 vs P4 — pick by editability, not by width

Both are tables and they are otherwise unalike in every respect. **P3 is read-oriented** — a roster or log, needs an empty state, sign-coloured numerics via `.pos`/`.neg`, row actions as `.forecast-home-action.ghost`. **P4 is the dense editable grid** — always inside `.forecast-table-shell.brand-scroll-shell`, often `.forecast-compact-table`, columns carry `.time-col`/`.yt` classes.

Choosing P3 for an editable grid gets you a table with no scroll shell that breaks at width. Choosing P4 for a five-row summary gets you scroll chrome you do not need.

### 3.3 P2's naming drift — pick the semantic one

`.forecast-submit-actions` and `.forecast-submit-chips` are used interchangeably for the same job (F-13); only `RegionalContextStage.js` reaches for the semantic one. **Use `.forecast-submit-chips` for a row of chips and `.forecast-submit-actions` for a row of buttons.** Do not perpetuate the coin-flip.

### 3.4 P6 — the strongest idea and the worst duplication

Four implementations of one concept, none sharing a class. Before writing a fifth, pick from:

| Implementation | Shape | Where |
|---|---|---|
| `consol-bridge` | diverging gap bars | `ForecastReviewConsole.js` (20 occurrences, 1 file) |
| `pulse-card` | progress meter | — |
| `forecast-bullet-row` | guardrail bullet chart | — |
| `forecast-peer-distribution-row` | peer range dotplot | — |

---

## 4. Chart rules

Full detail in `CHARTS.md`. These are the rules that decide whether a chart survives a theme switch.

**Library: Recharts `^2.5.0`.** 16 imports across the app, no alternative. There is **no shared chart abstraction** — no `useChartTheme` hook, no `<ChartCard>`, no base component, no shared loading/error/empty state. Every convention below is honoured by copy-paste, not enforced by code. That means nothing stops you getting it wrong, and nothing will tell you that you did.

### 4.1 The rule that breaks dark mode

> Set the axis **`stroke`**, the **`tick.fill`**, *and* the grid stroke — all three, from `--chart-axis` / `--chart-grid`.

Omitting any of them is what breaks dark mode, and it fails in the direction nobody catches: the chart looks fine in light, and the axis labels go near-invisible in dark. `ForecastNiiBuildup.js` is the standing example.

```jsx
<CartesianGrid strokeDasharray="3 3" stroke="hsl(var(--chart-grid))" />
<XAxis dataKey="period" stroke="hsl(var(--chart-axis))" tick={{ fill: 'hsl(var(--chart-axis))', fontSize: 12 }} />
<YAxis stroke="hsl(var(--chart-axis))" tick={{ fill: 'hsl(var(--chart-axis))', fontSize: 12 }} />
```

`--chart-axis` is the most-consumed chart token (33 uses) precisely because it has to be written three times per chart.

### 4.2 The two prop conventions — and which to standardise on

Two conventions coexist. **Match the one where your chart lives**; do not import a third.

| | **Convention A** — `components/charts/*` | **Convention B** — `pages/forecasting/stages/*` |
|---|---|---|
| Colour source | inline `hsl(var(--chart-N))` | `chartAxisColor` / `chartGridColor` consts from `vm` |
| Grid dash | `"3 3"` | `"3 5"` |
| Grid direction | both axes | `vertical={false}` |
| Tick font size | 12 | 10–11 |
| Legend | `formatter` + `.brand-chart-legend-text` | `wrapperStyle` inline |
| Tooltip | per-file `contentStyle` object | shared `tooltipStyle` from `vm` |
| Line weight | `strokeWidth={3}` primary | `strokeWidth={2}` |

**Standardise on Convention A's colour handling and Convention B's theme threading.** Concretely, for new work:

- **Take from A:** inline `hsl(var(--chart-*))` for series colours. Series colour belongs at the mark, where you can read it.
- **Take from B:** a single source for axis/grid/tooltip theme values rather than re-declaring `axisTickStyle` per file. The `axisTickStyle` / `legendFormatter` pair is currently copy-pasted verbatim into **5 of 6** chart files — that is the duplication B already solved.
- **Take from neither:** the divergent grid dashes and tick sizes. `"3 3"` and font-size 12 are the majority; use them.

The `vm` bag (F-20) is a deliberate pattern with a real benefit — stages import nothing and tests stub the whole slice in one line — but it is a ~30-key untyped contract with no compile-time checking, and it has **already been partially escaped**: `ForecastNiiBuildup.js` renders inside a `vm` stage but imports Recharts directly and takes explicit props. If you add a stage chart, thread it through `vm` or do not use a stage.

### 4.3 Up / down / neutral semantics

**One pair. Nothing else:**

```jsx
// marks in a chart
<Cell fill="hsl(var(--chart-up))" />
<Cell fill="hsl(var(--chart-down))" />

// alpha blending is free
<Cell fill={`hsl(var(--chart-up) / ${alpha})`} />
```

```css
/* signed numbers as text, in a table cell */
color: var(--brand-positive);
color: var(--brand-negative);
```

**Charts get `--chart-up`/`--chart-down`; text gets `--brand-positive`/`--brand-negative`.** They are close but not identical, and the split is intentional — the text pair is tuned for legibility at small sizes on a panel fill, the chart pair for area and stroke against a grid.

The three systems you must **not** use: the ad-hoc `GREEN`/`RED` JS constants (six files, cannot respond to theme), the dead Tailwind `icon.*` palette, and any raw hex.

Note the existing asymmetry so you recognise it rather than copy it: `.forecast-home-table .pos` uses `--chart-up`, but `.neg` uses `--destructive` rather than `--chart-down` (`index.css:2871-2874`). Safe only because those two tokens are byte-identical. Do not rely on that.

**There is no live neutral.** `--chart-neutral` is defined and dead (0 uses). If you need a neutral series, `--chart-5` is the live grey — and it is the one series colour that inverts across themes.

### 4.4 Tooltips — use the class, not the object

Three tooltip implementations exist. **Prefer the custom-content one:**

```jsx
<Tooltip content={customTooltip} />   // renders <div className="brand-tooltip">
```

`.brand-tooltip` (`index.css:495-501`) is token-backed and theme-safe. The inline `contentStyle` objects duplicate that styling by hand, in two mutually inconsistent ways (`backgroundColor` + `borderRadius:'10px'` versus `background` + `'12px'`).

### 4.5 Chart checklist

`CHARTS.md` ends with a 10-point checklist. The two that actually break things:

1. Set `stroke` **and** `tick.fill` **and** the grid stroke from `--chart-axis`/`--chart-grid` (§4.1).
2. **Never** use the `GOLD`/`GREEN`/`RED` JS constants.

Also: strip `console.log` from render paths before committing (F-25 — there are some in there now).

---

## 5. Colour and tone rules

### 5.1 Picking a colour

**Check `tokens.json` first. If you are reaching for a hex, you are almost certainly duplicating a token.** `#FFAC03` appears in six files and is just `--primary`.

The system currently contains five distinct greens and six distinct reds (FRAME §10). Every one added makes that worse.

### 5.2 The chip tone formula

Every `.forecast-health-chip` tone variant follows one formula. Copy it rather than inventing a colour:

```css
background: color-mix(in srgb, hsl(var(--token)) 13-16%, transparent);
border-color: color-mix(in srgb, hsl(var(--token)) 38-44%, transparent);
```

`.forecast-health-chip` is the workhorse atom — **186 occurrences, the single most-used class in the product**, doing four jobs (status badge, metric tag, filter toggle, nav link).

### 5.3 ⚠ The chip tone names lie (F-28)

This is a High finding and it is the one most likely to bite a new contributor, because the failure is semantic rather than visual — the chip renders *a* colour, just not the one you asked for.

| You write | It renders | Token behind it |
|---|---|---|
| `.green` | **amber** | `--primary` |
| `.moss` | **amber** — byte-identical to `.green` | `--primary` |
| `.gold` | amber (16% vs 13%) | `--primary` |
| `.yellow` | **olive green** | `--accent` |
| `.amber` | **olive green** — byte-identical to `.yellow` | `--accent` |
| `.red` | terracotta ✓ | `--destructive` |
| `.neutral` | 4% foreground wash ✓ | `--foreground` |

Seven names, **four of which describe a colour the chip does not render**, and only **four distinct renderings** across all seven.

Why it matters beyond tidiness: writing `className="forecast-health-chip green"` to signal *healthy* produces an **amber** chip — which in this product's own visual language reads as *caution*, because amber is `--primary` and every "watch"/"in review" state uses it. Writing `yellow` for a warning produces a **green** chip that reads as healthy. **The two most semantically loaded tones are effectively swapped at the call site, across 186 of them.**

> **Rule until this is fixed: use only `.red` and `.neutral`, which mean what they say. For any other tone, read F-28 and choose by the token, not the name.**

The recommended fix is a mechanical rename to token-derived or status-semantic names (`.tone-primary`/`.tone-accent`/`.tone-destructive`/`.tone-neutral`, or `.ok`/`.watch`/`.breach`), collapsing the duplicate pairs, in **one commit across all call sites**. Leaving old and new names side by side is exactly how this happened.

### 5.4 Borders — `--brand-line` or `--border`?

Both are live and they answer the same question in different layers.

- **Hand-written CSS → `var(--brand-line)`.** It is the app's default border, 179 uses, and it is what every `forecast-*` and `brand-*` surface uses.
- **Tailwind utilities → `--border`**, reached via `border-*`. You get this automatically; do not fight it.

Do not reach across: writing `border: 1px solid hsl(var(--border))` in hand-written CSS is off-system, and so is a Tailwind arbitrary value pointing at `--brand-line`.

---

## 6. Do / Don't

Drawn from `FINDINGS.md`. Each entry cites the finding.

### Do

- **Do wrap HSL-channel tokens in `hsl()`** and consume `--brand-*` bare. (FRAME §3)
- **Do set `class="brand-hub-theme-active"` on `<body>`** in any harness, portal root, print or export path that renders these components — plus `data-direction` on both `<html>` and `<body>`. (F-06)
- **Do use one up/down pair:** `--chart-up`/`--chart-down` for marks, `--brand-positive`/`--brand-negative` for text. (F-02)
- **Do check `FINDINGS.md` for a duplicate before adding a component.** Four distribution tracks, three table conventions, two stat strips already exist. (F-21, F-10, F-22)
- **Do reuse `brand-*` primitives** for surfaces, controls, overlays and scroll chrome — it is the only layer both products share. (F-09)
- **Do use `.brand-tooltip`** rather than an inline Recharts `contentStyle`. (CHARTS §Tooltips)
- **Do put `aria-pressed` on segmented controls** — half of them are missing it today. (F-14)
- **Do rename across all call sites in one commit** when you fix a naming defect. (F-28)
- **Do follow the mono formula** — 10–11px, uppercase, `letter-spacing` 0.06–0.12em — for section codes, chips, and table headers. (FRAME §9)

### Don't

- **Don't hard-code a hex.** 17 files under `pages/forecasting/` already do, and the highest offenders carry 6–7 each. Check `tokens.json` first. (F-04)
- **Don't use the `GOLD`/`GREEN`/`RED` JS constants.** Redeclared in six files, and the only colours in the product that cannot respond to the theme. (F-02)
- **Don't write a `var()` fallback for a phantom token.** All 11 resolve to their fallback in both themes — `var(--brand-chart-2, #6b7c93)` *looks* theme-aware in review and is not. It is a hard-coded hex wearing a costume. (F-01)
- **Don't define a phantom token to "fix" it.** Defining `--brand-chart-2` instantly re-colours six charts. Decide per token: map deliberately, or drop the wrapper and keep the literal. (F-01)
- **Don't use `var(--brand-shadow-soft)`.** It is undefined *and* written with no fallback at `index.css:4888, 4927`, so the declaration is dropped and those elements have no shadow at all. Use `var(--brand-shadow)`. (F-01)
- **Don't invent a second vocabulary per product.** Three already exist; a fourth guarantees the next shared primitive gets written four times. (F-09)
- **Don't use `.forecast-chip-button`.** Used in three files, defined in zero — it renders as an unstyled browser-default button. That is a visible defect, not a tidiness issue. (F-19)
- **Don't trust chip tone names.** `.green` renders amber; `.yellow` renders green. Use `.red`/`.neutral` or read F-28. (F-28)
- **Don't expect `--radius` to control anything hand-written.** Panels are `14px`, cards `18px`, table shells `12px`, pills `999px`, all literal. (F-27)
- **Don't reach for `brand-*` or `icon.*` Tailwind utilities.** Both palettes are dead — zero usages — and `brand-*` additionally carries a 100-line shim. (F-03)
- **Don't restyle `ui/Card` with Tailwind and expect it to win.** `.card-wrapper` overrides `rounded-xl` and `bg-card` because the stylesheet beats the utility. (F-15)
- **Don't assume `ui/Button` gives you every variant.** It exposes one of three; `-secondary` and `-danger` require dropping to a raw `<button>`. (F-16)
- **Don't skip the dark block** when adding a token. It fails silently in exactly one theme. (FRAME §11)

---

## 7. Before you commit

1. **Does it render in both themes?** Toggle `data-direction` A → B and look. Most drift in this codebase is dark-mode-only.
2. **Does it render outside `body.brand-hub-theme-active`?** If the component can ever mount in a portal, print view, or test harness, check it there too. (F-06)
3. **Is every colour a token?** `grep` your diff for `#`.
4. **Did you add a class that already exists under another name?** Check `FINDINGS.md`.
5. **If it is a chart:** `stroke`, `tick.fill`, and grid stroke all set from tokens; no JS colour constants; no `console.log`.
6. **If it is a new pattern:** document it in `PATTERNS.md` rather than leaving it to be discovered as a sixth implementation of something.

---

## 8. Known gaps in these rules

- **`ForecastWorkspaceShell`** (imported at `ForecastProjectWorkspace.js:8`, rendered at `:691`) provides the outer nav chrome for the workspace flow. Its source fell outside the sampled set, and the classes it would own (`forecast-workspace-shell`, `workspace-command-header`, `workspace-stage-content`) show zero occurrences in sampled files. **UNVERIFIED** — the nav chrome layer has no rules here.
- **`modules/close-intel/closeIntel.css`** (2,185 lines) is a third styling vocabulary. Not catalogued, not governed by this file.
- **Rules for RM Pro Forma are thinner than for Forecast**, because RMPF is two files of Tailwind composition against Forecast's 82 files and 249 selectors. §2.1 and P10 are the whole of it.
