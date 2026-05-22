# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Industry Command Center is a self-contained, single-file SPA (`index.html`, ~3,200 lines) for exploring AI industry metrics, company intelligence, investment trends, and forecasts. It has zero external dependencies — no npm, no build step, no framework.

## Running the App

Open `index.html` directly in any modern browser. No server or build process is required.

```bash
# macOS
open index.html

# Linux
xdg-open index.html
```

There are no linting, testing, or build commands.

## Architecture

Everything lives in `index.html` in three co-located layers:

### 1. Data Layer (hardcoded JS constants)
All datasets are embedded as JavaScript constants near the top of the `<script>` block:
- `INV_DATA` — annual investment totals (2019–2025) split by Total, GenAI, and China
- `SEG_DATA` — sector verticals with market share percentages and CAGRs
- `TRENDS_DATA` — monthly ARR trajectory arrays for Anthropic, OpenAI, and xAI
- `COMPANIES` — company cards with ARR, growth, and stage metadata
- Ticker KPI values (market size, enterprise adoption, forecast) defined as inline literals inside `initInteractivity()`

When updating data, change only these constants — the chart renderers consume them directly.

### 2. Chart Rendering Layer (Canvas API)
Custom chart renderers — no Chart.js, D3, or other libraries:
- `barCI(canvasId, data, opts)` — horizontal/vertical bar charts
- `lineCI(canvasId, data, opts)` — line/area time-series charts
- `donutCI(canvasId, data, opts)` — donut/pie charts
- `renderInvChart()`, `renderSegChart()`, `renderTrendChart()` — section-specific wrappers that pull from the data constants and call the primitives above
- `attachTip(canvas, ...)` — registers `mousemove` listeners to show floating tooltips

`drawAll(sectionId)` is the single entry point called whenever a section becomes visible; it dispatches to the appropriate render functions.

### 3. Navigation / UI Layer
- `show(sectionId, navEl)` — hides all `.section` divs, shows the target, updates the active nav state, then calls `drawAll(sectionId)`
- Sections are plain `<div id="section-name" class="section">` elements
- Mobile navigation is a slide-in overlay toggled by `toggleMobileNav()`
- The stakeholder simulator (`runSimulation()`) scores user choices across CEO/Investor/Product/Sales/Risk roles and renders a result breakdown

### Section IDs (navigation targets)
`overview`, `company`, `investment`, `sector`, `trends`, `simulator`, `pipeline`, `schema`, `about`, `guide`, `summary`, `download`

## Key Conventions

- **Single-file discipline**: keep everything in `index.html`. Do not introduce separate `.js` or `.css` files.
- **No external dependencies**: do not add CDN `<script>` tags or `import` statements. All rendering uses the browser's native Canvas API and vanilla DOM APIs.
- **CSS custom properties** drive the color scheme; always use the existing variables (e.g., `--accent`, `--accent2`, `--success`, `--bg`, `--surface`) rather than hardcoded hex values.
- **Canvas redraws**: every chart renderer clears and redraws from scratch on each call — there is no virtual DOM or diff. When resizing or toggling views, call the relevant render function again.
- **Data updates**: change the JS constants in the data layer only; never duplicate data into HTML attributes or CSS.
- **Mobile breakpoints**: 1100px (tablet sidebar collapse) and 768px (mobile overlay nav). Test layout changes at both breakpoints.
