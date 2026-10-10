# MetroPulse

**Turns scattered daily Metro, BRTS and AMTS data into a two-minute management brief that flags problems and suggests where to look.**

Built for DevFest Gandhinagar Metro Hacks, **PS-13: "The Two-Minute Morning Brief."**

> **Data notice:** All passenger counts and incident records are **fictional demo data**. They are not verified Ahmedabad transport statistics.

---

## The problem

Every morning, transport managers need to know how yesterday's ridership compared with the day before and what needs attention. The data is scattered across Metro, BRTS and AMTS reports and incident notes. Someone has to gather it, calculate the changes, spot anything unusual and write a summary by hand. That is slow, inconsistent, and easy to get wrong.

## Who it is for

- **Operations managers / control-room teams:** see the morning picture in two minutes.
- **Senior officials and transport authorities:** receive a consistent, ready-to-read daily brief.
- **Analysts and data teams:** upload their own data and see exactly how each alert is calculated.

---

## Quick start

**Requirements:** Node.js 18 or newer, and npm.

```sh
npm install
npm start
```

Open <http://localhost:3000>.

`npm install` is needed once, while online, to get Express. After that the app makes **no external requests** and works fully offline.

Run the automated checks:

```sh
npm test
```

---

## What works right now (try it)

| Page | What you can do |
|---|---|
| **Overview** | See today vs yesterday totals (45,200 vs 47,000, −3.83%), reported incidents, and systems to review. View the Metro / BRTS / AMTS comparison chart. Filter the system table. |
| **Alerts & Anomalies** | See automatic alerts for any system that changes by 10% or more. In the demo, Metro (−10.71%) is flagged **High Priority** with a recommended investigation. |
| **Transport Data** | Browse station and route counts and the incident register. **Upload your own JSON**, validated with clear error messages. Load or reset the demo dataset. |
| **Daily Brief** | Generate a one-page brief from the current data. **Download TXT** or **Print / Save PDF**. |
| **About** | Problem statement, method, caveats and demo flow. |

Other behavior:

- Incidents are shown as **possible leads, never proven causes.**
- A missing or zero previous-day count shows **N/A** with a data-quality warning instead of crashing.
- Charts are drawn with Canvas, so no external chart or CSS libraries are needed.

### Quick test for judges

Edit `data/demo.json` (for example, set BRTS `today` to `8000`) or upload a modified file. The totals, chart, alerts and brief all update. This shows the results are calculated, not hard-coded.

---

## Two-minute demo flow

1. **Overview:** show 45,200 today vs 47,000 yesterday and the comparison chart.
2. Point out Metro's **−10.71%** change, and BRTS **+9.09%** / AMTS **+2.50%**.
3. **Alerts & Anomalies:** explain the threshold alert. The Kalupur gate outage is a possible lead, not proof of cause.
4. **Transport Data:** show station and route counts and the incident register.
5. **Daily Brief:** generate the brief, then download the TXT or print to PDF.

---

## Architecture

```
MetroPulse/
├── server.js          Express server: static frontend, JSON API, in-memory upload/reset
├── lib/analysis.js    Calculations, threshold alerts, data-quality checks, report data
├── data/demo.json     Editable sample dataset (stations, routes, incidents)
├── public/            HTML / CSS / JavaScript interface (no remote assets)
├── work/smoke-test.js API and calculation smoke checks
└── package.json
```

### API

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Service status |
| GET | `/api/data` | Current dataset |
| GET | `/api/analysis` | Totals, changes, alerts, data-quality notes |
| GET | `/api/report` | Data for the daily brief |
| POST | `/api/data` | Validate and load an uploaded dataset |
| POST | `/api/data/reset` | Restore the bundled demo data |

### Dataset format

```json
{
  "meta": { "label": "...", "reportDate": "2026-10-10", "thresholdPercent": 10 },
  "systems": [
    { "name": "Metro", "yesterday": 28000, "today": 25000,
      "records": [{ "name": "Kalupur", "yesterday": 8200, "today": 7100 }] }
  ],
  "incidents": [
    { "id": "INC-101", "date": "2026-10-10", "system": "Metro", "location": "Kalupur",
      "type": "Ticket gate outage", "severity": "High",
      "description": "...", "status": "Resolved" }
  ]
}
```

Uploads must include `meta`, and `systems` containing **Metro, BRTS and AMTS**, each with non-negative numeric `yesterday` and `today`. Incidents, if present, need all eight fields.

---

## How the analysis works

- **Rule-based and transparent. No AI or black box.**
- Change % = (today − yesterday) / yesterday × 100.
- A system is flagged when the change reaches the threshold (default **10%**, set in `meta.thresholdPercent`).
- A decrease is **High** priority and an increase is **Review** priority.
- Incidents are matched to a system and reporting period only. **No causal inference is made.**

---

## Limitations

- Uploaded data is held in server memory and **resets when the server restarts**.
- The threshold is edited in the dataset JSON, not in the interface.
- Comparison is **day over day only**, with no trends or forecasts.
- Incident links are based on matching system and period, not proven cause.
- All included data is fictional.

---

## Tech

Node.js, Express, vanilla HTML/CSS/JavaScript, Canvas charts. No external scripts, fonts or APIs at runtime.
