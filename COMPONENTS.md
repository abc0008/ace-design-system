# Component Catalog

The recurring, genuinely reusable components in **Forecast** and **RM Pro Forma**, ranked by real usage. Harvested from classNames in `frontend/src/pages/forecasting/` (82 files, 22,754 lines), `RMProForma_Dashboard.js`, `RegionalForecastingModule.js`, and `components/charts/`.

**Inclusion bar:** appears in ≥2 files, or is a load-bearing primitive. One-offs are excluded (a list of near-misses is at the bottom). Markup snippets are distilled from real JSX — trimmed, structurally faithful, real class names.

**Read first:** every component below is plain CSS in `frontend/src/index.css`. There is no React component library for the Forecast vocabulary — `forecast-*` classes are applied by hand. Only the small `components/ui/` layer (§C) is componentised.

---

## The two vocabularies

The single most important structural fact about this codebase:

| | Forecast | RM Pro Forma |
|---|---|---|
| `forecast-*` classNames | **1,846** | **0** |
| Distinct `forecast-*` CSS selectors | 249 | — |
| `ace-*` classNames | 24 | 16 |
| Styling approach | hand-written CSS classes | `components/ui/` + Tailwind utilities |
| Inline `style={{}}` | 194 | 8 |

They share exactly three things: the token layer, the `ace-*` primitives (§B), and `card-wrapper`. Everything else is parallel invention. See `FINDINGS.md` F-09.

---

# A. Forecast vocabulary

## A1. `forecast-panel` — the structural primitive
**Purpose:** every content block in the product. Card surface with border, radius, and panel fill.
**Usage:** 100 occurrences, 15 of 17 sampled files. `index.css:708-713, 1249-1252`

```jsx
<section className="forecast-panel">
  <div className="forecast-panel-head">
    <span className="forecast-section-code">EX_00 ● NII composition — FTP basis</span>
    <h2>Lending spread vs deposit franchise ($MM, by year)</h2>
  </div>
  {/* table | chart | chip row */}
</section>
```

**Tokens:** `--ace-line` (border), `--ace-panel-strong` (fill, at 94% via `color-mix`), `--foreground`
**Variants:** `.forecast-panel-lite` (1178), `.forecast-panel-tall` (1258), `.forecast-subpanel` (3303 — nested, `h3`-scale, 2% foreground wash), `.forecast-grid .forecast-panel` (margin reset, 1254)
**Shares its rule with:** `.forecast-status-strip`, `.forecast-operating-bar` — same border/radius/fill triple
**Where used:** `ForecastExecView.js:902, 968, 992`, `ForecastReviewConsole.js:1075`, `stages/RegionalContextStage.js:193`, `ForecastNiiBuildup.js:119`, `ForecastProjectWorkspace.js:804`

## A2. `forecast-panel-head` — panel header block
**Purpose:** the header slot inside a panel: section code + heading, optional trailing icon.
**Usage:** 91 occurrences, 14 files. `index.css:1262-1279`

```jsx
{/* Form 1 — flat (≈30 sites) */}
<div className="forecast-panel-head">
  <span className="forecast-section-code">RV_02 ● Divisional roll-up</span>
  <h2>Consolidated net income bridge</h2>
</div>

{/* Form 2 — icon-bearing, children wrapped (≈20 sites) */}
<div className="forecast-panel-head">
  <div>
    <span className="forecast-section-code">OP_02X · Central macro controls</span>
    <h3>Corporate Planning only — changes propagate to every inputter</h3>
  </div>
  <ShieldCheck size={16} />
</div>
```

**Tokens:** `--foreground` (h2), `--primary` (svg colour, 1276-1279)
**Layout:** `display:flex; align-items:flex-start; justify-content:space-between; gap:14px`
**⚠ Two incompatible child shapes** (Form 1 vs Form 2) and an unruled `h2`/`h3` split — see FINDINGS F-11.

## A3. `forecast-health-chip` — the workhorse atom
**Purpose:** status badge, metric tag, filter toggle, and nav link — one class, four jobs.
**Usage:** **186 occurrences — the single most-used class in the product.** 13 files. `index.css:923-986, 3326-3337`

```jsx
{/* static badge */}
<span className="forecast-health-chip green">NII ${nii.toFixed(1)}MM/mo</span>

{/* toggle — the dominant interactive idiom */}
<button type="button"
  className={`forecast-health-chip clickable ${bridgeYear === year ? 'gold' : 'neutral'}`}
  onClick={() => setBridgeYear(year)}>FY{year}</button>

{/* nav */}
<Link className="forecast-health-chip neutral" to="/forecasting/macro?tab=compare">Macro isolation A/B</Link>
```

**Base:** `inline-flex`, `min-height:26px`, `border-radius:999px`, `padding:6px 9px`, JetBrains Mono 10px, uppercase, `letter-spacing:0.06em`, `border:1px solid var(--ace-line)`
**Tokens:** `--ace-line`, `--foreground`, `--primary` (green/moss/gold), `--accent` (yellow), `--destructive` (red)
**Variants:** `.green` `.yellow` `.red` `.neutral` `.moss` `.amber` `.gold` `.clickable` `.mini`
**Tone convention:** every variant is `color-mix(in srgb, hsl(var(--token)) 13-16%, transparent)` fill + the same token at 38-44% for the border. Copy that formula for new tones.
**⚠⚠ The tone names lie.** `.green` renders **amber**, `.yellow` and `.amber` render **olive green**, and `.green`/`.moss`, `.yellow`/`.amber` are byte-identical pairs. Only `.red` and `.neutral` mean what they say. Verified in-browser — see FINDINGS F-28 before using any tone but `.red`/`.neutral`.
**⚠** `.mini` is defined 3× (4232 scoped, 4429, 5361 — the last two conflict); `.gold` 2× (957, 3334). FINDINGS F-07.

## A4. `forecast-submit-note` — the explanatory footnote
**Purpose:** the caption/error line under nearly every panel and action row.
**Usage:** 116 occurrences, 13 files — **second most-used class.** `index.css:2985-2998`

```jsx
<p className="forecast-submit-note">Reviewer must return the package before re-anchoring.</p>
<p className="forecast-submit-note error-list">{error}</p>
```

**Tokens:** `--muted-foreground` (base), `--destructive` (`.error-list`)
**Base:** `font-size:12px; line-height:1.6; margin-top:12px`
**⚠** Error state is applied inconsistently: `.error-list` exists, but ~7 sites instead write `style={{ color: 'var(--ace-danger, #e5241d)' }}` inline against a **phantom token**. FINDINGS F-01.

## A5. `forecast-section-code` / `forecast-kicker` — the machine-readable eyebrow
**Purpose:** short uppercase mono code identifying a section. The product's signature detail.
**Usage:** 112 occurrences, 15 files. `index.css:724-735` (both classes share one rule)

```jsx
<span className="forecast-section-code">OP_03 · Operating levers</span>
```

**Tokens:** `--muted-foreground`
**Base:** JetBrains Mono 10px, uppercase, `letter-spacing:0.12em`, `gap:8px`
**Format convention:** `PREFIX_NN <separator> Label`. Prefixes in use: `EX_` `RV_` `OP_` `PRJ_` `CYCLE_`.
**⚠ Separator glyph is unstable** — `●` in ExecView/ReviewConsole/ProjectWorkspace, `·` in Op03Suite/stages/NiiBuildup. FINDINGS F-12.

## A6. `forecast-primary-button` / `forecast-secondary-button`
**Purpose:** the button pair. Pill-shaped, icon-first.
**Usage:** 32 / 63 occurrences. `index.css:3035-3066`, compact at `3599` and `4037`

```jsx
<button type="button" className="forecast-primary-button" disabled={busy} onClick={submit}>
  <RefreshCw size={14} /> Re-anchor to v{version}
</button>
<button type="button" className="forecast-secondary-button compact" onClick={cancel}>
  <X size={13} /> Reject
</button>
```

**Shared base:** `inline-flex; gap:7px; border-radius:999px; padding:9px 18px; font-size:12.5px`
**Primary tokens:** `--primary` (bg + border), `--primary-foreground` (text)
**Secondary tokens:** `--ace-line` (border), `transparent` bg, `--foreground` (text)
**States:** `:disabled { opacity:0.5; cursor:default }`; `.compact` → `padding:5px 12px; font-size:11.5px`
**Icon convention:** lucide, `size={13}` on compact, `size={14}` on default, always leading.
**⚠ There is no danger button** in the Forecast vocabulary, though `.ace-button-danger` exists (index.css:475).

## A7. `forecast-submit-chips` — the universal control strip
**Purpose:** horizontal wrap row holding chips, buttons, selects. The de-facto toolbar.
**Usage:** 44 occurrences, 10 files. `index.css:2973-2977`

```jsx
<div className="forecast-submit-chips" style={{ marginTop: 8 }}>
  <button type="button" className="forecast-primary-button compact" onClick={accept}>
    <Check size={13} /> Accept
  </button>
  <button type="button" className="forecast-secondary-button compact" onClick={reject}>Reject</button>
</div>
```

**Base:** `display:flex; flex-wrap:wrap; gap:8px` — that is the entire rule.
**⚠** Carries no spacing of its own, so nearly every site adds inline `marginTop`/`marginLeft:'auto'`. Also used interchangeably with the semantically-named `.forecast-submit-actions`. FINDINGS F-13.

## A8. `forecast-home-table` — the read-oriented list table
**Purpose:** roster, log, and summary tables. Narrow, no horizontal scroll, pos/neg numerics.
**Usage:** 30 occurrences + 31 wraps, 9 files. `index.css:2836-2878`

```jsx
<div className="forecast-home-table-wrap">
  <table className="forecast-home-table">
    <thead><tr><th>Event</th><th>Actor</th><th>When</th><th /></tr></thead>
    <tbody>
      {rows.length === 0 ? (
        <tr><td colSpan={4}><span className="dim-note">No events recorded.</span></td></tr>
      ) : rows.map((row) => (
        <tr key={row.id} className={row.isMine ? 'mine' : ''}>
          <td><span className="forecast-health-chip neutral mini">{row.event}</span></td>
          <td>{row.actor || '—'}</td>
          <td className="forecast-mono">{fmtWhen(row.createdAt)}</td>
          <td><button type="button" className="forecast-home-action ghost">Diff</button></td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

**Tokens:** `--ace-line` (th border; td border at 55%), `--muted-foreground` (th), `--primary` (`tr.mine` 5% wash), `--chart-up` (`.pos`), `--destructive` (`.neg`)
**th treatment:** JetBrains Mono 10px uppercase `letter-spacing:0.08em` — the table-header convention across the app.
**Variants:** `tr.mine`, `td.pos`, `td.neg`
**Empty state convention:** `colSpan` + `<span className="dim-note">` — consistent across all 9 files.

## A9. `forecast-table` + `forecast-table-shell` — the dense numeric grid
**Purpose:** wide, horizontally scrollable, sticky-header, often editable financial grid. Distinct from A8 by intent (input/wide vs read/narrow) and by an entirely separate class family.
**Usage:** 17 occurrences each. `index.css:1912-1979`

```jsx
<div className="forecast-table-shell ace-scroll-shell forecast-compact-table">
  <table className="forecast-table">
    <thead><tr><th>Metric</th>{years.map((y) => <th key={y}>FY{y}</th>)}</tr></thead>
    <tbody>
      {metrics.map((m) => (
        <tr key={m.key}>
          <td><strong>{m.label}</strong></td>
          {years.map((y) => (
            <td key={y}>
              <input value={m.values[y]} onChange={(e) => update(m.key, y, e.target.value)} />
            </td>
          ))}
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

**Tokens:** `--ace-line` (cell borders), `--ace-line-strong` (input borders), `--ace-input-bg`, `--ace-panel-strong` (sticky th), `--primary` (input focus ring), `--foreground`
**Base:** `min-width:960px` forces the scroll; `.forecast-compact-table` relaxes to 820px
**Sticky:** `th` sticky at 1949-1959; `td:first-child`/`:nth-child(2)` are sticky label columns (1965)
**Input state:** `:focus { border-color: hsl(var(--primary)); box-shadow: 0 0 0 2px color-mix(in srgb, hsl(var(--primary)) 28%, transparent) }` — the app's canonical focus treatment.
**Always pair with `ace-scroll-shell`** (B1) for the themed scrollbar.

## A10. `forecast-segmented` — pill tab / lens selector
**Purpose:** mutually-exclusive view switch. The app's only tab control.
**Usage:** 12 occurrences, 6 files. `index.css:1281-1320`

```jsx
<div className="forecast-segmented" role="group" aria-label="Results lens">
  {['Variance', 'Plan', 'Modified'].map((view) => (
    <button type="button" key={view}
      aria-pressed={resultsView === view}
      className={resultsView === view ? 'active' : ''}
      onClick={() => setResultsView(view)}>{view}</button>
  ))}
</div>
```

**Tokens:** `--ace-line` (track border), `--ace-panel` (track fill at 80%), `--primary` + `--primary-foreground` (active button), `--foreground` (rest)
**Base:** track is a `999px` pill with `padding:4px; gap:4px`; buttons are `min-height:30px`, transparent border, `transition:0.16s ease`
**States:** `button.active` → amber fill
**⚠ `aria-pressed` is applied at only 3 of 6 sites** — the other three announce nothing to assistive tech. FINDINGS F-14.

## A11. `forecast-status-chip` — compact state tag
**Purpose:** smaller sibling of A3, used for workflow status rather than metrics.
**Usage:** 11 occurrences. `index.css:2774-2809`

```jsx
<span className={`forecast-status-chip ${TONE[pkg.status]}`}>{pkg.status}</span>
```

**Base:** `min-height:22px; padding:3px 9px` (vs A3's 26px/6px 9px), otherwise identical mono/uppercase treatment.
**Variants:** `.moss` `.gold` `.amber` `.red` `.neutral`
**⚠** A3 and A11 are near-duplicates with overlapping but *non-identical* variant sets (A3 has green/yellow, A11 has gold/amber). No rule distinguishes them.

## A12. `forecast-accordion` — page-section disclosure
**Purpose:** collapsible section built on native `<details>`, with the panel-head inside `<summary>`.
**Usage:** 11 occurrences, stages only. `index.css:1107-1166`

```jsx
<details className="forecast-accordion" open>
  <summary>
    <div>
      <span className="forecast-section-code">OP_02 · Macro environment</span>
      <h2>Base economic scenario and annual divergence</h2>
    </div>
    <SlidersHorizontal size={18} />
  </summary>
  <div className="forecast-accordion-body">{/* panels */}</div>
</details>
```

**Tokens:** `--ace-line`, `--muted-foreground`, `--primary` (summary svg)
**Signature behaviour:** `summary::after` renders a pill reading `Collapse`, swapped to `Expand` via `:not([open])` (1141-1156). `::-webkit-details-marker` is suppressed.
**⚠** `summary` padding and its children are redefined in three successive override waves (1116 → 3924 → 3995). FINDINGS F-07.

## A13. `forecast-mono` — tabular figure treatment
**Purpose:** the numeric/timestamp cell treatment.
**Usage:** 69 occurrences. `index.css:3079-3082`

```jsx
<td className="forecast-mono">{fmtWhen(row.createdAt)}</td>
<span className="forecast-mono neg">{delta.toFixed(1)}</span>
```

**Base:** `font-family:'JetBrains Mono', monospace; font-size:11px` — that is the whole rule.
**Composes with:** `.pos` / `.neg` for sign colour.

## A14. `forecast-module` — product shell
**Purpose:** the root wrapper for every Forecast page. Establishes canvas and box-sizing.
**Usage:** 12 occurrences. `index.css:695-705`

```jsx
<div className="forecast-module">{/* page */}</div>
```

**Tokens:** `--background`, `--foreground`
**Base:** `min-height: calc(100vh - 72px)` (72px = header), negative margins `-22px -6px -24px` to bleed past the app's default padding, and `.forecast-module * { box-sizing: border-box }`.

## A15. `forecast-grid` — panel layout grid
**Purpose:** multi-column arrangement of panels.
**Usage:** 5 + `.forecast-grid-2` / `.forecast-grid-3` variants. `index.css:1231-1256`

```jsx
<div className="forecast-grid forecast-grid-2">
  <section className="forecast-panel">…</section>
  <section className="forecast-panel">…</section>
</div>
```

**Base:** `display:grid; gap:12px; margin-top:12px`; nested panels get `margin-top:0`.

---

# B. Shared `ace-*` primitives

29 selectors in `index.css`, all semantic-token-backed and theme-aware. **This is the de-facto shared layer between the two products** and the right foundation for any future consolidation.

## B1. `ace-scroll-shell` — themed scrollbar
**Usage:** 17 in Forecast, 2 in charts. `index.css:607-624`
```jsx
<div className="forecast-table-shell ace-scroll-shell">…</div>
```
`scrollbar-width:thin`; thumb is `color-mix(in srgb, hsl(var(--foreground)) 26%, transparent)`, `999px` radius, 10px track. Pair with any scrolling container.

## B2. `card-wrapper` — the RM Pro Forma card surface
**Usage:** 11 in charts, 1 in RMPF dashboard. `index.css:382-393`
```jsx
<Card className="card-wrapper overflow-hidden">
  <CardHeader className="border-b border-border/60 bg-card/45">…</CardHeader>
  <CardContent>…</CardContent>
</Card>
```
**Tokens:** `--ace-panel` (fill), `--ace-line` (border), `--ace-shadow`
**Base:** `border-radius:18px` — note this **overrides** the `rounded-xl` and `bg-card` that `ui/Card` applies, because the stylesheet wins. FINDINGS F-15.
**vs `forecast-panel`:** same idea, different radius (18 vs 14), different fill token (`--ace-panel` vs `--ace-panel-strong`), and this one has a shadow. They are not interchangeable and should be.

## B3. `ace-input` — the text input
**Usage:** 6 in RMPF, also the base for `SelectTrigger`. `index.css` (~406)
```jsx
<input className="ace-input" value={value} onChange={onChange} />
```
**Tokens:** `--ace-input-bg`, `--ace-line-strong`, `--foreground`
**Note:** `select.js:12` reuses this class for `SelectTrigger`, so triggers and text inputs are deliberately identical.

## B4. `ace-button-primary` / `-secondary` / `-danger`
`index.css:437-454 / 456 / 475`. Pill (`999px`), `--primary` fill, hover `filter: brightness(0.96)`.
**⚠** Only `-primary` is reachable through `ui/Button` — the other two require dropping to a raw `<button>`. FINDINGS F-16.

## B5. `ace-floating-surface` / `ace-floating-item` — dropdown menu
**Usage:** Radix `SelectContent`/`SelectItem`, plus RMPF's portal menu. `index.css:~425-434`
```jsx
<div className="ace-floating-surface fixed z-[9999] w-56 p-1">
  <button className="ace-floating-item block w-full px-3 py-2 text-left text-sm">Duplicate</button>
</div>
```
Item hover: `color-mix(in srgb, hsl(var(--primary)) 16%, transparent)`. **The one primitive genuinely shared by both products.**

## B6. `ace-tooltip` / `ace-table-header` / `ace-chart-legend-text`
`index.css:489-501`. Chart and table chrome, all on `--ace-panel-strong` + `--foreground`. Used by `ui/Tooltip`, the statement tables, and every chart legend.

## B7. `ace-rm-overview` / `-eyebrow` / `-title` / `-description` / `ace-serif`
`index.css:337-380`. The RM Pro Forma hero band — the only Fraunces usage in the app.
```jsx
<section className="ace-rm-overview">
  <span className="ace-rm-eyebrow">OP_PRLX · rm-pro-forma</span>
  <h1 className="ace-rm-title">Model every RM hire with
    <span className="ace-serif text-primary"> hub-grade consistency.</span></h1>
  <p className="ace-rm-description">…</p>
</section>
```
Eyebrow is the same mono/uppercase idiom as `forecast-section-code` (A5) with a `::before` dot in `--primary` — **the same component invented twice.**

---

# C. `components/ui/` — the React primitive layer

Radix-based, Tailwind-styled, semantic-token-only. 3 files, 10 exports. Used almost exclusively by RM Pro Forma.

| Export | File:line | Styling |
|---|---|---|
| `Accordion` / `Item` / `Trigger` / `Content` | `accordion.js` | Radix accordion. Trigger `text-base font-semibold text-foreground hover:text-primary`, chevron `text-muted-foreground` rotates on `[data-state=open]`. Animated via `tailwind.config.js:149-162` keyframes off `--radix-accordion-content-height`. |
| `Select` (+7 parts) | `select.js` | Radix select. `SelectTrigger` = `ace-input`; `SelectContent` = `ace-floating-surface`, portalled; `SelectItem` = `ace-floating-item` + `data-[highlighted]:bg-primary/20`. |
| `Slider`, `Label` | `index.js:5-26` | Radix. Track `bg-secondary`, Range/Thumb `bg-primary`, focus `ring-ring`. |
| `Input` | `index.js:28-38` | Plain `<input className="ace-input …">`. |
| `Card` / `Header` / `Title` / `Content` | `index.js:40-79` | `rounded-xl border-border bg-card`; header `px-5 py-3`; title `text-lg font-semibold text-primary`; content `p-5 pt-0`. |
| `Button` | `index.js:81-89` | `ace-button-primary` + disabled states. **Single variant.** |
| `Tooltip` | `index.js:91-100` | `ace-tooltip p-2`. Not Radix — a plain div, no positioning or portal. |

**⚠ Two real defects here:**
1. `index.js` and `select.js` concatenate `className` with raw template literals; only `accordion.js` imports `cn`. So `<CardContent className="p-0">` yields `"p-5 pt-0 p-0"` with no merge — a latent collision bug. FINDINGS F-17.
2. `accordion.js` uses `@radix-ui/react-icons`, `select.js` uses `lucide-react` — two icon libraries in three files. FINDINGS F-18.

---

# D. RM Pro Forma components

## D1. `CustomInput` — numeric stepper with directional flash
The signature RMPF control; every numeric field uses it. `RMProForma_Dashboard.js:163-243`

```jsx
<input className={`ace-input ${flashDirection === 'up' ? 'text-accent'
                  : flashDirection === 'down' ? 'text-destructive' : 'text-foreground'} transition-colors duration-200`} />
<button className="rounded-sm p-1 text-muted-foreground hover:bg-primary/15 hover:text-accent"><ChevronUp size={16}/></button>
<button className="rounded-sm p-1 text-muted-foreground hover:bg-primary/15 hover:text-destructive"><ChevronDown size={16}/></button>
```

**Tokens:** `--ace-input-bg`, `--ace-line-strong`, `--accent` (up), `--destructive` (down), `--primary` (hover wash)
**Behaviour:** value flashes for **300ms** on change (`:210`), colour keyed to direction.
**⚠** This is a **third** up/down colour semantic — `--accent`/`--destructive`, where charts use `--chart-up`/`--chart-down`. FINDINGS F-02.

## D2. Entry tab pills
`RMProForma_Dashboard.js:905-926`. Type-discriminated tab selector.

```jsx
<button className={`px-4 py-2 text-sm font-medium rounded-full border transition-colors ${
  activeIndex === index
    ? (entry.type === 'Support' ? 'border-secondary bg-secondary text-secondary-foreground'
                                : 'border-primary bg-primary text-primary-foreground')
    : 'border-border bg-card/80 text-foreground hover:bg-primary/12'
} ${entry.result ? (entry.type === 'Support' ? 'ring-1 ring-secondary/45' : 'ring-1 ring-accent/45') : ''}`}>
```

**Semantic load:** `--secondary` = Support role, `--primary` = RM role, `ring-accent` = has computed result. Three tokens carrying product meaning — worth preserving explicitly if this is ever refactored.
**vs `forecast-segmented` (A10):** same control, completely different implementation.

## D3. Alert banners — destructive and emphasis
```jsx
<div className="rounded-xl border border-destructive/45 bg-destructive/15 p-4 text-destructive">{error}</div>
<div className="rounded-xl border border-primary/35 bg-primary/12 p-4">{note}</div>
```
**⚠** The destructive variant is copy-pasted **4 times** (`RMProForma_Dashboard.js:1046`, `RMProForma_Payback.js:22`, `RMProForma_FinancialStatements.js:48, 303`) — an unextracted de-facto component.

## D4. Yearly input grid
`RMProForma_Dashboard.js:703-757`. **Bare markup, no table class at all:** `<table className="w-full">`, cells `px-2 py-1`, each hosting a `CustomInput`. The third table implementation in the codebase. FINDINGS F-10.

## D5. Statement tables
`MonthlyAnnualBalanceSheet.js`, `MonthlyAnnualIncomeStatement.js` — despite living in `components/charts/`, these render **zero** Recharts primitives. They are expand/collapse `<table>` components: `<table className="min-w-full">` in an `ace-scroll-shell`, header class on the `<tr>` (`ace-table-header`), sized by a scoped `<style jsx>` block of `custom-text-size-*` rules. See `CHARTS.md §7`.

---

# E. Considered and excluded

| Candidate | Why excluded |
|---|---|
| `forecast-guardrail` / bullet chart | Single instance (`ForecastGuardrailBoard.js`, rendered once at `ForecastExecView.js:1215`). Documented in `PATTERNS.md P7` as a duplication target, not a library component. |
| `forecast-statement-matrix` | Fully styled (`index.css:4113-4118`) but **1 usage**. Near-dead. |
| `forecast-chip-button` | **Used in 3 files, defined in zero stylesheets.** Renders unstyled. FINDINGS F-19. |
| `workspace-stage-tabs` | Its CSS active state (`[aria-selected='true']`, 5050) never matches its markup (`aria-pressed` + `.active`). Styling is dead. |
| `macro-diff-table`, `op3-year-table` | Page-local variants of A9 with one rule each. |
| `forecast-waterfall-*` | Markup exists but the real waterfalls are Recharts configs; no shared class contract. |
