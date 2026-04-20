# FY27 Accountant Channel Forecast Visualizer

Interactive single-page HTML dashboard for the FBOS team to present and explore the FY27 accountant channel forecast. Built for live walkthrough, fully self-contained (no external dependencies).

## File

`index.html` — deploy directly to Google Apps Script or open locally in any browser.

## Sections

1. **Overall Forecast** — Baseline vs. Current Forecast metric grid (Firms, Opps, Adds, ACV, GNARR) by quarter. Toggle between scenarios.
2. **Where Does Growth Come From?** — Waterfall bridge: FY26 → Baseline Growth → Firm Acq → Frequency → ACV/Mix → FY27.
3. **When Does It Show Up?** — Gantt chart grouped by lever (Firm Acquisition, Frequency, ACV/Mix) with Confidence, DRI, and FY27 Impact columns. Outcome metric row shows quarterly GNARR, YoY, and delta vs baseline.
4. **What Could Break?** — Risk cards (High/Medium) and Opportunity cards. Hardcoded narrative — update manually each cycle.

## Data Sources

All sections 1–3 pull from a single Google Sheets file:

**Spreadsheet:** `FY26 Accountant Forecast vs. Actual`
**ID:** `1lv4RwsOXVHgtbaYAR1AISrK0o1w_OL_T6gthMypGSfw`

| Data | Tab | Range |
|---|---|---|
| Baseline metrics (quarterly) | FY27 Scenarios | Baseline block |
| Forecast metrics (quarterly) | FY27 Scenarios | Forecast block |
| Gantt initiatives (name, start, impact, confidence) | Operator Sandbox (Q4.FY26) | AR26:AZ65 |
| Lever totals (Firm Acq, Freq, ACV) | Operator Sandbox (Q4.FY26) | AZ column totals |
| Confidence band totals | Operator Sandbox (Q4.FY26) | AU1:AZ5 |

## Waterfall Logic

- **FY26 Base** and **FY27 Forecast** — pulled directly from SD data object
- **Firm Acquisition** and **ACV** — fixed values from Operator Sandbox lever totals
- **Frequency** — plug (FY27 Forecast minus all other bars)
- **Baseline Growth** — FY27 Baseline minus FY26 Base

## Confidence Bands

All three tiers (High, Medium, Low) pulled directly from `AU1:AZ5` — Medium is no longer a plug.

## Gantt Data Structure

The Gantt uses `LEVER_GROUPS` (not `const data`) — this is the live data structure. Items filtered to those with Lift per Month > 0 and FY27 GNARR Impact > 0. Sorted by start month within each lever group.

Start month index: `0=May'26, 1=Jun, 2=Jul, 3=Aug, 4=Sep, 5=Oct, 6=Nov, 7=Dec, 8=Jan'27, 9=Feb, 10=Mar, 11=Apr`

## Refreshing Data

1. Authenticate Google Sheets MCP (`mcp__gsheetsgusto__authenticate`)
2. Fetch `FY27 Scenarios` tab for SD object (baseline + forecast quarterly metrics)
3. Fetch `Operator Sandbox (Q4.FY26)!AR26:AZ65` for LEVER_GROUPS (initiatives)
4. Fetch `Operator Sandbox (Q4.FY26)!AU1:AZ5` for confidence bands
5. Update `const SD`, `LEVER_GROUPS`, `OUTCOME` array, full-year outcome row, and confidence constants in `index.html`

## Deployment

Hosted via Google Apps Script:
- In `Code.gs`: `function doGet() { return HtmlService.createHtmlOutputFromFile('index').setTitle('FY27 Accountant Channel Forecast'); }`
- Deploy as Web App → Execute as Me → Access: Anyone at Gusto

## Last Updated

April 2026 — data refreshed from Operator Sandbox (Q4.FY26) and FY27 Scenarios tabs.
