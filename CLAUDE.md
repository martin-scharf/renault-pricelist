# CLAUDE.md — AI Assistant Guide for renault-pricelist

## Project Overview

**OE Projekt 1** is a single-page web tool for managing Renault automotive parts price lists with discount administration. It allows importing supplier CSV data, managing per-family discounts, and generating customer and internal (CSS/ERP) price lists as CSV downloads.

- **Language:** German UI throughout (labels, error messages, placeholders)
- **Domain:** Supply chain / pricing management for Renault OE (Original Equipment) parts

## Architecture

This is a **zero-dependency, single-file application**. The entire app lives in `index.html` (~867 lines) containing inline HTML, CSS, and JavaScript. There is no build system, no package manager, no backend, and no external libraries.

```
renault-pricelist/
├── index.html    ← Entire application (HTML + CSS + JS)
└── CLAUDE.md     ← This file
```

### Technology Stack

- **HTML5** — Semantic structure with two-tab SPA layout
- **CSS3** — Dark theme (#0f0f1a background, #e94560 Renault-red accent), Flexbox/Grid, responsive breakpoints at 900px/768px/600px
- **Vanilla JavaScript (ES6+)** — No frameworks, no transpilation
- **Browser localStorage** — Sole persistence mechanism (key: `renault_families`)

## Code Structure (within index.html)

| Section | Lines | Description |
|---|---|---|
| HTML markup | 1–524 | Nav bar, Generator page (upload + stats + downloads), Admin page (family table + config actions) |
| `<style>` | 7–420 | All CSS including dark theme, component styles, responsive breakpoints, toast notifications |
| `<script>` | 528–864 | All application logic |

### Key JavaScript Components

| Function/Area | Lines | Purpose |
|---|---|---|
| State variables | 530–532 | `parsedData[]`, `families{}`, `articlesByFamily{}` |
| `loadFamilies()` / `saveFamiliesToStorage()` | 535–544 | localStorage read/write |
| `showPage()` | 550–560 | Tab navigation (Generator / Admin) |
| File upload handlers | 562–601 | Click, drag-and-drop, FileReader |
| `parseCSV()` | 603–669 | CSV parsing with auto-separator detection (`;` or `,`), German decimal support |
| `renderFamiliesTable()` | 692–736 | Admin table rendering with search/filter |
| `updateFamily()` | 742–746 | In-memory family field update |
| `downloadCustomerList()` | 754–775 | Generate customer CSV (article, list_price, buying_price) |
| `downloadFullList()` | 777–800 | Generate full CSV with all fields |
| `exportConfig()` / `handleConfigImport()` | 815–854 | JSON config export/import (version 1 format) |
| `showToast()` | 857–863 | Toast notification display |

### Data Model

**CSV columns parsed (in order):** article, list_price, buying_price, buying_price_self, discount, discount_self, description, weight, dlnr

**Family key format:** `{discount}_{discount_self}` (e.g., `"10_5"`)

**Price calculation:** `buyingPrice = listPrice * (1 - discount / 100)`

**Config JSON format:**
```json
{
  "version": 1,
  "exported": "ISO_DATE",
  "families": { "<key>": { "code", "description", "discountCustomer", "discountSelf" } }
}
```

## Development Workflow

### Running the Application

Open `index.html` directly in a browser — no server, build step, or installation needed.

```bash
# Any of these work:
open index.html              # macOS
xdg-open index.html          # Linux
start index.html             # Windows
python3 -m http.server 8000  # Serve via local HTTP if needed
```

### Making Changes

1. Edit `index.html` directly — all HTML, CSS, and JS are in this one file
2. Refresh the browser to see changes
3. No linting, formatting, or type-checking tools are configured

### Testing

There are **no automated tests**. All testing is manual:
- Upload a CSV file and verify parsed stats
- Check discount calculations in downloaded CSVs
- Test config export/import round-trip
- Verify localStorage persistence across page reloads

## Conventions

### Code Style

- **Variables/functions:** camelCase (`parsedData`, `saveFamiliesToStorage`)
- **DOM interaction:** `document.getElementById()` / `document.querySelector()`
- **Event handling:** Mix of inline `onclick`/`oninput` in HTML and `addEventListener` in JS
- **Error feedback:** Toast notifications via `showToast(message, isError)`
- **No module system:** Everything is in global scope within the single `<script>` block

### CSS Conventions

- BEM-like class names: `.upload-section`, `.stat-card`, `.download-card`, `.table-row`
- Hardcoded color values (no CSS custom properties)
- Key colors: `#0f0f1a` (background), `#1a1a2e` (card bg), `#e94560` (accent red), `#4ade80` (success green)

### Localization

- All UI text is in **German**. Maintain German for any user-facing strings.
- Number formatting uses `toLocaleString('de-DE')` for display
- CSV parsing handles both comma and dot as decimal separators

## Key Considerations When Modifying

- **Single-file architecture:** All changes go in `index.html`. Do not split into separate files unless explicitly requested.
- **No dependencies:** Do not introduce npm packages, CDN imports, or external libraries without explicit request.
- **localStorage limits:** Data is browser-local only. There is no sync, no backend, no database.
- **CSV format sensitivity:** The parser auto-detects separators (`;` vs `,`) and requires minimum 3 columns. Changes to parsing logic should maintain this flexibility.
- **Family key stability:** Families are keyed by `{discount}_{discount_self}`. Changing this format would break existing localStorage data.
- **German language:** Keep all user-facing text in German.

## Git Conventions

Commit messages follow the pattern: `type: description` in German or English.

Examples from history:
- `fix: flexibler CSV parser, besseres Feedback`
- `fix: reset file input after import`
- `rename: OE Projekt 1`
- `Admin-Backend mit Rabattverwaltung`
