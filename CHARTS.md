# Charting Conventions

## Library

**Recharts `^2.5.0`** (`frontend/package.json`). 16 `from 'recharts'` imports across the app. No other charting library — `d3-array` appears once (`AssetBalanceChart.js:4`, for `extent`) and `date-fns` is the universal date utility.

**There is no shared chart abstraction.** No `useChartTheme` hook, no `<ChartCard>`, no base component, no shared loading/error/empty component. Conventions below are honoured by convention and copy-paste, not enforced by code.

---

## The `--chart-*` palette

13 tokens, defined identically in `:root` (`index.css:38-50`), `body.ace-hub-theme-active` (`84-96`), and `[data-direction='B']` (`144-156`). **9 are live, 4 are dead.**

| Token | Light | Dark | Alias of | Status | Role |
|---|---|---|---|---|---|
| `--chart-1` | `40 100% 51%` | *invariant* | `--primary` | live (20) | Series 1 — amber |
| `--chart-2` | `103 18% 50%` | *invariant* | `--accent` | live (21) | Series 2 — olive |
| `--chart-3` | `203 32% 22%` | `203 21% 53%` | `--secondary` | live (19) | Series 3 — slate |
| `--chart-4` | `14 65% 55%` | `14 72% 62%` | `--destructive` | live (14) | Series 4 — terracotta |
| `--chart-5` | `0 0% 38%` | `0 0% 72%` | — | live (11) | Series 5 — grey (**inverts** across themes) |
| `--chart-6` | `0 0% 64%` | `0 0% 54%` | — | **dead (0)** | — |
| `--chart-7` | `0 0% 48%` | `0 0% 38%` | — | **dead (0)** | — |
| `--chart-8` | `0 0% 20%` | `0 0% 84%` | — | **dead (0)** | — |
| `--chart-up` | `103 18% 50%` | *invariant* | ≈`--accent` | live (9) | Favourable variance |
| `--chart-down` | `14 65% 55%` | `14 72% 62%` | **≡`--destructive`** | live (5) | Unfavourable variance |
| `--chart-neutral` | `0 0% 45%` | `0 0% 62%` | — | **dead (0)** | — |
| `--chart-grid` | `0 0% 76%` | `0 0% 30%` | — | live (12) | Every `CartesianGrid` |
| `--chart-axis` | `0 0% 42%` | `0 0% 68%` | — | live (33) | Every axis stroke, tick, legend |

**Series order:** 1 → 2 → 3 → 4 → 5. Use them in order; don't skip.
**`--chart-down` is byte-identical to `--destructive` in both themes.** The codebase treats them as interchangeable. That's currently safe, and fragile — changing one without the other silently breaks the pairing.

### Consumption syntax — two idioms

```jsx
// Idiom A — components/charts/*: inline string literal in JSX
<Line stroke="hsl(var(--chart-1))" strokeWidth={3} dot={false} />
<Bar  fill="hsl(var(--chart-2))" radius={[3, 3, 0, 0]} />

// Alpha blending (RMProForma_Payback.js:69-71)
<Cell fill={`hsl(var(--chart-up) / ${alpha})`} />

// Idiom B — forecasting stages: pre-built JS constants threaded via `vm`
// RegionalForecastingModule.js:865-866
const chartGridColor = 'hsl(var(--chart-grid))';
const chartAxisColor = 'hsl(var(--chart-axis))';
<CartesianGrid stroke={chartGridColor} />
```

Tailwind also exposes all 13 as `theme.extend.colors.chart.*` (`tailwind.config.js:58-76`), plus redundant aliases `chart.line1-4` duplicating `chart.1-4`. No source file uses the Tailwind chart utilities.

---

## Canonical prop sets

Two conventions coexist. **Pick the one matching where your chart lives.**

### Convention A — `components/charts/*`

```jsx
const axisTickStyle  = { fill: 'hsl(var(--chart-axis))', fontSize: 12 };
const legendFormatter = (value) => <span className="ace-chart-legend-text">{value}</span>;

<div className="h-64">
  <ResponsiveContainer width="100%" height="100%">
    <LineChart data={data} margin={{ top: 8, right: 16, bottom: 8, left: 0 }}>
      <CartesianGrid strokeDasharray="3 3" stroke="hsl(var(--chart-grid))" />
      <XAxis dataKey="period" stroke="hsl(var(--chart-axis))" tick={axisTickStyle} />
      <YAxis stroke="hsl(var(--chart-axis))" tick={axisTickStyle} />
      <Tooltip contentStyle={{
        backgroundColor: 'hsl(var(--card))',
        border: '1px solid hsl(var(--border))',
        borderRadius: '10px',
      }} />
      <Legend formatter={legendFormatter} />
      <Line dataKey="value" stroke="hsl(var(--chart-1))" strokeWidth={3} dot={false} />
    </LineChart>
  </ResponsiveContainer>
</div>
```

The `axisTickStyle` / `legendFormatter` pair is copy-pasted verbatim into 5 of 6 chart files.

### Convention B — `pages/forecasting/stages/*` (vm-injected)

```jsx
const { CartesianGrid, ComposedChart, XAxis, YAxis, Tooltip, Legend, Line, Bar,
        ResponsiveContainer, chartAxisColor, chartGridColor, tooltipStyle } = vm;

<ResponsiveContainer width="100%" height={280}>
  <ComposedChart data={rows}>
    <CartesianGrid stroke={chartGridColor} vertical={false} strokeDasharray="3 5" />
    <XAxis dataKey="year" tick={{ fill: chartAxisColor, fontSize: 11 }} />
    <YAxis tick={{ fill: chartAxisColor, fontSize: 10 }} />
    <Tooltip contentStyle={tooltipStyle} formatter={formatMillions} />
    <Legend wrapperStyle={{ color: chartAxisColor, fontSize: 11 }} />
    <Bar  dataKey="plan"   fill="hsl(var(--chart-2))" radius={[3, 3, 0, 0]} />
    <Line dataKey="actual" stroke="hsl(var(--chart-1))" strokeWidth={2} dot />
  </ComposedChart>
</ResponsiveContainer>
```

### Where they differ

| | Convention A | Convention B |
|---|---|---|
| Grid dash | `"3 3"` | `"3 5"` |
| Grid direction | both axes | `vertical={false}` (horizontal only) |
| Tick font size | 12 | 10-11 |
| Legend | `formatter` + `.ace-chart-legend-text` | `wrapperStyle` inline |
| Tooltip | per-file `contentStyle` object | shared `tooltipStyle` from `vm` |
| Colour source | inline `hsl(var(--chart-N))` | `chartAxisColor` consts; series still inline |

### Line weight
`strokeWidth={3}` for primary series (`AssetBalanceChart.js:154`, `NetIncomeChart.js:76-77`, `RoaRoeChart.js:76-79`, `RMProForma_FinancialStatements.js:136, 283`); `strokeWidth={2}` in the stages. `RMProForma_YieldCurve.js:82` uses `isCurrent ? 3 : 2` to distinguish the live curve. Forecast/prior series use `strokeDasharray="4 3"` or `"5 4"` with `dot={false}`.

### Tooltips — three implementations
1. `contentStyle` with `backgroundColor` (`AssetBalanceChart.js:134-138`, `NetIncomeChart.js:63-67`)
2. `contentStyle` with `background` and `borderRadius:'12px'` (`RMProForma_YieldCurve.js:56-59`) — same intent, different key and radius
3. Custom `content={customTooltip}` rendering `<div className="ace-tooltip">` (`RMProForma_FinancialStatements.js:71-87`, `RMProForma_Payback.js:74-89`)

**Prefer (3)** — `.ace-tooltip` (`index.css:495-501`) is token-backed and theme-safe; the inline `contentStyle` objects duplicate that styling by hand.

---

## Up / down / neutral semantics

**There are three parallel, conflicting systems.** A designer asking "what is the app's green?" gets three answers.

| System | Positive | Negative | Where |
|---|---|---|---|
| **Chart tokens** (canonical) | `--chart-up` ≈ `#739666` olive | `--chart-down` ≈ `#D7653F` terracotta | `RMProForma_Payback.js:69-71`; `.variance-pos`/`.variance-neg` (`index.css:3842-3843`); `.pos`/`.neg` (`2871-2874`) |
| **Tailwind `icon.*`** | `#02a88e` vivid teal | `#e5241d` vivid red | `tailwind.config.js:84-88` — **0 usages**, dead |
| **Ad-hoc JS constants** | `GREEN = '#2f9e6e'` | `RED = '#d9534f'` | Redeclared in 6 files: `ForecastNiiBuildup.js:6-9`, `ForecastExecView.js:19-22`, `ForecastOp03Suite.js:18-20`, `ForecastGuide.js:11-15`, `stages/CorporateProjectProFormaStage.js:16-19`, used raw in `ForecastReviewDetail.js:476, 512`, `ForecastReviewConsole.js:1259-1284` |

Plus a fourth in RM Pro Forma's input flash: `--accent` up / `--destructive` down (`RMProForma_Dashboard.js:221`).

**Every distinct green in the codebase:** `#739666`/`#85aa76` (`--ace-positive`), `#02a88e` (Tailwind, dead), `#2f9e6e` (ad-hoc, 6 files), `#5d8a66` (`--ace-moss` fallback).
**Every distinct red:** `#d95f3d`/`#e6724f` (`--ace-negative`), `#e5241d`/`#e61d24` (Tailwind, dead), `#d9534f` (ad-hoc, 6 files), `#c4554d` (`--ace-red` fallback).

**Asymmetry worth knowing:** `.forecast-home-table .pos` uses `--chart-up` but `.neg` uses `--destructive`, *not* `--chart-down` (`index.css:2871-2874`). Safe only because those two tokens hold identical values.

**Rule for new work:** `hsl(var(--chart-up))` / `hsl(var(--chart-down))`. Nothing else.

---

## The `vm` chart-theme injection

`chartAxisColor`, `chartGridColor`, and `tooltipStyle` are defined **once** at `RegionalForecastingModule.js:865-866` (+ `tooltipStyle` nearby) and threaded into every stage through a single `vm` object literal built per stage-switch branch (`:5066` baseline, `:5069` context, `:5078` results).

The `vm` bag carries: Recharts primitives themselves, custom components (`WaterfallChart`, `ForecastGrowthMatrix`, `SourceBadge`, `NumericInput`), constants (`WORKSPACE_READINESS`, `MACRO_SETS`, `FORECAST_YEARS`), formatters (`formatMillions`, `formatPercent`), and the three chart-theme values. Stages destructure it at their first line (`RegionalBaselineStage.js:5`, `RegionalContextStage.js:12`, `RegionalResultsStage.js:7`).

Tests stub the slice with plain hex: `chartAxisColor: '#000', chartGridColor: '#ccc'` (`RegionalContextStageMacroStale.test.js:36`, `RegionalResultsStageAnchor.test.js:51`).

**`ForecastNiiBuildup.js` breaks the pattern** — rendered inside a `vm` stage (`RegionalResultsStage.js:495-502`) but imports Recharts directly and takes explicit props. See DRIFT §4.

---

## The 8 components in `components/charts/`

**Two of them are not charts.** `MonthlyAnnualBalanceSheet.js` and `MonthlyAnnualIncomeStatement.js` import **zero** Recharts primitives — verified by grep. They are expand/collapse `<table>` components living in `charts/` by convention only. Documented as `COMPONENTS.md D5`.

| Component | Props | Recharts | Loading / error |
|---|---|---|---|
| `AssetBalanceChart` | `{assetBalances, startDateISO, months=60}` | LineChart | empty-state card only; **8 `console.log`s left in** |
| `NetIncomeChart` | `{chartData, startDateISO, months=60}` | LineChart, 2 Lines | **none at all** |
| `RoaRoeChart` | `{chartData, startDateISO, months=60}` | LineChart, 4 Lines | **none at all** |
| `MonthlyAnnualBalanceSheet` | `{annualSummary, monthlySchedule, totalMetrics}` | — *table* — | none |
| `MonthlyAnnualIncomeStatement` | `{annualSummary, monthlySchedule, totalMetrics}` | — *table* — | empty-state card (asymmetric with its sibling) |
| `RMProFormaStatements` | `{annualSummary, monthlySchedule, isLoading, error}` | 8 Bar / 4 Line / 2 ComposedChart | Loader2 + destructive alert |
| `RMCumulativePayback` | `{annualSummary, monthlySchedule, cumulativePayback, paybackDate, isLoading, error}` | BarChart + Cell | Loader2 + destructive alert (hand-duplicated) |
| `RMProFormaYieldCurve` | `{yieldCurves}` | LineChart | `!yieldCurves` text fallback only |

**Shape conventions that do hold:**
- Time-series charts take `{data, startDateISO, months = 60}` and compute their own domain/ticks via `useMemo`.
- RM Pro Forma charts take `{annualSummary, monthlySchedule, isLoading, error}`.
- `RMProFormaStatements` is a *panel*, not a chart — it renders 4 sub-charts internally.

**3 of 8 have no loading/error handling whatsoever.** Two duplicate identical Loader2/destructive JSX by copy-paste. No shared component absorbs this.

---

## Drift findings

1. **4 of 13 chart tokens are dead** — `--chart-6/7/8`, `--chart-neutral` defined in all three theme blocks and in Tailwind, consumed nowhere.
2. **Three conflicting up/down colour systems** (above). The ad-hoc `GOLD/GREEN/RED` constants are redeclared in 6 files and **respond to neither theme** — no `[data-direction='B']` variant exists for a JS constant.
3. **`GOLD = '#FFAC03'` is a hand-written literal of `--primary`** (`hsl(40 100% 51%)` ≈ `#FFAB05`). Six files carry a dead copy of the brand colour.
4. **`ForecastNiiBuildup.js` does not respond to the dark theme.** Its `CartesianGrid` has no `stroke=` (Recharts default grey), its axes have no `tick` fill, and its `<Tooltip>` has no `contentStyle` — so it renders a white tooltip on a dark canvas. **The clearest dark-mode defect in the charting layer.**
5. **Tooltip implemented 3 ways** across 6 chart files (above).
6. **`RMProForma_YieldCurve.js:56`** uses `background:` where its siblings use `backgroundColor:`, with a different `borderRadius` (12px vs 10px). Renders fine; inconsistent.
7. **Two of the eight "charts" are tables** (above) — misleading directory placement.
8. **Loading/error handling is inconsistent across all 8** — 3 none, 2 copy-pasted, 1 partial, 1 bespoke.
9. **`console.log` left in production paths** — `AssetBalanceChart.js:10, 16, 37, 54, 60, 82, 87, 121`; also `RMProForma_FinancialStatements.js:8`, `RMProForma_Payback.js:8`.

---

## Checklist for a new chart

1. Live in `components/charts/` only if it renders Recharts.
2. Use Convention A (standalone) or B (inside a `vm` stage) — don't blend them.
3. Series colours in order: `hsl(var(--chart-1))` → `--chart-5`. Never a hex literal.
4. Directional colour: `--chart-up` / `--chart-down`. Never `GREEN`/`RED` constants.
5. Grid `stroke="hsl(var(--chart-grid))"`; axes `stroke` + `tick.fill` `hsl(var(--chart-axis))`. **Always set all three** — omitting them is what breaks dark mode.
6. Tooltip: custom `content` rendering `.ace-tooltip`.
7. Legend: `formatter` returning `<span className="ace-chart-legend-text">`.
8. Accept and render `isLoading` and `error`.
9. `ResponsiveContainer width="100%" height="100%"` inside a fixed-height wrapper.
10. No `console.log`.
