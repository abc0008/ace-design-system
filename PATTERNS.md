# Composite Patterns

Multi-component recurring layouts that Forecast and RM Pro Forma are built from. Where `COMPONENTS.md` catalogues atoms, this catalogues how they compose.

Each pattern lists its parts in composition order, a canonical snippet distilled from real JSX, usage evidence, and tokens consumed.

---

## P1. Panel + head + body — the universal content block
**The structural spine of the product.** Every screen is a stack or grid of these.

**Parts:** `.forecast-panel` › `.forecast-panel-head` › (`.forecast-section-code` + `h2`) › body

```jsx
<section className="forecast-panel">
  <div className="forecast-panel-head">
    <div>
      <span className="forecast-section-code">EX_00 ● NII composition — FTP basis</span>
      <h2>Lending spread vs deposit franchise vs treasury mismatch ($MM, by year)</h2>
    </div>
    <TrendingUp size={16} />
  </div>

  <div className="forecast-table-shell ace-scroll-shell">
    <table className="forecast-table">…</table>
  </div>

  <p className="forecast-submit-note">FTP basis; excludes intercompany eliminations.</p>
</section>
```

**Evidence:** 100 panel / 91 head / 112 code occurrences across 15 files. `ForecastExecView.js:902, 968, 992`, `ForecastReviewConsole.js:1075`, `stages/RegionalContextStage.js:193`, `ForecastNiiBuildup.js:119`, `ForecastProjectWorkspace.js:804`
**Tokens:** `--ace-line`, `--ace-panel-strong`, `--foreground`, `--muted-foreground`, `--primary`, `--section-color`
**Variants:** `.forecast-panel-lite`, `.forecast-panel-tall`, `.forecast-subpanel` (nested, `h3`), `.forecast-exec-pair` (two-up)
**Section identity:** when the panel sits under an id like `#forecast-macro`, `--section-color` tints the code and the left border (`index.css:3760-3799`).

**Rule to follow:** a panel body is one of exactly four things — a table (P3/P4), a chart, a chip row (P2), or a stack of subpanels. Nothing in the codebase does anything else.

---

## P2. Action bar + note — the decision footer
Closes nearly every panel that accepts input.

**Parts:** `.forecast-submit-chips` (or `.forecast-submit-actions`) › buttons › `.forecast-submit-note`

```jsx
<div className="forecast-submit-actions">
  <button type="button" className="forecast-primary-button" disabled={busy} onClick={confirm}>
    <RefreshCw size={14} /> Re-anchor to v{publishedVersion}
  </button>
  <button type="button" className="forecast-secondary-button" onClick={close}>Close</button>
  <span className="forecast-submit-note">Reviewer must return the package before re-anchoring.</span>
</div>
```

**Evidence:** 44 chip-row + 116 note occurrences. `stages/RegionalContextStage.js:160`, `ForecastCycleHome.js:257`, `ForecastOp03Suite.js:844`, `ForecastPromptBox.js:235`
**Tokens:** `--primary`, `--primary-foreground`, `--ace-line`, `--foreground`, `--muted-foreground`, `--destructive`
**States:** buttons `:disabled` at 0.5 opacity; note swaps to `.error-list` in `--destructive`
**⚠ Drift:** `.forecast-submit-actions` and `.forecast-submit-chips` are used interchangeably for the same job — only `RegionalContextStage.js` reaches for the semantic one. FINDINGS F-13.

---

## P3. Read table — roster / log / summary
Narrow tabular data with an empty state and sign-coloured numerics.

**Parts:** `.forecast-home-table-wrap` › `table.forecast-home-table` › cells decorated with `.forecast-mono`, `.pos`/`.neg`, chips, `.forecast-home-action.ghost`

```jsx
<div className="forecast-home-table-wrap">
  <table className="forecast-home-table">
    <thead><tr><th>Division</th><th>Status</th><th>NI Δ</th><th>When</th><th /></tr></thead>
    <tbody>
      {rows.length === 0 ? (
        <tr><td colSpan={5}><span className="dim-note">No packages submitted.</span></td></tr>
      ) : rows.map((row) => (
        <tr key={row.id} className={row.isMine ? 'mine' : ''}>
          <td>{row.division}</td>
          <td><span className="forecast-health-chip neutral mini">{row.status}</span></td>
          <td className={`forecast-mono ${row.delta >= 0 ? 'pos' : 'neg'}`}>{row.delta.toFixed(1)}</td>
          <td className="forecast-mono">{fmtWhen(row.createdAt)}</td>
          <td><button type="button" className="forecast-home-action ghost">Diff</button></td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

**Evidence:** 9 files. `ForecastAuditTrail.js:113`, `ForecastExecView.js:906`, `ForecastReviewConsole.js` (×4), `ForecastOp03Suite.js` (×3), `ForecastMacroScenarios.js` (×3), `ForecastCycleHome.js`, `ForecastProjects.js`
**Tokens:** `--ace-line`, `--muted-foreground`, `--primary` (`tr.mine`), `--chart-up` (`.pos`), `--destructive` (`.neg`)
**Conventions worth keeping:** mono uppercase `th`; `colSpan` + `.dim-note` empty state; chip in the status column; ghost action button in the last, header-less column.

---

## P4. Dense matrix — the wide editable grid
Where the actual forecasting happens. Distinct from P3 in every respect except being a table.

**Parts:** `.forecast-table-shell.ace-scroll-shell[.forecast-compact-table]` › `table.forecast-table[.forecast-time-matrix]`

```jsx
<div className="forecast-table-shell ace-scroll-shell forecast-compact-table">
  <table className="forecast-table forecast-time-matrix">
    <thead><tr>
      <th>Metric · $MM</th>
      {columns.map((col) => (
        <th key={col.label} className={col.actual ? 'time-col yt' : 'time-col y'}>
          {col.label}{col.sub && <small className="growth-col-sub">{col.sub}</small>}
        </th>
      ))}
    </tr></thead>
    <tbody>
      {METRICS.map((metric) => (
        <tr key={metric.key}>
          <td><strong>{metric.label}</strong></td>
          {columns.map((col) => (
            <td key={col.label} className="time-cell growth-cell">
              <span className="forecast-mono">{fmt(val, metric.digits)}</span>
              <small className={favorable ? 'variance-pos' : 'variance-neg'}>{delta.toFixed(1)}%</small>
            </td>
          ))}
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

**Evidence:** `stages/RegionalResultsStage.js:88, 127, 192` (6 total), `stages/RegionalContextStage.js` (×3), `stages/RegionalBaselineStage.js`, `ForecastGrowthMatrix.js:42`, `ForecastPeerProForma.js`
**Tokens:** `--ace-line`, `--ace-line-strong`, `--ace-panel-strong` (sticky th), `--ace-input-bg`, `--primary` (focus), `--chart-up`/`--destructive` (variance)
**Column typing:** `th.time-col` × `.q` (quarter) `.m` (month) `.yt` (year-to-date) `.ya`/`.actual` `.yr` (forecast year) — a real, reusable convention for period-typed columns.
**Cell shape:** value in `.forecast-mono` + delta in `<small class="variance-pos|neg">` is the canonical two-line numeric cell.
**Sibling implementations:** `.op3-year-table` (`ForecastOp03Suite.js`, ×3) and `.macro-diff-table` (`ForecastMacroScenarios.js`, ×3) are the same idea under different names.

---

## P5. Workspace stage flow — the multi-step model builder
The spine of `RegionalForecastingModule.js` and `ForecastProjectWorkspace.js`: a linear stage sequence, one stage rendered at a time.

**Parts:** stage nav › `.workspace-stage-panel` › `.forecast-accordion` sections › `.workspace-action-row`

```jsx
{/* orchestrator: RegionalForecastingModule.js:5066-5078 */}
activeStageContent = <RegionalContextStage vm={{ CartesianGrid, ComposedChart, Line, XAxis, YAxis,
  Legend, Tooltip, ResponsiveContainer, chartAxisColor, chartGridColor, tooltipStyle,
  formatMillions, formatPercent, model, activeWorkspaceStage, setWorkspaceState, /* …~30 keys */ }} />;

{/* stage: stages/RegionalContextStage.js:12, 53 */}
const { CartesianGrid, ComposedChart, chartAxisColor, chartGridColor, tooltipStyle, … } = vm;

<section id="regional-stage-context" className="workspace-stage-panel">
  <details className="forecast-accordion" open>
    <summary>
      <div>
        <span className="forecast-section-code">OP_02 · Macro environment</span>
        <h2>Base economic scenario and annual divergence</h2>
      </div>
      <SlidersHorizontal size={18} />
    </summary>
    <div className="forecast-accordion-body">
      <section className="forecast-panel">…</section>
    </div>
  </details>
</section>
```

**Evidence:** `RegionalForecastingModule.js:5066, 5069, 5078`; `stages/RegionalBaselineStage.js:5`, `stages/RegionalContextStage.js:12`, `stages/RegionalResultsStage.js:7`; `ForecastProjectWorkspace.js:644, 674, 804`
**Tokens:** `--ace-line`, `--ace-panel-strong`, `--section-color`, `--primary`, `--muted-foreground`
**The `vm` prop bag is the defining mechanic:** the parent injects *everything* — Recharts primitives, custom components, constants, formatters, and the chart theme strings — into one object per stage. Stages import almost nothing directly.

**Trade-off, stated plainly:** this keeps stages dependency-free and trivially testable (the tests stub `vm` wholesale — `RegionalContextStageMacroStale.test.js:36`), at the cost of a ~30-key untyped contract with no compile-time checking. It is a deliberate pattern, not an accident, and it is worth preserving *or* replacing wholesale — but not partially. `ForecastNiiBuildup.js` already breaks out of it (imports Recharts directly, takes explicit props) while being rendered *inside* a `vm` stage at `RegionalResultsStage.js:495`. FINDINGS F-20.

**⚠ Two stage-identity conventions:** `ForecastProjectWorkspace.js:674` derives ids programmatically (`project-stage-${activeStage}`) and manages focus; `RegionalContextStage.js:53` hard-codes `id="regional-stage-context"` with no focus handling. The concept is also named two ways: `activeStage` vs `activeWorkspaceStage`.

---

## P6. Labelled distribution track — **four implementations of one idea**
A label block, a horizontal track, and absolutely-positioned markers. This is the strongest composite idea in the product **and its worst duplication.** All four share the shape `label(span + strong + small) → track → markers positioned by left:%`; none share a class.

### P6a. `consol-bridge` — diverging gap bars
`ForecastReviewConsole.js:1078, 1151` (20 occurrences, 1 file). CSS `index.css:4263-4299`
```jsx
<div className="consol-bridge">
  {rows.map((row) => (
    <div className="consol-bridge-row" key={row.division}>
      <span className="consol-bridge-label">{row.division}</span>
      <span className="consol-bridge-track">
        <span className={`consol-bridge-bar ${row.niGapMM >= 0 ? 'pos' : 'neg'}`}
              style={{ width: `${(Math.abs(row.niGapMM) / maxAbs) * 100}%` }} />
      </span>
      <span className={`forecast-mono ${row.niGapMM >= 0 ? 'pos' : 'neg'}`}>{row.niGapMM.toFixed(1)}</span>
    </div>
  ))}
</div>
```

### P6b. `pulse-card` — progress meter
`ForecastReviewConsole.js:703-731`. CSS `index.css:4305-4328`
```jsx
<div className="pulse-grid">
  <div className="pulse-card">
    <span>Submitted+</span>
    <strong>{pulse.done}<em>/{pulse.total}</em></strong>
    <span className="pulse-track"><span className="pulse-fill moss" style={{ width: `${pct}%` }} /></span>
  </div>
</div>
```
Variants `.pulse-fill.moss/.gold/.amber/.red` — **`.gold` and `.amber` are hard-coded `#FFAC03` / `#d99a2b`**, not tokens.

### P6c. `forecast-bullet-row` — guardrail bullet chart
`ForecastGuardrailBoard.js:52`, rendered once at `ForecastExecView.js:1215`
```jsx
<div className={`forecast-bullet-row ${row.status}`}>
  <div className="forecast-bullet-label"><span>{row.metric}</span><strong>{row.display}</strong><small>{row.driver}</small></div>
  <div className="forecast-bullet-track" role="img" aria-label={`${row.metric}: ${row.display}; ${row.threshold}.`}>
    <div className="forecast-bullet-bands" aria-hidden="true">
      {row.bands.map((b) => <span className={`forecast-bullet-band ${b.tone}`} style={{ left: `${b.x}%`, width: `${b.w}%` }} />)}
    </div>
    <span className="forecast-bullet-actual" style={{ width: `${actualPct}%` }} aria-hidden="true" />
    <span className="forecast-bullet-target" style={{ left: `${targetPct}%` }} aria-hidden="true" />
  </div>
  <div className="forecast-bullet-status">
    <span className={`forecast-status-pill ${row.status}`}><StatusIcon size={13} />{STATUS_LABEL[row.status]}</span>
    <small>{row.threshold}</small>
  </div>
</div>
```

### P6d. `forecast-peer-distribution-row` — peer range dotplot
`ForecastPeerProForma.js:108`
```jsx
<div className="forecast-peer-distribution-row">
  <div className="forecast-peer-distribution-label"><span>{row.label}</span><strong>{fmt(row.value)}</strong><small>Median {fmt(row.median)}</small></div>
  <div className="forecast-peer-distribution-track" role="img" aria-label={`${row.label}: company ${fmt(row.value)} versus peer range`}>
    <span className="forecast-peer-range-line" />
    {row.peerPositions.map((p) => <span className="forecast-peer-dot" style={{ left: `${p.position}%` }} title={`${p.peer}: ${fmt(p.value)}`} />)}
    <span className="forecast-peer-marker" />
  </div>
</div>
```

**P6c and P6d are structurally near-identical** — same three-part label, same `role="img"` + `aria-label` track, same `left:%` markers — with zero shared classes. **This is the highest-value consolidation target in the codebase.** FINDINGS F-21.

**Accessibility convention worth adopting product-wide:** P6c/P6d put `role="img"` + a full-sentence `aria-label` on the track and `aria-hidden` on every decorative marker. That is the correct treatment and neither P6a nor P6b does it.

---

## P7. Stat strip — two implementations
**P7a. `forecast-status-strip`** — `<section>`, items are `span` + `strong`. `ForecastCycleHome.js:200-229` (5 items)
```jsx
<section className="forecast-status-strip">
  <div><span>Due date</span><strong><CalendarClock size={13} />{dueLabel}</strong></div>
  <div><span>Status</span><strong>{cycle.status}</strong></div>
</section>
```
**P7b. `forecast-kpi-grid`** — `<div>`, items are `span` + `strong` + `small`. `stages/RegionalContextStage.js:326-357` (6 items)
```jsx
<div className="forecast-kpi-grid">
  <div><span>Net hires</span><strong>{model.netHires}</strong>
       <small>{model.grossHires} gross / {model.departures} departures</small></div>
</div>
```
They share CSS at `index.css:827-840` and diverge at `807-825` vs `1456-1480` — evidence they were meant to be one component. No file uses both. FINDINGS F-22.

---

## P8. Notification / audit list
Unread-aware item list in a popover, plus its table-based twin in the audit trail.

```jsx
<div className="forecast-notif-panel forecast-panel" role="menu">
  <div className="forecast-notif-head">
    <span className="forecast-section-code">Notifications</span>
    <button type="button" className="forecast-chip-button" onClick={markAllRead}>Mark all read</button>
  </div>
  <ul className="forecast-notif-list">
    {items.map((n) => (
      <li key={n.id}>
        <button type="button" className={`forecast-notif-item ${n.status === 'unread' ? 'unread' : ''}`}
                onClick={() => open(n)}>
          <span className="forecast-notif-item-top">
            <span className={`forecast-health-chip ${PRIORITY_TONE[n.priority] || 'neutral'} mini`}>{n.type}</span>
            <strong>{n.title}</strong>
            <em className="forecast-mono">{formatAge(n.ageHours)}</em>
          </span>
          {n.body && <span className="forecast-notif-item-body">{n.body}</span>}
        </button>
      </li>
    ))}
  </ul>
</div>
```
**Evidence:** `ForecastNotificationsTray.js:116-146`; audit twin at `ForecastAuditTrail.js:113-175` using `.forecast-audit-diff-row` / `.forecast-diff-arrow` with `.forecast-mono.neg` → `.forecast-mono.pos` before/after pairs.
**⚠ Two defects:** `.forecast-notif-head` reimplements P1's panel-head with a bespoke class (F-23); `.forecast-chip-button` is **undefined in CSS** so that button renders unstyled (F-19).

---

## P9. AI prompt / narrative box
Explicit four-state machine, expressed in markup.

```jsx
<div className="forecast-prompt-box">
  <div className="forecast-panel-head">
    <div><span className="forecast-section-code">OP_03P · AI prompt box</span>
         <h3><Wand2 size={15} style={{ color: PURPLE }} />Describe a change in plain English…</h3></div>
  </div>
  <div className="forecast-prompt-input">
    <textarea rows={2} disabled={readOnly} aria-label="Describe the forecast change" />
    <button type="button" className="forecast-primary-button compact" onClick={draft}>Draft</button>
  </div>

  {state === 'idle' && (
    <div className="forecast-prompt-examples">
      {EXAMPLES.map((ex) => (
        <button key={ex} type="button" className="forecast-health-chip clickable neutral"
                onClick={() => setPrompt(ex)} disabled={readOnly}>{ex}</button>
      ))}
    </div>
  )}
  {state === 'error' && <p className="forecast-submit-note forecast-prompt-error">{error}</p>}
  {(state === 'ready' || state === 'applied') && (
    <div className="ai-draft-panel forecast-prompt-proposal">
      <div className="forecast-submit-chips">{/* accept / reject / modify */}</div>
    </div>
  )}
</div>
```
**Evidence:** `ForecastPromptBox.js:204-283`; draft-panel twin at `ForecastOp03Suite.js:844`
**States:** `idle | ready | applied | error` — worth preserving as the pattern's contract.
**⚠** The purple accent is hard-coded `#9C6ADE` in both JS (`ForecastPromptBox.js:19`) and CSS (`index.css:4049`). No token. F-04.

---

## P10. RM Pro Forma entry workspace
The whole RMPF page in one shape — and note it shares only `card-wrapper` with everything above.

```jsx
<section className="ace-rm-overview">
  <span className="ace-rm-eyebrow">OP_PRLX · rm-pro-forma</span>
  <h1 className="ace-rm-title">Model every RM hire with <span className="ace-serif text-primary">hub-grade consistency.</span></h1>
  <p className="ace-rm-description">…</p>
</section>

<div className="flex gap-2">{/* D2 entry tab pills */}</div>

<Card className="card-wrapper overflow-hidden">
  <CardHeader className="border-b border-border/60 bg-card/45"><CardTitle>Assumptions</CardTitle></CardHeader>
  <CardContent>
    <Accordion type="single" collapsible>
      <AccordionItem value="rm-type-start"><AccordionTrigger>RM Type &amp; Start</AccordionTrigger>
        <AccordionContent>{/* CustomInput grid */}</AccordionContent></AccordionItem>
      <AccordionItem value="item-1"><AccordionTrigger>Balance Sheet: Volume, Pricing &amp; Dynamics</AccordionTrigger>…</AccordionItem>
      <AccordionItem value="item-2"><AccordionTrigger>Fee Revenue</AccordionTrigger>…</AccordionItem>
      <AccordionItem value="item-3"><AccordionTrigger>Direct Expenses</AccordionTrigger>…</AccordionItem>
    </Accordion>
    <Button type="submit" className="mt-4">Calculate Pro Forma</Button>
  </CardContent>
</Card>
```
**Evidence:** `RMProForma_Dashboard.js:843-853, 854-858, 905-926, 1121-1300, 1303`
**Tokens:** `--ace-panel`, `--ace-line`, `--ace-shadow`, `--primary`, `--secondary`, `--accent`, `--muted-foreground`
**Note the parallel to P1:** `ace-rm-eyebrow` is `forecast-section-code` reinvented; `card-wrapper` is `forecast-panel` reinvented; the Radix accordion is `forecast-accordion` reinvented. Three pairs, zero shared code.

---

## Pattern-to-component index

| Pattern | Components used |
|---|---|
| P1 Panel block | A1, A2, A5, A13 |
| P2 Action bar | A6, A7, A4 |
| P3 Read table | A8, A3, A13 |
| P4 Dense matrix | A9, B1, A13 |
| P5 Stage flow | A12, A1, A2, P4 |
| P6 Distribution track | A13, bespoke ×4 |
| P7 Stat strip | bespoke ×2 |
| P8 Notification list | A1, A3, A5, A13 |
| P9 Prompt box | A2, A3, A6, A4, A7 |
| P10 RMPF workspace | B2, B7, C (ui layer), D1, D2 |

## Gap — not documented here
`ForecastWorkspaceShell` (imported at `ForecastProjectWorkspace.js:8`, rendered at `:691`) provides the outer nav chrome for the workspace flow. Its source was outside the sampled set, and the classes it would own (`forecast-workspace-shell`, `workspace-command-header`, `workspace-stage-content`) have **zero** className occurrences in the sampled files. **UNVERIFIED** — if the shell needs documenting, stage that file and re-run.
