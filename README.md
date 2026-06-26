# 🏠 Real Estate Analytics Dashboard

> A single-file, zero-dependency CRM dashboard for the Australian property market — built with vanilla HTML, CSS, and JavaScript.

![Dashboard Preview](https://img.shields.io/badge/status-active-brightgreen) ![HTML](https://img.shields.io/badge/HTML-5-orange) ![CSS](https://img.shields.io/badge/CSS-3-blue) ![JavaScript](https://img.shields.io/badge/JavaScript-ES6-yellow) ![Chart.js](https://img.shields.io/badge/Chart.js-4.4.1-pink)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Pages & Sections](#pages--sections)
- [Data Model](#data-model)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Customisation](#customisation)
- [Printing & Export](#printing--export)

---

## Overview

This dashboard is a **single HTML file** that delivers a full-featured real estate CRM analytics suite — no backend, no database, no build step required. All data lives in JavaScript objects in memory and is filtered, computed, and rendered entirely in the browser.

It targets the **Australian property market for FY2024–25**, covering national KPIs, state-level breakdowns, rental yields, suburb growth leaderboards, and 12-month price forecasts.

---

## Features

| Feature | Description |
|---|---|
| **6 KPI cards** | Live-updating listings, median price, days on market, sales volume, clearance rate, new developments |
| **State filtering** | NSW, VIC, QLD, WA — all KPIs and charts react instantly |
| **Property type filter** | Residential, Commercial, Investment — applies weighted multipliers to every figure |
| **5 dashboard pages** | Executive summary, trends, state comparison, rental analysis, suburb report |
| **5 Chart.js charts** | Bar, horizontal bar, pie, line (price index), line (forecast scenarios) |
| **Suburb leaderboard** | 12 tracked suburbs with mini trend sparkbars, filterable by state and type |
| **Rental yield table** | Estimated weekly rent derived from median price × gross yield ÷ 52 |
| **12-month forecast** | Bull / Base / Bear scenario lines |
| **Print / export** | Full multi-page print layout with light-mode colour swap via `@media print` |
| **Settings panel** | Toggle analysis text, compact KPI layout, default region on load |
| **Zero dependencies** | One CDN script (Chart.js). No frameworks, no bundler, no server |

---

## Project Structure

```
real-estate-dashboard/
│
└── index.html          ← The entire application (HTML + CSS + JS in one file)
```

Everything is intentionally kept in one file for maximum portability — open it in any browser and it works.

### Inside `index.html`

```
index.html
├── <link>              Google Fonts (Inter + Roboto Mono)
├── <style>             All CSS — palette variables, layout, components
├── <div class="db">    Application shell
│   ├── .topbar         Logo, title, state filter tags
│   ├── .sidebar        Icon navigation (5 pages)
│   ├── .main           Page router container
│   │   ├── #page-dashboard     Executive KPIs + 3 charts
│   │   ├── #page-trends        Price index + forecast + momentum
│   │   ├── #page-map           State comparison grid
│   │   ├── #page-rental        Yield chart + rent table
│   │   ├── #page-reports       Suburb leaderboard + written summary
│   │   └── #page-settings      Display preferences
│   └── .footer         Branding / timestamp
├── <script src="...">  Chart.js 4.4.1 from cdnjs
└── <script>            All application logic (IIFE)
```

---

## How It Works

### Three languages, one file

| Layer | Role | How it connects |
|---|---|---|
| **HTML** | Structure — canvas elements, buttons, tables | Provides `id` and `class` attributes |
| **CSS** | Appearance — dark theme, layout, responsive grid | Reacts to class changes (`.active`, `.on`) |
| **JavaScript** | Logic — data, filtering, chart rendering | Reads `id`s, toggles classes, writes `innerHTML` |

### Data flow on a filter click

```
User clicks "NSW"
      │
      ▼
setState('nsw')
      │
      ▼
filters.state = 'nsw'
      │
      ▼
renderAll()
  ├── renderKPIs()         → BASE values × STATE_RATIO['nsw'] × TYPE_RATIO[...]
  ├── updateMonthlyChart() → Multiplies BASE.monthly array by combined ratio
  ├── updateMedianHighlight() → Highlights NSW bar in chart c2
  ├── updatePriceIndexHighlight() → Bolds NSW-relevant line in chart c4
  ├── updateYieldHighlight() → Highlights Perth in chart c5
  ├── renderRentTable()    → Recomputes weekly rent estimates
  ├── renderMomentum()     → Highlights NSW in suburb momentum panel
  ├── renderMapCards()     → Marks NSW card as active
  ├── renderSuburbTable()  → Filters suburb rows to NSW
  ├── renderReportParagraph() → Updates written summary text
  ├── renderSegmentSnapshot() → Updates segment metrics table
  └── syncFilterUI()       → Updates active button/tag styling
```

### No database — data stored as JS objects

```javascript
// National baseline figures
const BASE = { listings: 48312, median: 892, dom: 28, ... };

// State multipliers (ratio vs. national baseline)
const STATE_RATIO = {
  nsw: { listings: 14977/48312, median: 1180/892, ... },
  vic: { listings: 13044/48312, median: 890/892, ... },
  ...
};

// Applied together at render time:
kpi.median = BASE.median × STATE_RATIO[state].median × TYPE_RATIO[type].median
```

This means all figures stay internally consistent across every filter combination.

---

## Pages & Sections

### 1. Executive Dashboard (`#page-dashboard`)
- Six live KPI cards
- Monthly sales volume bar chart (July–June)
- Median price by state horizontal bar chart
- Property type mix pie chart (House / Unit / Townhouse / Rural)
- Top 3 growth suburbs quick highlights

### 2. Market Trends (`#page-trends`)
- 5-year national price index (Base 100 = Jul 2020) — National, Sydney, Melbourne
- 12-month forecast scenarios — Bull (+9%), Base (+5%), Bear (+1%)
- Suburb momentum panel — average YoY growth by state

### 3. State Comparison (`#page-map`)
- Clickable state/territory cards (NSW, VIC, QLD, WA + national)
- Clicking a card filters the entire dashboard
- Shows median, listings, clearance rate, days on market per region

### 4. Rental Market (`#page-rental`)
- Gross rental yield bar chart by city (%)
- Estimated weekly rent table (median price × yield ÷ 52)
- Covers Darwin, Perth, Adelaide, Brisbane, Hobart, Canberra, Melbourne, Sydney

### 5. Suburb Report (`#page-reports`)
- 12 tracked suburbs with state, category, median, YoY delta, mini sparkbar
- Filterable by state tab (All / NSW / VIC / QLD / WA)
- Auto-generated written summary paragraph
- Segment snapshot table for current filter combination
- Print / export button

---

## Data Model

### Key constants

| Constant | Purpose |
|---|---|
| `BASE` | National baseline KPI figures (all states, all types) |
| `STATE_RATIO` | Per-state multipliers relative to `BASE` |
| `STATE_ABS` | Absolute figures for the Map page display |
| `TYPE_RATIO` | Per-property-type multipliers relative to `BASE` |
| `MEDIAN_DELTA` | YoY % growth figures per state |
| `CLEARANCE_DELTA` | Clearance rate point change per state |
| `CITY_MEDIAN` | Median price by city (for rental calculations) |
| `suburbs` | Array of 12 suburb objects with growth data |

### Filter state

```javascript
let filters = { state: 'all', type: 'all' };
```

Everything is derived from this single object at render time. No state management library needed.

---

## Tech Stack

| Tool | Version | Purpose |
|---|---|---|
| HTML5 | — | Structure and canvas containers |
| CSS3 | — | Layout (grid/flexbox), variables, print styles |
| Vanilla JavaScript | ES6+ | Data model, filtering, DOM manipulation |
| [Chart.js](https://www.chartjs.org/) | 4.4.1 | Bar, pie, and line chart rendering |
| [Google Fonts](https://fonts.google.com/) | — | Inter (UI) + Roboto Mono (numbers) |

No build tools, no npm, no frameworks.

---

## Getting Started

### Option 1 — Open directly
```bash
# Clone the repo
git clone https://github.com/your-username/real-estate-dashboard.git

# Open in your browser — no server needed
open index.html
```

### Option 2 — Serve locally (optional, for development)
```bash
# Python
python -m http.server 8080

# Node.js
npx serve .
```

Then visit `http://localhost:8080`.

> **Note:** The dashboard requires an internet connection on first load to fetch Chart.js and Google Fonts from their CDNs. After that, only the font may re-request on reload.

---

## Customisation

### Changing the colour palette
Edit the CSS variables at the top of the `<style>` block:
```css
:root {
  --db-bg: #0B0E11;      /* Page background */
  --db-panel: #181A20;   /* Sidebar / topbar */
  --db-card: #1E2329;    /* Panel cards */
  --db-teal: #F0B90B;    /* Primary accent (gold) */
  --db-green: #0ECB81;   /* Positive / up */
  --db-red: #F6465D;     /* Negative / down */
}
```

### Replacing sample data with real data
The `BASE`, `STATE_RATIO`, `STATE_ABS`, and `suburbs` constants at the top of the `<script>` block are the only things that need updating. All rendering functions read from those objects — swap in real figures and everything updates automatically.

### Adding a new state
1. Add an entry to `STATE_RATIO` and `STATE_ABS`
2. Add a tag in the topbar HTML: `<span class="tag clickable" data-state="sa">SA</span>`
3. Add an event listener (or extend the existing `querySelectorAll` loop)

---

## Printing & Export

Click **Print / export full report** on the Reports page. This:

1. Initialises any lazy-loaded charts (Trends, Rental)
2. Adds `.printing-all` to the root element, making all pages visible
3. Swaps to a light-mode palette via `@media print` CSS
4. Calls `window.print()` — use your browser's "Save as PDF" option to export

After printing, `.printing-all` is removed and the dark theme is restored.

---

## Data Notice

> All figures in this dashboard are **illustrative sample data** for demonstration purposes only. They are not live market data. State and property-type filters apply modelled weighting factors to a FY2024–25 national baseline to keep all KPIs, charts, and analysis sentences internally consistent.

---

## License

MIT — feel free to use, adapt, and build on this for your own projects.
