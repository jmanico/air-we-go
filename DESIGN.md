# Air We Go — Design System

**Breathe Better. Live Smarter.**

This document defines the brand and visual design system for **Air We Go**, the
global air-quality platform described in [`REQUIREMENTS.md`](./REQUIREMENTS.md).
It is the single source of truth for the logo, color, typography, iconography,
data-visualization, and accessibility rules used across the responsive website,
the iOS/Android apps, and any administrative or marketing surfaces.

> Design decisions here are constrained by the product requirements — most
> notably **WCAG 2.2 Level AA** (NFR-A11Y-001), **"air-quality categories must
> not be represented by color alone"** (NFR-A11Y-002, AQ-004), and the mandate
> to **identify the air-quality standard for every displayed index** (AQ-001).
> Treat those as non-negotiable design inputs, not suggestions.

---

## 1. Brand Foundation

### 1.1 Essence

Air We Go makes invisible air-quality data feel **global, trustworthy, and
immediately usable**. The brand pairs the optimism of clean air (green, leaf,
lightness) with the authority of earth science and data (blue, globe, grid).

| Attribute | We are | We are not |
|-----------|--------|------------|
| Tone | Clear, calm, factual | Alarmist, preachy |
| Feel | Fresh, open, breathable | Cluttered, heavy |
| Voice | Plain-language, science-backed | Jargon-y, absolutist |
| Trust | Sourced, standard-labeled | Vague, "just trust us" |

### 1.2 Tagline

**Breathe Better. Live Smarter.**

Use with a period after each clause. Title Case. Do not translate literally —
localize for meaning (NFR-I18N-003).

---

## 2. Logo

The logo is a **globe** wrapped by three **air currents**, with a **location
pin** carrying a **leaf** — location + air + planet + green, in one mark.

### 2.1 Assets

All logo files live in [`assets/logo/`](./assets/logo/). SVG is the source of
truth; PNGs are convenience rasters.

| File | Use |
|------|-----|
| `air-we-go-logo.svg` | **Primary** lockup: mark + wordmark + tagline. Marketing, README, splash. |
| `air-we-go-horizontal.svg` | Mark + inline wordmark, no tagline. App headers, nav bars, email. |
| `air-we-go-mark.svg` | Mark only. Avatars, watermarks, tight spaces where the wordmark won't fit. |
| `air-we-go-icon.svg` | Mark on a rounded tile. App icon, favicon, PWA install. |

### 2.2 Clear space & minimum size

- **Clear space:** keep padding equal to the height of the globe on all sides of
  the mark. Nothing (text, edges, other logos) intrudes into that zone.
- **Minimum size:** mark ≥ 24 px tall (digital); primary lockup ≥ 120 px wide.
  Below the primary lockup's minimum, switch to the horizontal or mark-only asset.

### 2.3 Placement on backgrounds

- Prefer the logo on white, very light neutral, or the light sky-to-mint gradient
  (§3.4). 
- On dark or photographic backgrounds, use a knockout: keep the full-color mark
  but set the wordmark to white (`--awg-neutral-000`). Do not place the
  gradient wordmark on a dark field — it fails contrast.
- Maintain the globe's internal contrast; never recolor the globe to a single flat hue.

### 2.4 Misuse — do not

- Stretch, skew, rotate, or re-proportion the mark and wordmark independently.
- Recolor to off-brand hues, add drop shadows/bevels/outlines, or apply gradients
  other than the defined brand gradients.
- Swap the typeface, re-letter the wordmark, or change word breaks.
- Place on low-contrast or busy backgrounds that reduce legibility below AA.
- Reconstruct the mark from parts — always use the provided SVGs.

---

## 3. Color

Air We Go color is split into three layers: **brand** (identity), **UI**
(product surfaces), and **air-quality** (data semantics). Keep them separate —
brand blue is not a status color, and an AQI color is never decorative.

### 3.1 Brand palette

| Token | Hex | Role |
|-------|-----|------|
| `--awg-blue-700` | `#0B4DA2` | Primary brand blue — "Air/We" wordmark base, headings, primary actions |
| `--awg-blue-500` | `#1C7FB8` | Mid blue — gradients, links, focus |
| `--awg-green-600`| `#3E9E4F` | Deep leaf green — "Go" wordmark base, success |
| `--awg-green-500`| `#5FBB46` | Brand green — accents, positive emphasis |
| `--awg-green-400`| `#7BC143` | Bright leaf — gradient highlight |
| `--awg-teal-500` | `#1FA7C4` | Ocean/data accent — sparingly, for charts and highlights |

**Brand gradients** (identity only — do not use as UI fills for text
backgrounds):

- **Wordmark "Air/We":** `#1C7FB8 → #0B4DA2` (vertical)
- **Wordmark "Go" / leaf:** `#7BC143 → #3E9E4F` (vertical / diagonal)
- **Globe:** `#7BC143 → #3E9E4F → #1C7FB8 → #0B4DA2` (top-to-bottom)

### 3.2 Neutrals

| Token | Hex | Role |
|-------|-----|------|
| `--awg-neutral-000` | `#FFFFFF` | Surface / knockout text |
| `--awg-neutral-050` | `#F5F8FB` | App background (light) |
| `--awg-neutral-100` | `#E7EEF4` | Cards, dividers |
| `--awg-neutral-400` | `#8A99A8` | Muted text, disabled |
| `--awg-neutral-700` | `#33475B` | Body text |
| `--awg-neutral-900` | `#0E1B2A` | Max-contrast text, dark-mode surface |

### 3.3 Semantic UI colors

| Purpose | Token | Hex |
|---------|-------|-----|
| Info / link | `--awg-blue-500` | `#1C7FB8` |
| Success | `--awg-green-600` | `#3E9E4F` |
| Warning | `--awg-amber-500` | `#E0A400` |
| Danger / error | `--awg-red-600` | `#C0392B` |
| Focus ring | `--awg-focus` | `#1C7FB8` (3 px, 2 px offset) |

> These are **product-state** colors. They are intentionally distinct from the
> air-quality category colors in §3.5 so that "the app has an error" never reads
> as "the air is unhealthy."

### 3.4 Ambient background

The signature background is a soft sky-to-mint gradient used behind hero and
empty states: `#EAF6FF → #FFFFFF → #EAF7E4`. Keep it subtle — never let it
reduce text contrast below AA.

### 3.5 Air-quality category colors (US EPA AQI reference set)

These colors carry **meaning** and are governed by requirement **AQ-004**
(configurable labels/thresholds) and **AQ-001** (the standard must be named on
screen). The set below is the **US EPA AQI** scale; other standards (European
AQI, provider-specific) define their own bands and **must not be silently
converted** (AQ-002). When a different standard is active, load its band colors
and labels from configuration rather than hard-coding this table.

| AQI band | Category | Color | Hex | On-color text |
|----------|----------|-------|-----|---------------|
| 0–50 | Good | Green | `#00A65A` | white |
| 51–100 | Moderate | Yellow | `#E0C000` | `#0E1B2A` |
| 101–150 | Unhealthy for Sensitive Groups | Orange | `#F07D02` | `#0E1B2A` |
| 151–200 | Unhealthy | Red | `#D6202B` | white |
| 201–300 | Very Unhealthy | Purple | `#8332A8` | white |
| 301+ | Hazardous | Maroon | `#7E0F26` | white |

> **Never rely on these colors alone (NFR-A11Y-002).** Every air-quality value
> must *also* carry the numeric index, the category label, and — where space
> allows — a shape/icon (see §5.2) and a text description. On maps this means
> markers combine color **with** a label, symbol, or pattern (requirement §
> "map markers"), and the map always has a non-map tabular alternative
> (NFR-A11Y-004).

---

## 4. Typography

| Role | Typeface | Weight | Notes |
|------|----------|--------|-------|
| Brand wordmark | **Poppins** | 800 (ExtraBold) | Geometric, rounded — matches the mark. Fallback: Montserrat. |
| Headings (UI) | **Poppins** | 600–700 | Tight tracking on large sizes. |
| Body / UI | **Inter** | 400–600 | Excellent screen legibility, wide language coverage. |
| Data / numerals | **Inter** (tabular figures) | 500–700 | Use `font-variant-numeric: tabular-nums` for readings and charts. |
| Mono (admin/logs) | **JetBrains Mono** / system mono | 400 | Diagnostics, IDs, raw provider payloads. |

Fallback stack: `Poppins, Montserrat, "Segoe UI", system-ui, Arial, sans-serif`
for display; `Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial,
sans-serif` for text.

**Type scale (rem, 1rem = 16px):** 3.0 / 2.25 / 1.75 / 1.375 / 1.125 / 1.0 /
0.875 / 0.75. Body default 1.0rem, line-height 1.5.

**Rules**

- Respect user font-size / dynamic type; never disable zoom (NFR-A11Y-005).
- Body text meets **≥ 4.5:1** contrast; large text (≥ 24px or 19px bold) meets
  **≥ 3:1** (WCAG 2.2 AA). Verify AQI on-color pairings in §3.5.
- Externalize all UI strings for localization (NFR-I18N-001); allow for text
  expansion (German/Finnish can be +35%). Support RTL mirroring of layout and
  the horizontal logo lockup.

---

## 5. Iconography & Data Visualization

### 5.1 Icon style

- Line icons, ~2px stroke, rounded caps/joins, on a 24px grid — consistent with
  the wind-current line weight in the logo.
- Reuse brand motifs: **pin** for location/search, **leaf** for clean-air /
  eco, **globe** for global/coverage, **waves** for air/wind, **gear** for admin.

### 5.2 Air-quality visual encoding (redundant by design)

To satisfy "not color alone," each AQI category has a **shape token** used
alongside color and the numeric value:

| Category | Shape token | Example use |
|----------|-------------|-------------|
| Good | ● circle | map marker, legend |
| Moderate | ◆ diamond | |
| Unhealthy for Sensitive Groups | ▲ triangle | |
| Unhealthy | ■ square | |
| Very Unhealthy | ✚ plus | |
| Hazardous | ✖ cross | |

Pair color + shape + number + label everywhere a category appears.

### 5.3 Charts (historical & trends — §4 Historical, NFR-A11Y-003)

- Support line, bar, and heat-map forms as required. Default to line for trends,
  bar for period comparisons, heat map for calendar/hour density.
- **Distinguish missing data from a valid zero** (hard requirement): render gaps
  as broken/greyed segments with a distinct "no data" legend entry — never draw
  a 0 where a reading is absent.
- Every chart ships an **accessible equivalent**: ARIA labels plus a toggle to a
  data table (NFR-A11Y-003). Provide `aria-label`s on series and summary stats
  (average, min, max, Δ vs previous period per §4 Trends).
- Use tabular numerals, localized number/date/unit formatting (NFR-I18N-002),
  and always show the **units and the index standard** (AQ-001, AQ-003).
- Do not encode a series by hue alone — add direct labels, patterns, or markers.

### 5.4 Maps

- Markers/regions combine category **color + shape + label**; clustering shows a
  representative category with a count.
- Always provide the **non-map alternative** (ranked/searchable list or table)
  for the same data (NFR-A11Y-004).
- Name the active index standard and attribution/licensing on or near the map
  (AQ-001, data-provider attribution requirements).

---

## 6. Accessibility Checklist (WCAG 2.2 AA)

Design is not "done" until:

- [ ] Text contrast ≥ 4.5:1 (body) / 3:1 (large); UI components & focus ≥ 3:1.
- [ ] No information conveyed by **color alone** — AQI always has number + label + shape/text.
- [ ] Visible, non-color focus indicator on every interactive element.
- [ ] Full keyboard operability; logical focus order; target size ≥ 24×24px.
- [ ] Charts have text/table equivalents; maps have a non-map alternative.
- [ ] Motion respects `prefers-reduced-motion`; no essential info in animation only.
- [ ] Supports OS text scaling / dynamic type and screen readers (NFR-A11Y-005).
- [ ] Every displayed index states its **standard** and **units**.

---

## 7. Design Tokens (starter)

```css
:root {
  /* Brand */
  --awg-blue-700:#0B4DA2; --awg-blue-500:#1C7FB8;
  --awg-green-600:#3E9E4F; --awg-green-500:#5FBB46; --awg-green-400:#7BC143;
  --awg-teal-500:#1FA7C4;

  /* Neutrals */
  --awg-neutral-000:#FFFFFF; --awg-neutral-050:#F5F8FB; --awg-neutral-100:#E7EEF4;
  --awg-neutral-400:#8A99A8; --awg-neutral-700:#33475B; --awg-neutral-900:#0E1B2A;

  /* Semantic UI */
  --awg-info:#1C7FB8; --awg-success:#3E9E4F; --awg-warning:#E0A400;
  --awg-danger:#C0392B; --awg-focus:#1C7FB8;

  /* Air-quality (US EPA AQI reference — load per active standard) */
  --aqi-good:#00A65A; --aqi-moderate:#E0C000; --aqi-usg:#F07D02;
  --aqi-unhealthy:#D6202B; --aqi-very-unhealthy:#8332A8; --aqi-hazardous:#7E0F26;

  /* Type */
  --awg-font-display:"Poppins","Montserrat","Segoe UI",system-ui,Arial,sans-serif;
  --awg-font-body:"Inter",system-ui,-apple-system,"Segoe UI",Roboto,Arial,sans-serif;
}
```

---

## 8. Governance

- The SVGs in `assets/logo/` are canonical. Regenerate PNGs from them; do not
  hand-edit rasters.
- AQI colors/labels/thresholds are **configuration**, versioned and test-covered
  per requirement (business rules for categorization must be versioned and
  tested). This document's §3.5 is the US EPA reference set, not a hard-coded
  contract.
- Changes to brand color, logo, or the accessibility rules in §6 require review;
  open a PR against this file.

*Design system v1.0 — derived from the Air We Go brand artifact and
[`REQUIREMENTS.md`](./REQUIREMENTS.md).*
