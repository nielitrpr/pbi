# FutureSkills PRIME — Training Dashboard

Live registration dashboard for **Bureau of Investigation, Punjab Police** — built for the NIELIT Ropar FutureSkills PRIME training program.

Fetches participant data directly from a published Google Sheet and renders real-time stats, charts, and a searchable/sortable table — all in a single HTML file with zero backend.

---

Sheet Live Link: https://docs.google.com/spreadsheets/d/1X_Chv3xMYSZPQokTfH9ft5bI4Pba1fN1H-Uxj8xSXqc/edit?usp=sharing

## Features

| Feature | Detail |
|---|---|
| **Live data** | Auto-fetches from Google Sheets every 60 seconds |
| **Stat cards** | Total, Male, Female, Designations — with delta detection on refresh |
| **Charts** | Gender (doughnut), Qualification (bar), Designation (bar) |
| **Search** | Instant filter across name, father, email, designation, contact, qualification |
| **Column sorting** | Click any table header — toggles ascending/descending |
| **Pagination** | 25 / 50 / 100 / 200 rows per page |
| **CSV export** | One-click download with BOM for Excel compatibility |
| **Qualification normalization** | 16 categories (B.Tech, MCA, MBA, 12th, etc.) — cleans messy free-text entries |
| **XSS safe** | All user-supplied text is HTML-escaped before rendering |
| **Responsive** | Works on desktop, tablet, and mobile |
| **Print friendly** | Hides controls, avoids page-break inside cards |
| **Offline graceful** | Shows last loaded data on network failure; clear error if no data yet |

---

## Architecture

```
┌──────────────┐        ┌─────────────────────────────────┐
│  Google      │  /pub  │                                 │
│  Sheet       ├───────►│  Browser (single HTML file)     │
│  (Published) │  CSV   │                                 │
│              │◄───────┤  fetch() → parseCSV() → render  │
└──────────────┘  cache- │       Charts + Table + Stats    │
                    bust └─────────────────────────────────┘
```

No server. No API key. No build step. The browser fetches the CSV directly from Google's "Publish to web" endpoint.

---

## Setup

### 1. Publish your Google Sheet

1. Open your Google Sheet
2. Go to **File → Share → Publish to web**
3. Under "Entire Document", select **CSV** as the format
4. Click **Publish**, confirm
5. Copy the URL — it looks like:

```
https://docs.google.com/spreadsheets/d/e/2PACX-1v.../pub?output=csv
```

### 2. Update the URL in the dashboard

Open `index.html`, find this line near the top of the `<script>`, and paste your URL:

```js
const CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1v.../pub?output=csv";
```

### 3. Ensure your sheet has these columns

The parser expects these exact column headers in Row 1:

| Column Header | Used For |
|---|---|
| `Full Name` | Name display, search |
| `Father Name` | Table column, search |
| `Gender` | Stats, gender chart, badges |
| `Highest/Current Qualification (with Branch /domain as applicable)` | Normalized into qualification chart |
| `Designation (if applicable)` | Designation chart, search |
| `Email` | Table, search |
| `Contact Number` | Table, search |
| `Timestamp` | Registration date display |

> **Tip:** Extra columns are preserved in memory and included in CSV export — they just won't appear in the table. You can add more columns to the `COLS` array in the script to display them.

### 4. Host it

Any static hosting works:

- **GitHub Pages** — push to a repo, enable Pages
- **Netlify** — drag and drop the HTML file
- **Local** — just double-click `index.html`
- **NIELIT internal server** — drop into any webroot

---

## Customization

### Change refresh interval

```js
const REFRESH_SEC = 60;  // change to 30, 120, 300, etc.
```

### Change page size options

```js
const PAGE_SIZES = [25, 50, 100, 200];  // add or remove values
```

### Add a table column

Find the `COLS` array and append:

```js
const COLS = [
  // ... existing columns ...
  { key: 'District',  label: 'District', w: '120px', numeric: false },
  { key: 'Batch',     label: 'Batch',    w: '80px',  numeric: false },
];
```

The `key` must exactly match the column header in your Google Sheet.

### Change colors

All colors are CSS custom properties at the top of `<style>`:

```css
:root {
  --accent: #1e3a5f;    /* navy blue — primary */
  --accent2: #c8440a;   /* burnt orange — female / designation */
  --accent3: #1a6b3c;   /* dark green — male / qualified */
  --gold: #b8860b;      /* gold — designations stat */
  --bg: #f5f4f0;        /* page background */
  /* ... */
}
```

### Change qualification categories

Edit the `normalizeQual()` function. It runs a regex cascade — first match wins:

```js
function normalizeQual(raw) {
  const s = raw.toLowerCase();
  if (/m\.?tech/.test(s))  return 'M.Tech';
  if (/mca/.test(s))       return 'MCA';
  // add your own rules here:
  if (/b\.?e/.test(s))     return 'B.E';
  // ...
}
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| "Failed to fetch" | Sheet not published, or wrong URL | Re-publish: File → Share → Publish to web → CSV |
| "Parsed 0 rows" | Column headers don't match | Ensure Row 1 has exact header names listed above |
| "Empty response" | Publish URL is for a specific sheet/tab, not entire document | In Publish dialog, select "Entire Document" not a single sheet |
| Data stale after sheet update | Google caches `/pub` output for ~1–5 min | This is Google's CDN cache — unavoidable. Dashboard still auto-refreshes and picks up changes when cache clears. |
| Charts show wrong percentages | N/A (fixed) | Previous version had a bug dividing by wrong total. Current version uses correct per-chart totals. |
| CSV export opens as garbled text in Excel | Missing BOM | Fixed — export prepends `\uFEFF` BOM byte for proper UTF-8 Excel detection |
| Qualifications showing as raw text | New qualification type not in `normalizeQual()` | Add a regex rule for it in the function |

### Google's /pub cache

Google CDN caches the `/pub?output=csv` response for **1 to 5 minutes** after a change. There is no way to bypass this from the client side. The dashboard fetches every 60 seconds, so it will pick up changes within one cache cycle. If you need instant updates, you'd need a server-side proxy (e.g., a Cloud Function that fetches and re-serves the CSV).

---

## File Structure

```
project/
├── index.html      ← Everything lives here (HTML + CSS + JS)
└── README.md       ← This file
```

No `package.json`. No `node_modules`. No build tools. One file.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Structure | Semantic HTML5 |
| Styling | Custom CSS (no framework) |
| Charts | Chart.js 4.4 (CDN) |
| Icons | Font Awesome 6.5 (CDN) |
| Fonts | IBM Plex Sans + IBM Plex Mono (Google Fonts) |
| Data | Google Sheets "Publish to web" CSV endpoint |
| Parsing | Custom CSV state-machine parser (handles quoted fields, embedded commas, escaped quotes) |

---

## Credits

- **MeitY** — Ministry of Electronics & Information Technology
- **NASSCOM** — FutureSkills PRIME initiative
- **NIELIT Ropar** — Training facilitation
- **Bureau of Investigation, Punjab Police** — Participant organization

---

## License

This dashboard is built for internal use by BOI Punjab Police / NIELIT Ropar. No external license restrictions on the code — use as needed.

