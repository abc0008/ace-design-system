# FRAME — the token layer

**FRAME is the primitive layer: the 49 custom properties, what they mean, which theme block defines them, and how to consume each one without breaking.** It answers *what exists and what is it called*. `DESIGN.md` answers *what do I do with it*.

Everything here is extracted from `frontend/src/index.css` in the BankAnalysis app, which remains canonical. `tokens.css` and `tokens.json` in this repo are mirrors; editing them changes nothing. If a statement here disagrees with `index.css`, `index.css` is right and this file is stale.

> **Naming provenance.** This repo normalises the source app's brand prefixes: `--ace-*` → `--brand-*`,
> `.ace-*` → `.brand-*`, `body.ace-hub-theme-active` → `body.brand-hub-theme-active`, `.ace-brand-mark` →
> `.brand-mark`, and `--aa-ink`/`--aa-accent` → `--brand-ink`/`--brand-accent`. **The source application still
> uses the original `ace-`/`aa-` prefixes.** So every `file:line` citation in this repo points at a source line
> where the identifier appears with its original prefix — the line numbers are accurate, only the prefix differs.
> Citations are not rewritten; see `README.md` § Naming provenance for the full mapping and the one token merge
> (`--ace-ink` + `--aa-ink` → `--brand-ink`) it produces.

---

## 1. The shape of the system

49 distinct custom-property names, in four families:

| Family | Count | Format | Defined in | Consume as |
|---|---|---|---|---|
| shadcn semantic | 20 | bare HSL channels (+ `--radius`, a length) | all three blocks | `hsl(var(--x))` |
| chart | 13 | bare HSL channels | all three blocks | `hsl(var(--chart-x))` |
| Brand surface (`--brand-*`) | 10 | complete `rgba()` / hex / shadow values | **theme blocks only** | `var(--brand-x)` |
| component-scoped | 6 | mixed | on individual selectors | read by nearby rules only |

20 + 13 + 10 + 6 = **49**. The first two families (33 properties) are the global scaffolding; the `--brand-*` family is the surface language; the component-scoped six are local variables that happen to use custom-property syntax.

Source lines: `:root` `index.css:7-51` · Theme A `55-107` · Theme B `115-167` · component-scoped `217-218` and `3792-3795`.

---

## 2. Theme architecture — read this before anything else

**There is no `.dark` class.** Themes are body-scoped, and they cascade in three layers:

```
:root                                            33 tokens   (20 shadcn + 13 chart)
  │                                              light values, NO --brand-* family
  │
  ▼
body.brand-hub-theme-active                        43 tokens   Theme A / light
  │                                              same 33, restated + the 10 --brand-*
  │
  ▼
body.brand-hub-theme-active[data-direction='B']    43 tokens   Theme B / dark
                                                 same names, dark values
```

| Selector | Role | Tokens |
|---|---|---|
| `:root` | Base scaffolding, light values. **No `--brand-*`** | 33 |
| `body.brand-hub-theme-active` | Theme A / light | 43 |
| `body.brand-hub-theme-active[data-direction='B']` | Theme B / dark | 43 |

Activation is three lines in `frontend/src/App.js:57-59`:

```js
document.documentElement.setAttribute('data-direction', direction);
document.body.classList.add('brand-hub-theme-active');
document.body.setAttribute('data-direction', direction);
```

Note that `data-direction` is set on **both** `<html>` and `<body>`. The theme selector keys off the one on `<body>`; the one on `<html>` exists for anything that needs to read direction above the body. Any harness that reproduces the theme must mirror both — `gallery.html` does exactly this.

### 2.1 The hard rule (F-06)

> **Anything that renders outside `body.brand-hub-theme-active` loses every `--brand-*` token.**

Not "degrades" — loses. The family is defined only inside that class, and `:root` has none of it. There is no fallback layer.

What that costs you: `--brand-line` is the app's default border and the **second-most-consumed token in the codebase (179 uses)**. Also gone: every panel fill (`--brand-panel`, `--brand-panel-strong`), the input background (`--brand-input-bg`), elevation (`--brand-shadow`), and the positive/negative/warning value colours. Borders, fills and shadows all vanish **simultaneously and silently** — no console error, no invalid-property warning. The surface just renders flat and borderless and looks like a CSS load failure.

The failure modes that actually hit this: a React portal mounted to a detached root, a print stylesheet, an isolated test render, a Storybook-style component harness, an email or PDF export path, an iframe.

**If you build any harness that renders these components, put `class="brand-hub-theme-active"` on its `<body>` and `data-direction` on both `<html>` and `<body>`.** That is the entire reason `gallery.html` in this repo carries those attributes.

The standing recommendation (not yet applied to the app) is to duplicate the light `--brand-*` block into `:root` as a safe default. It is purely additive and changes nothing visually inside the app, because the theme block would still win by specificity — it just removes this whole class of failure.

---

## 3. The two consumption conventions

The single most common way to break a token in this system is to consume it with the wrong syntax. There are two conventions and they are not interchangeable.

### 3.1 HSL-channel tokens — the shadcn and chart families (33 tokens)

These hold **bare HSL channel triples**, not colours. `--primary` is the string `40 100% 51%`. It is not a colour and cannot be used as one.

```css
color: hsl(var(--primary));                    /* correct */
background: hsl(var(--chart-up) / 0.4);        /* correct — slash alpha */
border: 1px solid hsl(var(--border));          /* correct */

color: var(--primary);                         /* WRONG — renders nothing */
```

The last line fails silently: `color: 40 100% 51%` is invalid, the declaration is dropped, and the element inherits. No error anywhere.

The payoff for the awkwardness is free alpha — `hsl(var(--chart-up) / 0.4)` needs no second token. The codebase also uses `color-mix()` on top of this for tonal fills:

```css
background: color-mix(in srgb, hsl(var(--primary)) 13%, transparent);
```

### 3.2 Complete-value tokens — the `--brand-*` family (10 tokens)

These hold **finished values** — `rgba()`, hex, or a whole `box-shadow`. Consume them bare:

```css
border: 1px solid var(--brand-line);             /* correct */
background: var(--brand-panel-strong);           /* correct */
box-shadow: var(--brand-shadow);                 /* correct — it's a full shadow value */
color: var(--brand-positive);                    /* correct — it's a hex */

border: 1px solid hsl(var(--brand-line));        /* WRONG — double-wrapping */
```

**The rule of thumb:** if the name starts with `--brand-`, use it bare. Everything else gets wrapped in `hsl()`.

`--brand-shadow` deserves a flag: it is a shadow, not a colour. `0 20px 46px rgba(0, 0, 0, 0.11)` in light, `0 24px 56px rgba(0, 0, 0, 0.38)` in dark. It only ever belongs on the right-hand side of `box-shadow`.

---

## 4. The shadcn semantic family (20)

Nineteen colours plus `--radius`. Defined in all three blocks.

| Token | Light | Dark | Uses | Role |
|---|---|---|---|---|
| `--background` | `0 0% 92%` | `0 0% 7%` | 12 | App canvas |
| `--foreground` | `0 0% 7%` | `0 0% 92%` | 136 | Primary text |
| `--card` | `0 0% 94%` | `0 0% 20%` | 17 | Card fill; chart tooltip background |
| `--card-foreground` | `0 0% 7%` | `0 0% 92%` | 1 | Text on card |
| `--popover` | `0 0% 94%` | `0 0% 20%` | 0 | Tailwind-only |
| `--popover-foreground` | `0 0% 7%` | `0 0% 92%` | 0 | Tailwind-only |
| `--primary` | `40 100% 51%` | *invariant* | 103 | **Amber — the brand colour** |
| `--primary-foreground` | `0 0% 7%` | *invariant* | 7 | Ink on amber |
| `--secondary` | `203 32% 22%` | `203 21% 53%` | 8 | Deep slate |
| `--secondary-foreground` | `0 0% 98%` | `0 0% 7%` | 0 | Tailwind-only |
| `--muted` | `0 0% 88%` | `0 0% 16%` | 2 | Muted surface |
| `--muted-foreground` | `0 0% 38%` | `0 0% 82%` | 223 | **Secondary text — most-consumed token in the codebase** |
| `--accent` | `103 18% 50%` | *invariant* | 32 | Olive — positive chips, RMPF increment flash |
| `--accent-foreground` | `0 0% 98%` | *invariant* | 0 | Tailwind-only |
| `--destructive` | `14 65% 55%` | `14 72% 62%` | 44 | Terracotta — errors **and** negative variance |
| `--destructive-foreground` | `0 0% 98%` | *invariant* | 0 | Tailwind-only |
| `--border` | `0 0% 78%` | `0 0% 28%` | 52 | Tailwind `border-*` utilities |
| `--input` | `0 0% 96%` | `0 0% 24%` | 9 | Input surface |
| `--ring` | `40 100% 51%` | *invariant* | 1 | Focus ring — identical to `--primary` |
| `--radius` | `0.75rem` | *not redefined* | — | Tailwind `borderRadius` base |

Four facts about this table that matter more than the values:

**`--muted-foreground` (223 uses) is the most-consumed token in the entire codebase**, ahead of `--brand-line` (179) and `--foreground` (136). Secondary text is the highest-volume design decision in this product. Changing it changes almost every screen.

**Four tokens are theme-invariant by design** — `--primary`, `--primary-foreground`, `--accent`, `--ring` hold the same value in light and dark. The brand amber does not flip. `--primary-foreground` is dark ink (`0 0% 7%`) in *both* themes because amber is a light colour: **never put white on amber.**

**Five tokens have zero direct `var()` consumers** — the `-foreground` pairs plus `--popover`. They are reachable only through Tailwind utilities. They are not dead, but a grep for `var(--accent-foreground)` will return nothing and that is expected.

**`--border` and `--brand-line` are two answers to one question.** Tailwind `border-*` utilities resolve to `--border`; hand-written CSS overwhelmingly prefers `var(--brand-line)`. Both are live. Which one you want depends on which layer you are writing in — see `DESIGN.md` §2.

### 4.1 `--radius` is defined and then ignored (F-27)

`--radius: 0.75rem` is set **only in `:root`** — neither theme block redefines it. It feeds Tailwind's `borderRadius` scale (`lg` = `calc(var(--radius) + 2px)`, `md` = `var(--radius)`, `sm` = `calc(var(--radius) - 2px)`, `DEFAULT` = `0.375rem`).

Hand-written CSS ignores it entirely. The real radii in the product are literals:

| Surface | Radius |
|---|---|
| `.forecast-panel` | `14px` |
| `.card-wrapper` | `18px` |
| table shells | `12px` |
| every pill / chip | `999px` |

Also: the comment at `tailwind.config.js:111` claims `--radius` is `0.375rem`. It is `0.75rem`. The comment is stale — trust the CSS.

**Practical consequence:** do not expect `rounded-lg` and a hand-written panel to agree. They do not, and matching them is a decision, not a default.

---

## 5. The chart family (13)

Defined identically in all three blocks (`index.css:38-50`, `84-96`, `144-156`). **Nine are live, four are dead.**

| Token | Light | Dark | Alias of | Uses | Status |
|---|---|---|---|---|---|
| `--chart-1` | `40 100% 51%` | *invariant* | `--primary` | 20 | live — Series 1, amber |
| `--chart-2` | `103 18% 50%` | *invariant* | `--accent` | 21 | live — Series 2, olive |
| `--chart-3` | `203 32% 22%` | `203 21% 53%` | `--secondary` | 19 | live — Series 3, slate |
| `--chart-4` | `14 65% 55%` | `14 72% 62%` | `--destructive` | 14 | live — Series 4, terracotta |
| `--chart-5` | `0 0% 38%` | `0 0% 72%` | — | 11 | live — Series 5, grey (**inverts**) |
| `--chart-6` | `0 0% 64%` | `0 0% 54%` | — | 0 | **dead** |
| `--chart-7` | `0 0% 48%` | `0 0% 38%` | — | 0 | **dead** |
| `--chart-8` | `0 0% 20%` | `0 0% 84%` | — | 0 | **dead** |
| `--chart-up` | `103 18% 50%` | *invariant* | ≈ `--accent` | 9 | live — favourable variance |
| `--chart-down` | `14 65% 55%` | `14 72% 62%` | ≡ `--destructive` | 5 | live — unfavourable variance |
| `--chart-neutral` | `0 0% 45%` | `0 0% 62%` | — | 0 | **dead** |
| `--chart-grid` | `0 0% 76%` | `0 0% 30%` | — | 12 | live — every `CartesianGrid` |
| `--chart-axis` | `0 0% 42%` | `0 0% 68%` | — | 33 | live — every axis, tick, legend |

**Series order is 1 → 2 → 3 → 4 → 5.** Use them in order; do not skip. `--chart-6/7/8` and `--chart-neutral` are defined in all three CSS blocks *and* mirrored into Tailwind, and consumed by nothing — they are available if a chart genuinely needs a sixth series, but reaching past 5 today means you are the first.

**`--chart-5` is the only series colour that inverts across themes** (`0 0% 38%` light → `0 0% 72%` dark). The other four are either invariant or shift within the same hue. A neutral grey series that stayed put would disappear in one theme or the other.

**`--chart-down` is byte-identical to `--destructive` in both themes.** The codebase treats them as interchangeable — `.forecast-home-table .pos` uses `--chart-up` while `.neg` uses `--destructive` rather than `--chart-down` (`index.css:2871-2874`). That is currently correct and permanently fragile: changing one without the other silently breaks the pairing with nothing to catch it.

Tailwind also exposes all 13 as `theme.extend.colors.chart.*` (`tailwind.config.js:58-76`), plus redundant `chart.line1-4` aliases duplicating `chart.1-4`. **No source file uses the Tailwind chart utilities** — charts consume the CSS variables directly.

---

## 6. The Brand surface family (10)

Complete values, consumed bare. **Defined only inside `body.brand-hub-theme-active` — see §2.1.**

| Token | Light | Dark | Uses | Role |
|---|---|---|---|---|
| `--brand-nav-bg` | `rgba(235,235,235,0.88)` | `rgba(17,17,17,0.88)` | 1 | Translucent nav bar fill |
| `--brand-panel` | `rgba(239,239,239,0.94)` | `rgba(36,36,36,0.92)` | 26 | `.card-wrapper` fill; segmented control track |
| `--brand-panel-strong` | `rgba(235,235,235,1)` | `rgba(31,31,31,1)` | 34 | `.forecast-panel` fill (opaque); table headers; tooltips |
| `--brand-line` | `rgba(17,17,17,0.14)` | `rgba(235,235,235,0.14)` | 179 | **THE DEFAULT BORDER** |
| `--brand-line-strong` | `rgba(17,17,17,0.24)` | `rgba(235,235,235,0.30)` | 18 | Input and control borders |
| `--brand-shadow` | `0 20px 46px rgba(0,0,0,0.11)` | `0 24px 56px rgba(0,0,0,0.38)` | 4 | Elevation — a shadow, not a colour |
| `--brand-input-bg` | `rgba(255,255,255,0.72)` | `rgba(46,46,46,0.82)` | 8 | `.brand-input` and table input fill |
| `--brand-positive` | `#739666` | `#85aa76` | 29 | Positive value **text** |
| `--brand-negative` | `#d95f3d` | `#e6724f` | 26 | Negative value **text** |
| `--brand-warning` | `#a97800` | `#e1bb45` | 8 | Caution state |

**`--brand-panel` vs `--brand-panel-strong`** is the one distinction worth memorising: `panel` is translucent (`0.94` / `0.92` alpha) and lets whatever is behind it show through; `panel-strong` is fully opaque. `.card-wrapper` uses the translucent one, `.forecast-panel` the opaque one — which is a large part of why the two card surfaces do not look interchangeable even though they should be.

**`--brand-positive` / `--brand-negative` are text colours, not chart colours.** They are close to but not identical with `--chart-up` / `--chart-down`, and the split is real: use the `--brand-*` pair for a signed number in a table cell, the `--chart-*` pair for a mark in a chart. See `DESIGN.md` §4.3 for why the codebase has both and which to reach for.

Only `--brand-warning` has no counterpart in any other family. There is no `--chart-warning` and no shadcn warning token; amber caution states elsewhere borrow `--primary`.

---

## 7. Component-scoped properties (6)

Not global tokens. These are set on specific selectors and read by rules nearby. They use custom-property syntax to get inheritance, not to participate in the theme.

| Property | Set on | Value | Source |
|---|---|---|---|
| `--brand-ink` | `.brand-mark` | `hsl(var(--foreground))` | `index.css:217` |
| `--brand-accent` | `.brand-mark` | `hsl(var(--primary))` | `index.css:218` |
| `--section-color` | `#forecast-*` section ids | hard-coded hex (below) | `index.css:3792-3795` |
| `--section-progress` | accordion | `50%` default | `index.css:3954` |
| `--waterfall-count` | waterfall chart markup | set inline | — |
| `--waterfall-zero` | waterfall chart markup | set inline | — |

`--section-color` carries the per-section identity colour, read via `color-mix()` by the panel and accordion patterns to tint a section code and left border. It is **hard-coded hex and bypasses the token system entirely** (12 uses):

| Section | Value | Token equivalent |
|---|---|---|
| `#forecast-base` | `#5B8DEF` | none |
| `#forecast-macro` | `#9C6ADE` | none |
| `#forecast-existing` | `#FFAC03` | **`--primary`, duplicated literally** |
| `#forecast-hiring` | `#2f9e6e` | none |

Three of the four are genuinely new colours with no token behind them. The fourth is the brand amber written out by hand. If section identity colours are ever formalised, this is where they should become tokens.

---

## 8. Phantom tokens — do not use (F-01)

Eleven custom properties are consumed via `var()` somewhere in the app and **defined in zero stylesheets**. Verified: `grep -rE '^\s*--<name>\s*:' frontend/src` returns nothing for each.

Every one of these resolves to its inline fallback, in **both** themes. The `var()` wrapper is decorative — it looks theme-aware in code review and is not.

| Phantom token | `var()` uses | Always renders |
|---|---|---|
| `--brand-red` | 9 | `#c4554d` |
| `--brand-chart-2` | 8 | `#6b7c93` |
| `--brand-moss` | 5 | `#5d8a66` |
| `--brand-gold` | 3 | `#FFAC03` |
| `--brand-danger` | 3 | `#e5241d` |
| `--font-mono` | 3 | inline stack |
| `--brand-chart-3` | 2 | `#9aa7b8` |
| `--brand-lust` | 2 | inline hex |
| `--brand-shadow-soft` | 2 | **nothing — see below** |
| `--brand-smokescreen` | 1 | inline hex |
| `--brand-ink` | 1 | inline hex |

**`--brand-shadow-soft` is a live bug, not just a naming problem.** It is written with **no fallback** at `index.css:4888` and `4927`:

```css
box-shadow: var(--brand-shadow-soft);   /* property undefined, no fallback */
```

With the property undefined this resolves to the guaranteed-invalid value and the declaration is dropped at computed-value time. **Those two elements have no shadow at all and never have.** Use `var(--brand-shadow)` instead.

### 8.1 Do not "fix" these by defining them

Defining a phantom token changes rendered output immediately and everywhere it is referenced. Defining `--brand-chart-2` would instantly re-colour six charts that currently render `#6b7c93`.

The two defensible moves, decided per token:

1. **Map it to a real token** and accept the visual change deliberately, as a reviewed diff.
2. **Delete the `var()` wrapper and keep the literal.** Uglier, but it tells the truth about what renders.

`--brand-shadow-soft` is the exception — it should be defined or its two rules deleted, because the current state is a straight defect rather than a misleading indirection.

---

## 9. Typography — no tokens exist

There are **no font custom properties.** Families are written as literal stacks at roughly 40 sites. (`--font-mono` is referenced three times and is a phantom — §8.)

Loaded via `@import` from Google Fonts:

```
https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,500&family=Inter+Tight:wght@400;500;600&family=JetBrains+Mono:wght@500
```

| Role | Stack | Where |
|---|---|---|
| Body / UI default | `'Inter Tight', system-ui, sans-serif` | Everything |
| Mono | `'JetBrains Mono', monospace` | Section codes, chips, tabular figures, table `th` |
| Display | `Fraunces`, italic 500, via `.brand-serif` | RM Pro Forma hero **only** |

**The mono treatment is a fixed formula, not a free choice:** always 10–11px, uppercase, `letter-spacing` 0.06–0.12em. Every section code, chip, and table header in the product follows it. Mono at 14px lowercase would be off-system.

**Avenir Pro is declared and absent** (F-08). `tailwind.config.js:114-118` declares it; nothing imports it and nothing references it. Dead — ignore it.

Because fonts load over the network, any offline harness falls back to system stacks. That is expected and is called out in `gallery.html`.

---

## 10. Dead palettes that are not part of FRAME

Three colour systems exist in config or code and render nowhere, or render outside the token system. They are documented so you recognise them and leave them alone — none of them is a source of truth.

**`tailwind.brand` (F-03)** — a 12-colour palette at `tailwind.config.js:91-104` (mulberry, lust, hobgoblin, caribbean-blue, pigeon, nero, smokescreen, …). Usages in `frontend/src`: **zero**. `index.css:503-601` then carries ~100 lines of shim remapping every `brand-*` utility onto semantic tokens. A dead palette, plus a shim for the dead palette, plus a stale doc describing it as current.

**`tailwind.icon` (F-02)** — `decrease: #e5241d`, `increase: #02a88e`, `neutral: #828282` at `tailwind.config.js:84-88`. Zero usages. It declares a *third* up/down semantic that contradicts `--chart-up` / `--chart-down`.

**Ad-hoc JS constants (F-02, F-04)** — `GOLD`, `GREEN`, `RED`, and friends, redeclared identically in six Forecast files:

| Constant | Value | Duplicate / conflict |
|---|---|---|
| `GOLD` | `#FFAC03` | literal copy of `--primary` |
| `GREEN` | `#2f9e6e` | conflicts with `--chart-up` |
| `RED` | `#d9534f` | conflicts with `--chart-down` / `--destructive` |
| `PURPLE` | `#9C6ADE` | matches `#forecast-macro` section colour |
| `AMBER` | `#e0a93c` | near-miss on `--primary` |
| `BLUE` | `#5b8dd6` | near-miss on `#forecast-base` |

**These are the only colours in the product that cannot respond to the theme.** They are plain hex in JS, evaluated once, identical in light and dark.

The arithmetic this produces: **five distinct greens** (`#739666`, `#85aa76`, `#02a88e`, `#2f9e6e`, `#5d8a66`) and **six distinct reds** (`#d95f3d`, `#e6724f`, `#e5241d`, `#e61d24`, `#d9534f`, `#c4554d`) live in one codebase. A designer asking "what is the app's green?" currently gets three defensible answers.

**The answer FRAME gives:** `hsl(var(--chart-up))` and `hsl(var(--chart-down))` for marks, `var(--brand-positive)` and `var(--brand-negative)` for text. Nothing else.

---

## 11. Adding a token

1. Add it to **all three blocks** in `index.css` — `:root`, `body.brand-hub-theme-active`, and the `[data-direction='B']` variant. Skipping the dark block is the common mistake and it fails silently in exactly one theme.
2. Decide its format deliberately: HSL channels if it is a colour that wants free alpha; a complete value if it is a shadow, a translucent surface, or a literal hex.
3. If it should be reachable as a Tailwind utility, wire it into `tailwind.config.js` too.
4. Mirror it into `tokens.css` and `tokens.json` in this repo, including its usage count and theme values.
5. Add a swatch to `gallery.html` — token swatches there are read live via `getComputedStyle`, so a new token is verified by rendering, not by being typed in twice.

Before adding a colour at all, check `tokens.json`. If you are reaching for a hex you are very probably duplicating something: `#FFAC03` already appears in six files and is just `--primary`.

---

## 12. Verifying FRAME against the app

```bash
# every custom property defined in index.css
grep -nE '^\s*--[a-zA-Z0-9-]+\s*:' frontend/src/index.css

# phantom check — for each var(--x) used, confirm a matching --x: exists
grep -roE 'var\(--[a-zA-Z0-9-]+' frontend/src | sed 's/.*var(//' | sort -u
```

The gallery is the other half of this check: it resolves every token live via `getComputedStyle` rather than restating values, so a token that has gone missing shows up as a blank swatch. **Note that `gallery.html` in this repo renders against a vendored snapshot of `index.css`, not the live app** — see the banner on the page and `README.md`. Refresh the snapshot before trusting it for a verification pass.

`FINDINGS.md` F-28 was found this way and could not have been found by reading the CSS: seven chip tone names, four of which render a colour other than the one they are named after.
