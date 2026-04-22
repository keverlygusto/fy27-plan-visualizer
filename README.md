# FY27 Accountant Channel Forecast Visualizer

Interactive single-page HTML dashboard for the FBOS team to present and explore the FY27 accountant channel forecast. Built for live walkthrough, fully self-contained (no external dependencies).

## File

`index.html` — deploy directly to Google Apps Script or open locally in any browser.

## Sections

1. **Overall Forecast** — Forecast metric grid (Firms, Opps, Adds, ACV, GNARR) by quarter with YoY. New ACV and GNARR ($k) rows are expandable to reveal Tier/Add-on sub-metrics (see "Expandable Sub-Metrics" below).
2. **Where Does Growth Come From?** — Waterfall bridge: FY26 → Baseline Growth → Firm Acq → Frequency → ACV/Mix → FY27.
3. **When Does It Show Up?** — Gantt chart grouped by lever (Firm Acquisition, Frequency, ACV/Mix) with Confidence, DRI, and FY27 Impact columns. Outcome metric row shows quarterly GNARR, YoY, and delta vs baseline.
4. **What Could Break?** — Risk cards (High/Medium) and Opportunity cards. Hardcoded narrative — update manually each cycle.

## Scenario View

The visualizer currently shows **Forecast only** by default. The Baseline/Forecast toggle is hidden from the UX but the underlying code and data remain intact.

- `setScenario('forecast')` is called on page load
- Toggle buttons are wrapped in `<div class="toggle-group" style="display:none">`
- `SD.baseline` and `CHILD_DATA.baseline` remain populated and current
- **To restore the toggle:** delete `style="display:none"` on the `toggle-group` div near line 884. No other changes needed.

## Expandable Sub-Metrics (Forecast view only)

Two of the seven parent rows have clickable expand carets (▸) that reveal sub-metric child rows:

| Parent | Children |
|---|---|
| New ACV | Tier ACV, Add-on ACV |
| GNARR ($k) | Tier GNARR ($k), Add-on GNARR ($k) |

Behavior:
- Child rows hidden by default; click the parent row (or press Enter on it) to toggle
- Caret rotates 90° when expanded
- Children are forecast-only — guarded in `renderSummary` by `scenario === 'forecast'`
- Expansion state resets on any re-render (scenario switch, refresh)
- Child rows are styled indented + italic + lighter background for visual hierarchy

Child row data lives in a separate `CHILD_DATA` object (not inside `SD`) to keep the parent SD structure unchanged and the diff minimal.

## Data Sources

All sections 1–3 pull from a single Google Sheets file:

**Spreadsheet:** `FY26 Accountant Forecast vs. Actual`
**ID:** `1lv4RwsOXVHgtbaYAR1AISrK0o1w_OL_T6gthMypGSfw`

| Data | Tab | Range |
|---|---|---|
| Baseline parent + child metrics (quarterly + YoY) | FY27 Scenarios | `H4:BI36` ("FY27 Baseline" block) |
| Forecast parent + child metrics (quarterly + YoY) | FY27 Scenarios | `H39:BI67` ("FY27 Financial Forecast" block) |
| Gantt initiatives (name, start, impact, confidence) | Operator Sandbox (Q4.FY26) | `AR26:AZ65` |
| Lever totals (Firm Acq $, ACV $) | Operator Sandbox (Q4.FY26) | `AW1:AX1` (Firm Acq), `AW3:AX3` (ACV) |
| Confidence band totals (High, Medium, Low) | Operator Sandbox (Q4.FY26) | `AU1:AZ5` |

### Range Structure Within Each Scenario Block

Each scenario block (baseline `H4:BI36`, forecast `H39:BI67`) contains two sub-blocks stacked vertically:

1. **Absolute values block** — row labels in col H, 36 monthly cols, 12 quarterly cols (Q1-FY25 through Q4-FY27), 3 annual cols (FY25, FY26, FY27)
2. **YoY % block** (immediately below, same column layout) — FY25 quarterly/annual columns are blank (no prior-year base); FY26 and FY27 columns are populated

### Metric Rows (in order, col H labels)

Within each scenario block:

1. Partner Program Firms
2. Opportunities
3. Opps/Firm
4. Adds
5. Adds/Firm
6. **Tier ACV** (child of New ACV)
7. **Add-On ACV** (child of New ACV)
8. New ACV
9. **Tier GNARR ($k)** (child of GNARR)
10. **Add-On GNARR ($k)** (child of GNARR)
11. GNARR ($k)

The Tier/Add-On rows for ACV were added in April 2026. Tier/Add-On GNARR rows existed previously.

## Waterfall Logic

- **FY26 Base** and **FY27 Forecast** — pulled directly from `SD` data object (`forecast.fy26.fy.gnarr` and `forecast.fy27.fy.gnarr`, ÷ 1,000 to convert $k → $M)
- **Firm Acquisition** and **ACV/Mix** — fixed values from Operator Sandbox lever totals (`AW1:AX1`, `AW3:AX3`)
- **Frequency** — plug (FY27 Forecast minus all other bars)
- **Baseline Growth** — FY27 Baseline annual GNARR minus FY26 Base

## Confidence Bands

All three tiers (High, Medium, Low) pulled directly from `AU1:AZ5` — Medium is no longer a plug.

## Gantt Data Structure

The Gantt uses `LEVER_GROUPS` (not `const data`) — this is the live data structure. Items filtered to those with Lift per Month > 0 and FY27 GNARR Impact > 0. Sorted by start month within each lever group.

Start month index: `0=May'26, 1=Jun, 2=Jul, 3=Aug, 4=Sep, 5=Oct, 6=Nov, 7=Dec, 8=Jan'27, 9=Feb, 10=Mar, 11=Apr`

## Refreshing Data

Follow this exact sequence to avoid drift between code and sheet.

### 1. Authenticate Google Sheets MCP
```
mcp__gsheetsgusto__authenticate
```

### 2. Fetch FY27 Scenarios tab
```
mcp__gsheetsgusto__fetch
  spreadsheet_id: 1lv4RwsOXVHgtbaYAR1AISrK0o1w_OL_T6gthMypGSfw
  range: FY27 Scenarios!H1:BJ80
```

This single fetch returns both baseline and forecast blocks. Extract quarterly + annual values from the right-hand summary columns for each metric row.

### 3. Update `const SD` (parent metrics)

For each scenario (`baseline`, `forecast`) × each period group (`fy26`, `fy26yoy`, `fy27`, `fy27yoy`) × each period (`q1`, `q2`, `q3`, `q4`, `fy`), write the values for these 7 parent keys:

`firms`, `opps`, `oppsPerFirm`, `adds`, `addsPerFirm`, `acv`, `gnarr`

### 4. Update `const CHILD_DATA` (sub-metrics)

Same nesting as `SD`, but with these 4 child keys:

`acvTier`, `acvAddon`, `gnarrTier`, `gnarrAddon`

Both baseline and forecast blocks are stored even though only forecast is displayed — keeps structure parallel with the sheet and allows re-enabling baseline display later without a refresh.

### 5. Format conventions

- **Integer values** (firms, opps, adds) — stored as raw numbers
- **Decimals** (oppsPerFirm, addsPerFirm) — two decimals (e.g., `0.58`)
- **Dollar values** (acv, acvTier, acvAddon) — stored as integers (the `sFmt` helper prepends `$` and adds thousands separators at render)
- **GNARR values** (gnarr, gnarrTier, gnarrAddon) — stored in $k (thousands); the `gnarr` format divides by 1,000 and renders as `$X.Xm`
- **YoY %** — stored as strings with explicit `+` or `-` prefix (e.g., `'+9%'`, `'-8%'`, `'+0%'`). Round to nearest whole percentage point for display consistency (the sheet sometimes shows fractional values like `2.8%` → store as `'+3%'`).

### 6. Update Operator Sandbox data (Gantt + Waterfall levers)
```
mcp__gsheetsgusto__fetch
  range: Operator Sandbox (Q4.FY26)!AR26:AZ65
```
Update `LEVER_GROUPS`, then fetch `AU1:AZ5` for confidence bands, `AW1:AX3` for lever totals.

### 7. Update `OUTCOME` array and full-year outcome row

Quarterly GNARR values + YoY % + delta vs baseline — from the same `H39:BI67` forecast range, GNARR rows.

### 8. Verify + redeploy

- Open `index.html` locally, sanity-check all four sections render without errors
- Copy full HTML to clipboard: `pbcopy < ~/claudecode/fy27-plan-visualizer/index.html`
- Paste into Apps Script editor → save → redeploy (use existing deployment, don't create new)

## Deployment

Hosted via Google Apps Script:
- **Live dashboard:** https://script.google.com/a/macros/gusto.com/s/AKfycbxmokOWBJJEb9MwFWw-dRzRA8c3N4eUjfg_GMfRWXviX0lXX-bo5MWGkphJH4qnrI6UiQ/exec
- **Apps Script editor:** https://script.google.com/home/projects/16fmP0s7d0vdND4N1IoNaU51qgFYsyY9vjvCDcbEm7s60_zE5i82sxYuS/edit
- In `Code.gs`: `function doGet() { return HtmlService.createHtmlOutputFromFile('index').setTitle('FY27 Accountant Channel Forecast'); }`
- Deploy as Web App → Execute as Me → Access: Anyone at Gusto
- **Important:** redeploying overwrites the existing `index.html` file in Apps Script. Use the same deployment ID — do not create a new deployment, or the shared URL will break.

## Last Updated

April 2026:
- Added expandable Tier ACV / Add-on ACV / Tier GNARR / Add-on GNARR child rows under New ACV and GNARR parent rows (forecast view only)
- Source sheet ranges shifted to `H4:BI36` (baseline) and `H39:BI67` (forecast) after inserting the Tier ACV / Add-On ACV rows
- Refreshed all baseline FY27 parent values against current sheet state (prior values were stale)
- Corrected several Q4-FY26 YoY values that were off by 1pp
- Default view changed from Baseline → Forecast; scenario toggle hidden from UX
