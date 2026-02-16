# CLAUDE.md - AI Assistant Guide for green-mobile-checks

## Project Overview

Green Mobile Checks (צ'קים) is a single-page application for managing financial check transactions for Green Mobile. It fetches data from Monday.com via GraphQL API, displays checks with filtering/sorting capabilities, and supports adding new checks with file uploads.

The entire application is contained in a single file: **`index.html`** (~2,800 lines of embedded HTML, CSS, and JavaScript).

## Tech Stack

- **Language**: Vanilla JavaScript (ES6+), no framework
- **Markup**: HTML5 with RTL (Hebrew) support
- **Styling**: CSS3 embedded in `<style>` tags (~1,500 lines)
- **Font**: Google Fonts - Heebo (Hebrew-optimized)
- **Backend**: Monday.com GraphQL API (v2, version 2024-10)
- **File Upload Proxy**: Cloudflare Workers (`monday-upload.greenmobile-eshop.workers.dev`)

## Architecture

### No Build System

There is **no package.json, no bundler, no transpiler, and no build step**. The app is a static HTML file served directly. No npm, yarn, or any package manager is used.

### No Tests

There are **no test files, test frameworks, or test configurations**.

### No Linting/Formatting

There are **no ESLint, Prettier, or other code quality tools** configured.

### No CI/CD

There are **no GitHub Actions, pipelines, or automated deployment** configurations.

## Project Structure

```
green-mobile-checks/
  index.html          # The entire application (HTML + CSS + JS)
  CLAUDE.md           # This file
```

## Application Architecture (inside index.html)

### CSS Section (~lines 1-1500)
- Responsive design with breakpoints at 1400px, 900px, 600px, 500px
- Mobile-first approach
- RTL layout throughout
- Custom card-based UI with gradients and shadows

### HTML Section
- Loading screen with spinner
- Header: logo, live indicator, refresh button, add-check button, Monday.com link
- Alert banner: marquee for dates with checks exceeding threshold (200,000 ILS)
- Stats grid: 4 summary cards (Total, Pending, Completed, Cancelled)
- Filters: search, date range, quick-date shortcuts, bank/supplier dropdowns, sort controls
- Checks list: grid of check cards with images, details, and actions
- Modals: image viewer, alerts detail, add-check form

### JavaScript Section
- Global state variables (`allChecks`, `filteredChecks`, `currentFilters`, `sortBy`, `sortOrder`)
- Data fetching with cursor-based pagination (500 items per page)
- Client-side filtering and sorting
- Auto-refresh every 5 minutes
- Inline event handlers + `addEventListener` for inputs

### Key Functions
| Function | Purpose |
|---|---|
| `loadData()` | Fetch checks from Monday.com with pagination |
| `applyFilters()` | Client-side filtering and sorting |
| `updateStats()` | Recalculate summary statistics |
| `renderChecks()` | Render check cards to DOM |
| `openAddCheckModal()` | Open the add-check form modal |
| `submitNewCheck()` | Create a new check via Monday.com API |
| `uploadFileToMonday()` | Upload file via Cloudflare Worker proxy |
| `filterByGroup()` | Filter by check status group |
| `setQuickDate()` | Apply date range shortcuts |
| `openImageModal()` / `closeImageModal()` | View check images |

### Data Model (Check object)
```javascript
{
  id: string,            // Monday.com item ID
  name: string,          // Check number
  numbers__1: number,    // Amount in ILS
  date__1: date,         // Due date
  status: string,        // Supplier name
  bank: string,          // Bank name
  notes: string,         // Notes
  group: string,         // Status group ID (topics|group_title|group_mkzhynwd)
  files__1: array,       // Attached image files
  entry_date: date       // Entry date
}
```

### Monday.com Group Mapping
| Group ID | Meaning |
|---|---|
| `topics` | Pending (ממתינים) |
| `group_title` | Completed (הושלמו) |
| `group_mkzhynwd` | Cancelled (בוטלו) |

## External Services

- **Monday.com API**: `https://api.monday.com/v2` - Board ID `1689501530`
- **Cloudflare Workers**: `https://monday-upload.greenmobile-eshop.workers.dev` - File upload proxy
- **Google Fonts**: Heebo font family

## Localization

- **Language**: Hebrew (hardcoded, no i18n framework)
- **Text direction**: RTL (`dir="rtl"`)
- **Date format**: `he-IL` locale
- **Currency**: Israeli Shekel (ILS / &#8362;)
- **Time format**: 24-hour

## Development Workflow

### Running Locally
Serve `index.html` via any HTTP server. The app auto-loads data on `DOMContentLoaded` and auto-refreshes every 5 minutes.

```bash
# Example using Python
python3 -m http.server 8080

# Example using Node.js
npx serve .
```

Then open `http://localhost:8080/index.html` in a browser.

### Making Changes
1. Edit `index.html` directly - all code is in this single file
2. CSS is in the `<style>` block at the top
3. HTML structure is in the `<body>` section
4. JavaScript logic is in the `<script>` block at the bottom
5. Test changes by refreshing the browser

### Git Conventions
- All commits historically use simple messages ("Add files via upload")
- Single branch workflow on `main`

## Coding Conventions

- **No modules**: All JavaScript is in a single `<script>` block with global scope
- **Inline handlers**: Many UI elements use `onclick` attributes
- **Template literals**: HTML is built using JavaScript template literals and `innerHTML`
- **Async/await**: Used for all API calls
- **Naming**: camelCase for JavaScript functions and variables
- **Numbers**: Formatted with `toLocaleString('he-IL')` for Israeli number formatting
- **Dates**: Formatted with `toLocaleDateString('he-IL')` options

## Responsive Breakpoints

| Breakpoint | Target |
|---|---|
| 1400px+ | Desktop - full layout, 4-column stats |
| 900px - 1399px | Tablet - 2-column stats |
| 600px - 899px | Mobile - adjusted header, flexible spacing |
| < 500px | Small mobile - compact layout, smaller text |

## Important Notes for AI Assistants

1. **Single file application**: All changes go in `index.html`. Do not split into separate files unless explicitly requested.
2. **No build step**: Changes are immediately reflected by refreshing the browser. No compilation needed.
3. **API token is embedded**: The Monday.com API token is hardcoded in the JavaScript. Be aware of this when sharing code.
4. **Hebrew RTL**: All text is Hebrew and the layout is right-to-left. Maintain RTL conventions when adding UI elements.
5. **No dependencies to install**: There is no `package.json` or `node_modules`. The project has zero local dependencies.
6. **Auto-refresh**: The app polls Monday.com every 5 minutes. Keep this in mind when debugging data issues.
7. **Threshold alerts**: Checks pending on a single date exceeding 200,000 ILS trigger a marquee alert banner.
