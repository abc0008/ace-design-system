# Findings — drift, inconsistency, and dead code

Every finding below is evidenced with `file:line`. **Nothing here has been fixed.** This is a documentation deliverable; the app's source is untouched. Recommendations are recommendations.

Severity: **High** = renders wrong, silently broken, or actively misleading · **Medium** = real inconsistency with maintenance cost · **Low** = tidy-up.

---

## Summary

| ID | Severity | Finding |
|---|---|---|
| F-01 | **High** | 11 phantom tokens — referenced via `var()`, defined nowhere |
| F-02 | **High** | Four conflicting up/down colour semantics |
| F-03 | **High** | An entire dead brand palette + its 100-line compatibility shim |
| F-04 | Medium | Hard-coded hex bypassing tokens in 17 files |
| F-05 | Medium | 4 dead chart tokens, 5 Tailwind-only tokens |
| F-06 | **High** | `--brand-*` family absent from `:root` — unthemed surfaces lose all borders |
| F-07 | Medium | Duplicate and triplicate CSS definitions |
| F-08 | Low | Declared-but-absent Avenir Pro font family |
| F-09 | **High** | Forecast and RM Pro Forma share almost no vocabulary |
| F-10 | Medium | Three incompatible table implementations |
| F-11 | Medium | `forecast-panel-head` has two incompatible child shapes |
| F-12 | Low | Section-code separator glyph is unstable |
| F-13 | Low | `submit-chips` vs `submit-actions` used interchangeably |
| F-14 | Medium | `aria-pressed` missing on half the segmented controls |
| F-15 | Medium | `card-wrapper` silently overrides `ui/Card`'s Tailwind classes |
| F-16 | Medium | `ui/Button` exposes only one of three button variants |
| F-17 | Medium | No `cn()` merge in `ui/index.js` and `ui/select.js` |
| F-18 | Low | Two icon libraries across three UI files |
| F-19 | **High** | `.forecast-chip-button` used in 3 files, defined in zero |
| F-20 | Medium | The `vm` prop bag is a ~30-key untyped contract, partially escaped |
| F-21 | Medium | Four implementations of the labelled-distribution-track pattern |
| F-22 | Low | Two stat-strip implementations that share half their CSS |
| F-23 | Low | Notification tray reimplements the panel head |
| F-24 | Medium | `RM_Pro_Forma_Color_Mapping.md` is comprehensively stale |
| F-25 | Low | `console.log` left in chart render paths |
| F-26 | Low | `.forecast-statement-matrix` fully styled, one usage |
| F-27 | Medium | `--radius` defined but ignored by hand-written CSS |
| F-28 | **High** | Chip tone names contradict the colours they render; 3 pairs are duplicates |

---

## F-01 · Phantom tokens — referenced but never defined · **High**

Eleven custom properties are consumed via `var()` in the app and defined in **zero** stylesheets (verified: `grep -rE '^\s*--<name>\s*:' frontend/src` returns 0 for each).

| Token | `var()` uses | Always renders | Notable sites |
|---|---|---|---|
| `--brand-red` | 9 | `#c4554d` | `ForecastProjects.js:307`; several `index.css` rules in the 4200-4400 band |
| `--brand-chart-2` | 8 | `#6b7c93` | `ForecastExecView.js:20`, `ForecastNiiBuildup.js:7`, `ForecastOp03Suite.js:19`, `ForecastReviewConsole.js:1259, 1281`, `ForecastReviewDetail.js:476, 511` |
| `--brand-moss` | 5 | `#5d8a66` | `ForecastReviewConsole.js:1284`; `index.css` pulse/bridge rules |
| `--brand-gold` | 3 | `#FFAC03` | `stages/CorporateProjectSpreadStage.js:422, 533, 658` |
| `--brand-danger` | 3 | `#e5241d` | `ForecastBulkActionBar.js:97`, `ForecastCommentThreadPanel.js:182` |
| `--font-mono` | 3 | inline stack | `index.css` (3 rules) |
| `--brand-chart-3` | 2 | `#9aa7b8` | `ForecastReviewConsole.js:1260, 1283` |
| `--brand-lust` | 2 | inline hex | `index.css` |
| `--brand-shadow-soft` | 2 | **nothing** | `index.css:4888, 4927` |
| `--brand-smokescreen` | 1 | inline hex | `index.css` |
| `--brand-ink` | 1 | inline hex | `index.css` |

**Why it matters:** `var(--brand-chart-2, #6b7c93)` *looks* theme-aware in review and is not. Every one of these resolves to its fallback in both light and dark. The `var()` wrapper is decorative.

**`--brand-shadow-soft` is worse** — it is written with **no fallback**:
```css
/* index.css:4888 and 4927 */
box-shadow: var(--brand-shadow-soft);
```
With the property undefined this resolves to the guaranteed-invalid value and the declaration is dropped. **Those two elements have no shadow at all and never have.**

**Recommend:** don't define these blind — defining `--brand-chart-2` would instantly change six charts. Decide per token: either (a) map to a real token and accept the visual change deliberately, or (b) delete the `var()` wrapper and keep the literal, which at least tells the truth. `--brand-shadow-soft` should be defined or the two rules deleted; that one is a straight bug.

---

## F-02 · Four conflicting up/down colour semantics · **High**

| System | Positive | Negative | Where | Live? |
|---|---|---|---|---|
| Chart tokens | `--chart-up` `#739666` olive | `--chart-down` `#D7653F` terracotta | `index.css:2871-2874, 3842-3843`; `RMProForma_Payback.js:69-71` | yes |
| Ad-hoc JS consts | `GREEN = '#2f9e6e'` | `RED = '#d9534f'` | 6 files (below) | yes |
| RM Pro Forma flash | `--accent` | `--destructive` | `RMProForma_Dashboard.js:221` | yes |
| Tailwind `icon.*` | `#02a88e` teal | `#e5241d` red | `tailwind.config.js:84-88` | **0 usages** |

`GOLD`/`GREEN`/`RED` are redeclared identically in `ForecastNiiBuildup.js:6-9`, `ForecastExecView.js:19-22`, `ForecastOp03Suite.js:18-20`, `ForecastGuide.js:11-15`, `stages/CorporateProjectProFormaStage.js:16-19`, and used as raw literals in `ForecastReviewDetail.js:476, 512` and `ForecastReviewConsole.js:1259-1284`.

**Distinct greens in use:** `#739666`, `#85aa76`, `#02a88e`, `#2f9e6e`, `#5d8a66` — five.
**Distinct reds:** `#d95f3d`, `#e6724f`, `#e5241d`, `#e61d24`, `#d9534f`, `#c4554d` — six.

**Also:** `GOLD = '#FFAC03'` is a hand-written literal of `--primary` (`hsl(40 100% 51%)` ≈ `#FFAB05`) — six files carry a dead copy of the brand colour, and none of them respond to the theme.

**Recommend:** one pair — `hsl(var(--chart-up))` / `hsl(var(--chart-down))`. Delete the JS constants; they are the only ones that can't respond to the dark theme.

---

## F-03 · A dead brand palette and its 100-line shim · **High**

`tailwind.config.js:91-104` declares a 12-colour `brand` palette (mulberry `#8f0f56`, lust `#e5241d`, hobgoblin `#02a88e`, caribbean-blue `#00bed5`, pigeon, white-smoke, nero, smokescreen, input-fill, tamahagane, mt-rushmore, lust-border).

`index.css:503-601` then contains ~100 lines of compatibility shim remapping every one of those utilities onto semantic tokens whenever `body.brand-hub-theme-active` is on:
```css
.brand-hub-theme .text-brand-mulberry,
body.brand-hub-theme-active .text-brand-mulberry { color: hsl(var(--primary)); }
```

**Usages of any `brand-*` utility in `frontend/src`: zero.** (verified by grep across all `.js`)

So: a dead palette, plus 100 lines of CSS shimming a palette nobody uses, plus a stale doc (F-24) describing the palette as current. Three layers of fiction.

The config's own comments compound it — `tailwind.config.js:79-80` asserts `--foreground` "is" NERO `#242424` and `--muted-foreground` "is" SMOKESCREEN `#595959`. The actual values are `0 0% 7%` (≈`#121212`) and `0 0% 38%` (≈`#616161`). Both comments are wrong.

**Recommend:** delete `brand`, `icon`, and `text.nero` from the Tailwind config and delete `index.css:503-601`. Verify with a build + grep first. This is the single largest dead-code removal available.

---

## F-04 · Hard-coded hex bypassing tokens · Medium

17 files under `pages/forecasting/` contain hard-coded hex. Highest counts: `ForecastReviewConsole.js` (7), `ForecastOp03Suite.js` (7), `stages/CorporateProjectSpreadStage.js` (6), `ForecastGuide.js` (6), `ForecastExecView.js` (6).

Representative:
- `ForecastGuide.js:11-15` — `GOLD #FFAC03`, `GREEN #2f9e6e`, `RED #d9534f`, `AMBER #e0a93c`, `BLUE #5b8dd6`
- `ForecastPromptBox.js:19, 286, 370` — `PURPLE #9C6ADE`, plus `color-mix(in srgb, #9C6ADE 14%, transparent)` inline
- `stages/CorporateProjectSpreadStage.js:260, 269` — sticky footer rows `background: '#fff'` and `'#f8f9fb'` — **these will render white-on-white in dark mode**
- `ForecastGuide.js:311` — `background: '#000'`

In CSS: `index.css:994` (`#E8743B` panel-head icon), `4049` (`#9C6ADE`), `4290` (`#FFAC03`), `4326-4327` (`#FFAC03`, `#d99a2b`), `5050` (`#FFAC03`, `#201400`).

**`--section-color` is hard-coded too** (`index.css:3792-3795`): `#5B8DEF`, `#9C6ADE`, `#FFAC03`, `#2f9e6e`. The mechanism is good — per-section identity read via `color-mix()` — but the values sit outside the token system, and `#FFAC03` is a literal duplicate of `--primary`.

**Recommend:** the two sticky-footer backgrounds in `CorporateProjectSpreadStage.js` are a genuine dark-mode bug and worth fixing first. Promote the four `--section-color` values to real tokens; they're a legitimate palette that just isn't declared as one.

---

## F-05 · Dead and Tailwind-only tokens · Medium

**Truly dead (defined in all 3 blocks, in Tailwind, consumed nowhere):** `--chart-6`, `--chart-7`, `--chart-8`, `--chart-neutral`.

**Tailwind-only (no direct `var()` consumer, but reachable as a utility):** `--popover`, `--popover-foreground`, `--accent-foreground`, `--secondary-foreground`, `--destructive-foreground`.

The distinction matters: a naive "unused token" sweep flags all nine. Only the first four are safe to delete. The five `*-foreground` and popover tokens are wired through `tailwind.config.js:21-55` and a utility like `bg-popover` would break without them.

**Recommend:** delete the four dead chart tokens. Leave the Tailwind-only five, or verify utility usage first.

---

## F-06 · `--brand-*` absent from `:root` · **High**

The 10-token `--brand-*` family is defined **only** inside `body.brand-hub-theme-active` (`index.css:98-107`) and its `[data-direction='B']` variant. `:root` (lines 7-51) has none of them.

Activation is `App.js:57-59`:
```js
document.documentElement.setAttribute('data-direction', direction);
document.body.classList.add('brand-hub-theme-active');
document.body.setAttribute('data-direction', direction);
```

`--brand-line` is the app's default border and the **second-most-consumed token overall** (179 uses). Any surface rendering outside that body class — a portal to a detached root, a print stylesheet, an isolated test render, a Storybook-style harness — loses every border, panel fill, input background, and shadow simultaneously, with no error.

This is also why `gallery.html` in this folder must set `class="brand-hub-theme-active"` on `<body>`: without it, the specimens render borderless.

**Recommend:** duplicate the light `--brand-*` block into `:root` as a safe default. Purely additive, zero visual change in the app, removes a whole class of failure.

---

## F-07 · Duplicate and triplicate CSS definitions · Medium

| Selector | Defined at | Conflict |
|---|---|---|
| `.forecast-health-chip.mini` | `4232` (scoped `.ai-cell`), `4429`, `5361` | `4429` sets `font-size:0.6875rem; padding:1px 6px`; `5361` sets `padding:0 6px; font-size:11px`. **Last wins; 4429 is dead.** |
| `.forecast-health-chip.gold` | `957`, `3334` | two definitions |
| `.forecast-accordion > summary` (+ its `h2`, `.forecast-section-code`) | `1116`, `3924`, `3995` | three override waves |
| `summary::after` | `1141`, `4010` | two |
| `.forecast-accordion summary` sticky `top` | `1125` (`226px`) → `3986` (`112px`) | overridden |
| `.forecast-panel` | `708` (border/radius/fill), `1249` (margin/padding) | split across 540 lines — not a conflict, but hard to find |

The pattern is clear: `index.css` has grown in successive "waves" that override rather than replace earlier rules. 5,581 lines with three generations of the same components layered on top of each other.

**Recommend:** collapse the triplicates. `.forecast-health-chip.mini` at 4429 is provably dead and safe to delete.

---

## F-08 · Declared-but-absent font family · Low

`tailwind.config.js:114-118` declares `avenir-pro`, `avenir-pro-light`, `avenir-pro-demi`. The font is never `@import`ed (`index.css:1` loads only Fraunces, Inter Tight, JetBrains Mono) and referenced by **zero** source files. Any `font-avenir-pro` utility would silently fall through to `Helvetica Neue`.

**Recommend:** delete.

---

## F-09 · The two products share almost no vocabulary · **High**

| | Forecast | RM Pro Forma |
|---|---|---|
| `forecast-*` classNames | 1,846 | **0** |
| Distinct `forecast-*` CSS selectors | 249 | — |
| `brand-*` classNames | 24 | 16 |
| Inline `style={{}}` | 194 | 8 |
| Approach | hand-written CSS classes | `ui/` primitives + Tailwind utilities |

Same visual ideas, invented twice, sharing nothing:

| Idea | Forecast | RM Pro Forma |
|---|---|---|
| Card surface | `.forecast-panel` (radius 14, `--brand-panel-strong`, no shadow) | `.card-wrapper` (radius 18, `--brand-panel`, `--brand-shadow`) |
| Mono eyebrow | `.forecast-section-code` | `.brand-rm-eyebrow` |
| Tab selector | `.forecast-segmented` | Tailwind pill buttons (`RMProForma_Dashboard.js:905-926`) |
| Accordion | `.forecast-accordion` (native `<details>`) | Radix `ui/accordion` |
| Buttons | `.forecast-primary-button` | `ui/Button` → `.brand-button-primary` |
| Input | `.forecast-table input` | `.brand-input` via `ui/Input` |

**The `brand-*` layer (29 selectors, all token-backed, both themes) is already the de-facto shared foundation** — it's what `card-wrapper`, `brand-scroll-shell`, `brand-floating-*`, `brand-tooltip`, and `brand-table-header` live in, and it's the only vocabulary both products touch.

**Recommend:** don't merge the products. Do declare `brand-*` the shared layer explicitly, and when a `forecast-*` and an RMPF component are the same idea (the six rows above), converge them onto an `brand-*` primitive one pair at a time.

---

## F-10 · Three incompatible table implementations · Medium

1. **Forecast:** `.forecast-home-table` / `.forecast-table` — `border-collapse`, 12.5-13px body, JetBrains Mono 10px uppercase `th`, `--brand-line` borders. `index.css:2843-2874, 1929-1979`
2. **RMPF form grid:** `<table className="w-full">` with per-cell `px-2 py-1` — **no shared class at all.** `RMProForma_Dashboard.js:706-755`
3. **RMPF statements:** `<table className="min-w-full">` in an `brand-scroll-shell`, header class on the `<tr>` (`brand-table-header`) not the `<thead>`, sized by a scoped `<style jsx>` block of `custom-text-size-*` rules. `MonthlyAnnualBalanceSheet.js:46, 149-162`; `MonthlyAnnualIncomeStatement.js:79, 217-230`

Three products' worth of table conventions in one app. (3) is the odd one — a `<style jsx>` block inside a component, unlike anything else in the codebase.

---

## F-11 · `forecast-panel-head` has two incompatible child shapes · Medium

~30 sites put `.forecast-section-code` as a direct child; ~20 wrap children in a `<div>` to allow a trailing icon sibling. CSS only special-cases the `> div` form under one ancestor (`index.css:5006`, `.workspace-ai-planning-tools`), so vertical alignment differs by site.

Heading level also splits `h2` (≈38 sites) / `h3` (≈16) with no rule beyond "subpanels use h3".

**Recommend:** standardise on the wrapped form (it degrades correctly with or without an icon) and document the heading rule.

---

## F-12 · Section-code separator glyph is unstable · Low

`●` in `ForecastExecView.js:903`, `ForecastReviewConsole.js`, `ForecastProjectWorkspace.js`; `·` in `ForecastOp03Suite.js`, `stages/RegionalContextStage.js:184`, `ForecastNiiBuildup.js`. Same visual slot, two characters.

---

## F-13 · `submit-chips` vs `submit-actions` · Low

`.forecast-submit-actions` is the semantically-named wrapper for a button row; only `stages/RegionalContextStage.js:160` uses it. Every other file drops buttons straight into `.forecast-submit-chips` (`ForecastOp03Suite.js:844`, `ForecastCycleHome.js:257`). `.forecast-submit-chips` also carries no spacing, so nearly every site adds inline `marginTop`.

---

## F-14 · `aria-pressed` missing on half the segmented controls · Medium

Present: `stages/RegionalResultsStage.js:176, 270`, `stages/RegionalContextStage.js:58`.
Absent: `ForecastExecView.js:880`, `ForecastReviewConsole.js:660`, `ForecastMacroScenarios.js:959` — active state is className-only, so those toggles announce nothing to assistive tech.

`role="group"` + `aria-label` likewise present at only 2 of 6 sites.

**Contrast:** the distribution tracks (PATTERNS P6c/P6d) do this correctly — `role="img"` + full-sentence `aria-label` on the track, `aria-hidden` on decorative markers. The good pattern exists; it just isn't applied consistently.

---

## F-15 · `card-wrapper` silently overrides `ui/Card` · Medium

`ui/Card` (`index.js:40-45`) applies `rounded-xl border-border bg-card text-card-foreground`. RM Pro Forma passes `card-wrapper` alongside (`RMProForma_Dashboard.js:854`), and `.card-wrapper` (`index.css:382-393`) sets `border-radius:18px` + `background: var(--brand-panel)`.

The stylesheet wins. The Tailwind classes on `ui/Card` are dead whenever `card-wrapper` is present — which is every usage. Anyone reading the JSX would conclude the card is `rounded-xl` on `bg-card`; it isn't.

---

## F-16 · `ui/Button` exposes one of three variants · Medium

`index.js:81-89` hard-codes `brand-button-primary`. There is no `variant` prop. But `.brand-button-secondary` (`index.css:456`) and `.brand-button-danger` (`475`) both exist and are styled.

RM Pro Forma works around it by dropping to a raw element: `<button className="brand-button-secondary rounded-full">` (`RMProForma_Dashboard.js:934`).

**Recommend:** add a `variant` prop. The CSS is already written.

---

## F-17 · No `cn()` merge in two of three UI files · Medium

`ui/index.js` and `ui/select.js` concatenate `className` with raw template literals. Only `ui/accordion.js:5` imports `cn` from `../../lib/utils`.

Consequence: `<CardContent className="p-0">` produces `class="p-5 pt-0 p-0"`. Tailwind has no cascade priority between conflicting utilities of the same property — the result depends on stylesheet order, not the caller's intent. A latent, silent bug in every consumer that tries to override padding.

**Recommend:** route all three through `cn`. `lib/utils.js` already exports it.

---

## F-18 · Two icon libraries across three UI files · Low

`accordion.js:2` imports `ChevronDownIcon` from `@radix-ui/react-icons`; `select.js:3` imports `Check, ChevronDown` from `lucide-react`. The rest of the app uses lucide.

**Recommend:** standardise on lucide.

---

## F-19 · `.forecast-chip-button` is used but undefined · **High**

Used in three files:
- `stages/RegionalResultsStage.js:164` — "Collapse all schedules"
- `ForecastCycleHome.js:142` — "Dismiss"
- `ForecastNotificationsTray.js:118` — "Mark all read"

Defined in **zero** stylesheets (verified across all `.css` in `frontend/src`).

All three render as unstyled browser-default buttons. This is a visible defect, not just tidiness.

**Recommend:** define it (probably as an alias of `.forecast-secondary-button.compact`), or change those three call sites.

---

## F-20 · The `vm` prop bag · Medium

`RegionalForecastingModule.js:5066, 5069, 5078` build a ~30-key object literal per stage carrying Recharts primitives, custom components, constants, formatters, and chart-theme strings. Stages destructure it wholesale (`RegionalBaselineStage.js:5`, `RegionalContextStage.js:12`, `RegionalResultsStage.js:7`).

**This is a deliberate pattern with a real benefit** — stages import nothing, and tests stub `vm` in one line (`RegionalContextStageMacroStale.test.js:36`). Recorded here not as a defect but because it's an untyped ~30-key contract with no compile-time checking, and because **it's already been partially escaped**: `ForecastNiiBuildup.js` is rendered inside a `vm` stage (`RegionalResultsStage.js:495-502`) but imports Recharts directly and takes explicit props.

**Recommend:** keep it or replace it wholesale — but don't leave it half-escaped. The mixed state is the expensive part.

---

## F-21 · Four implementations of the labelled-distribution track · Medium

`consol-bridge-*` (`ForecastReviewConsole.js:1078`), `pulse-*` (`:703`), `forecast-bullet-*` (`ForecastGuardrailBoard.js:52`), `forecast-peer-distribution-*` (`ForecastPeerProForma.js:108`).

All four: label block (`span` + `strong` + `small`) → track → markers positioned by `left:%`. Zero shared classes. **The bullet and peer variants are structurally near-identical**, down to the `role="img"` + `aria-label` treatment.

CSS cost: `index.css:4263-4299` (bridge) + `4305-4328` (pulse) + separate bullet and peer blocks.

**Recommend:** the highest-value consolidation in the codebase. Start with the bullet/peer pair — they're already the same component wearing two names.

---

## F-22 · Two stat-strip implementations · Low

`.forecast-status-strip` (`<section>`, `span` + `strong`, `ForecastCycleHome.js:200`) vs `.forecast-kpi-grid` (`<div>`, `span` + `strong` + `small`, `stages/RegionalContextStage.js:326`). They **share CSS at `index.css:827-840`** and diverge at `807-825` vs `1456-1480` — evidence they were once meant to be one. No file uses both.

---

## F-23 · Notification tray reimplements the panel head · Low

`ForecastNotificationsTray.js:117` uses `.forecast-notif-head` — same `forecast-section-code` + trailing action composition as `.forecast-panel-head`, bespoke class. One of only two files in the sample that never use `.forecast-panel-head`.

---

## F-24 · `RM_Pro_Forma_Color_Mapping.md` is comprehensively stale · Medium

The root-level doc is built on the 9-colour brand palette that no longer renders (F-03). Reconciled claim by claim:

**Still accurate:**
- Flash duration 300ms — `RMProForma_Dashboard.js:210` ✓
- Direction-dependent increment/decrement colouring — `:209` ✓
- Card headers `px-5 py-3` — `ui/index.js:53` ✓
- The four accordion section names, in order — `:1123, :1177, :1282, :1290` ✓
- Radix UI select components — `select.js:2` ✓
- `overscroll-behavior: none` on html/body — `index.css:170-175` ✓
- Line thickness 3px — ✓ (exception: `RMProForma_YieldCurve.js:82` uses `isCurrent ? 3 : 2`)
- Net income dashed line — `RMProForma_FinancialStatements.js:338` ✓
- Chart grid lines have a dedicated colour — concept ✓, colour ✗ (not Pigeon `#acafaf`; it's `--chart-grid`)

**Stale or wrong:**
| Doc claim | Reality |
|---|---|
| Entire brand palette table | None of the 9 hexes renders anywhere |
| "Main page titles `text-2xl font-bold text-brand-hobgoblin`" | `<h1 className="brand-rm-title">` (`:845`) — clamp 30-44px, weight 600 |
| "Accordion triggers Mulberry, chevron Nero, hover underline" | `accordion.js:23` — `text-foreground`, `hover:text-primary`, `font-semibold`; chevron `text-muted-foreground`. Hover is a colour change, not underline |
| "Calculate button `bg-brand-mulberry hover:bg-brand-mulberry/90`" | `<Button className="mt-4">` (`:1303`) → `.brand-button-primary` (`index.css:437-454`): amber, `border-radius:999px` pill, hover `filter: brightness(0.96)` |
| "Error messages `bg-red-100 border-red-400`" | Those classes appear nowhere. Actual: `border-destructive/45 bg-destructive/15 text-destructive` (`:1046`) |
| "Input arrows: base Smokescreen, hover Hobgoblin/Lust, bg Pigeon" | `:228, :236` — base `text-muted-foreground`, hover `text-accent`/`text-destructive`, hover bg `bg-primary/15` |
| "Dropdown hover: Caribbean Blue with white text" | `select.js:59` — `data-[highlighted]:bg-primary/20 data-[highlighted]:text-foreground` |
| "Statement table headers: Smokescreen bg, white text" | `brand-table-header` (`index.css:489-493`) — light panel bg, dark `--foreground` text. **The inverse.** |
| "Yearly input table cell backgrounds: Input Fill" | Cells unstyled (`:719`); the fill is on the nested `.brand-input` |
| "Payback bars: Hobgoblin/Lust gradients" | `RMProForma_Payback.js:64-72` — magnitude-scaled **alpha on solid fills**, `hsl(var(--chart-up) / …)`. No `<linearGradient>` exists in the file |
| All chart line-colour claims | Charts use `hsl(var(--chart-N))` exclusively |
| "Info tooltips `bg-brand-smokescreen text-white`" | `.brand-tooltip` (`index.css:495-501`) — `--brand-panel-strong` bg, `--foreground` text |
| "Navigation active: Mulberry background" | `.brand-nav-link` (`index.css:252+`); RMPF tabs use `bg-primary`/`bg-secondary` |

**Recommend:** replace the doc with a pointer to `design-system/tokens.json` + `CHARTS.md`. Its accurate content (flash timing, accordion structure, stroke widths) is already folded into this folder. Keeping a second colour doc guarantees a second drift.

---

## F-25 · `console.log` in chart render paths · Low

`AssetBalanceChart.js:10, 16, 37, 54, 60, 82, 87, 121` (8 calls, one logging the full `assetBalances` payload via `JSON.stringify` on every render). Also `RMProForma_FinancialStatements.js:8`, `RMProForma_Payback.js:8`.

---

## F-26 · `.forecast-statement-matrix` styled, barely used · Low

Fully styled at `index.css:4113-4118` with its own `.forecast`/`.priorForecast`/`.mixed`/`.actual` cell variants and a `.statement-line-col` sticky column — **1 usage** in `frontend/src`. Either near-dead or awaiting a page that was never finished.

---

## F-27 · `--radius` defined but ignored · Medium

`--radius: 0.75rem` is defined only in `:root` (`index.css:36`); neither theme block redefines it. It feeds `tailwind.config.js:109-113` (`lg`/`md`/`sm`).

**Hand-written CSS ignores it entirely:** panels `14px`, cards `18px`, table shells `12px`, all pills `999px`, small controls `10px`. None reference `var(--radius)`.

The Tailwind comment at `:111` says "Default radius (0.375rem)" — wrong twice over: the token is `0.75rem`, and `DEFAULT` is separately hard-coded to `0.375rem` at `:112`.

**Recommend:** either add real radius tokens (`--radius-panel: 14px`, `--radius-card: 18px`, `--radius-pill: 999px`) matching what the CSS actually does, or drop `--radius`. Right now it documents a system nobody follows.

---

---

## F-28 · Chip tone names contradict what they render · **High**

*Found by rendering `gallery.html` against the real stylesheet and reading computed styles — not visible from reading the CSS alone.*

`.forecast-health-chip` has seven tone variants. Computed `background-color` in the browser, Theme A:

| Variant | Renders | Token behind it | Verdict |
|---|---|---|---|
| `.green` | **amber** `srgb 1 0.673 0.02 / 0.13` | `--primary` | **name is wrong** |
| `.moss` | **amber** `srgb 1 0.673 0.02 / 0.13` | `--primary` | **byte-identical to `.green`** |
| `.gold` | amber `srgb 1 0.673 0.02 / 0.16` | `--primary` | correct, but ~indistinguishable from the two above |
| `.yellow` | **olive green** `srgb 0.461 0.59 0.41 / 0.16` | `--accent` | **name is wrong** |
| `.amber` | **olive green** `srgb 0.461 0.59 0.41 / 0.16` | `--accent` | **name is wrong; byte-identical to `.yellow`** |
| `.red` | terracotta `srgb 0.8425 0.394 0.2575 / 0.13` | `--destructive` | correct |
| `.neutral` | 4% foreground wash | `--foreground` | correct |

So of seven names, **four describe a colour the chip does not render**, and there are only **four distinct renderings** across the seven variants (amber ×3 at two opacities, olive ×2, terracotta, neutral).

CSS evidence — `index.css:948-980`:
```css
.forecast-health-chip.green  { background: color-mix(in srgb, hsl(var(--primary)) 13%, transparent); }
.forecast-health-chip.moss   { background: color-mix(in srgb, hsl(var(--primary)) 13%, transparent); }
.forecast-health-chip.yellow { background: color-mix(in srgb, hsl(var(--accent))  16%, transparent); }
```

**Why this is High, not cosmetic:** a developer writing `className="forecast-health-chip green"` to signal a healthy status gets an **amber** chip — which in this product's own visual language reads as *caution*, because amber is `--primary` and every "watch"/"in review" state uses it. Writing `yellow` for a warning produces a **green** chip that reads as healthy. The two most semantically loaded tones are effectively swapped at the call site, and there are 186 call sites.

Worth checking whether any of the 186 usages are currently mis-signalling status to users. That audit is out of scope here; the naming defect is confirmed.

**Recommend:** rename to token-derived, meaning-carrying names — `.tone-primary` / `.tone-accent` / `.tone-destructive` / `.tone-neutral`, or status-semantic `.ok` / `.watch` / `.breach` — and collapse the three duplicate pairs. Do it as a mechanical rename across all call sites in one commit; leaving both old and new names is how this happened.

## What was checked and found clean

- **Token values are internally consistent** across `:root`, Theme A, and Theme B — no orphaned or mismatched declarations in the three blocks.
- **Theme B (dark) covers every token Theme A defines.** No token goes missing in dark mode.
- **Chart tokens are correctly wired through Tailwind** (`tailwind.config.js:58-76`) — the config faithfully mirrors the CSS.
- **`--chart-up`/`--chart-down`/`--destructive` value relationships hold in both themes** — the aliasing is deliberate, not accidental.
- **No `!important` abuse** — across 5,581 lines of `index.css` the override waves are all managed by specificity and source order.
- **Every specimen in `gallery.html` renders correctly against the real stylesheet in both themes**, with no console errors — the token layer and the `forecast-*`/`brand-*` component layers are internally sound. F-28 was surfaced by that render.
- **The `brand-*` layer is uniformly token-backed** — all 29 selectors reference tokens; no hard-coded colour in that family.
