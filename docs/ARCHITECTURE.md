# Architecture

## 1. System type

The repository is a static site served directly from the repository root:

- HTML entry points live at the root (`*.html`)
- shared JavaScript lives in `app.js` and `search.js`
- shared styling lives in `styles.css`
- content lives in `data\*.json`
- deployment is static (`vercel.json`, no build step)

There is no framework router, bundler, server-side rendering layer, or API backend in this repository.

## 2. Shared components

| Component | Path | Responsibility |
| --- | --- | --- |
| Cockpit renderer | `app.js` | Fetches cockpit data, renders zones/cards, manages detail blade, filters, cockpit search box, Mermaid/Prism integration, and cockpit deep links |
| Global search | `search.js` | Builds a command-palette-style cross-page index from multiple JSON catalogs and navigates to deep links |
| Shared styles | `styles.css` | Defines layout, theming, cards, blades, tables, and utility styles for all pages |
| Static deployment config | `vercel.json` | Serves repository root and applies cache headers |

## 3. Page architecture

| Page | Path | Main controller | Data input(s) | Main behaviors |
| --- | --- | --- | --- | --- |
| Cockpit | `index.html` | `app.js` | `data\copilot-instruments.json`, `data\security-threats.json`, `data\governance-controls.json`, `data\copilot-models.json` | Zone grid, filters, detail blade, cross-links to Security/Tower/Runway |
| Terminal | `terminal.html` | inline script | `data\terminal-guide.json` | Plan cards, IDE setup, first-flight exercises |
| Security | `security.html` | inline script | `data\copilot-instruments.json`, `data\security-threats.json`, `data\security-frameworks.json` | X-Ray scanner, threat diagrams, posture score, `#scan=` deep link |
| Jet Bridge | `jet-bridge.html` | inline script | `data\jet-bridge-guide.json` | Prompt/context/edit/agent tutorials |
| Ramp | `ramp.html` | inline script | `data\copilot-instruments.json` | Filters instruments by `perspectives.includes('ramp')`, blade, `#instrument-` deep link |
| Runway | `runway.html` | inline script | `data\copilot-models.json` | Model filters, topology, NOTAMs, model blade, `#model-` deep link |
| Tower | `tower.html` | inline script | `data\governance-controls.json`, `data\copilot-models.json`, `data\sovereign-cloud.json` | Governance registry, sovereignty matrix, flight plans, `#control=` and `#sovereign=` deep links |
| Flight Log | `flight-log.html` | inline script | `data\known-changelog-entries.json` | Timeline grouping and filtering |
| Pre-Flight | `preflight.html` | inline script | `data\preflight-checklist.json` | Checklist persistence and progress |
| Wiring | `wiring.html` | inline script | `data\wiring-diagram.json`, `data\copilot-instruments.json` | Mermaid graph, connection filters, stats |

Every page except the cockpit embeds its page-specific controller directly inside the HTML file.

## 4. Data flow patterns

### 4.1 Standard page boot flow

The dominant runtime pattern is:

1. HTML loads shared CSS.
2. Page controller reads `localStorage['cockpit-theme']`.
3. Page controller fetches one or more JSON files from `data\`.
4. On success, the page renders DOM from JSON arrays and objects.
5. On failure, the main container is replaced with a `DATA LINK LOST` message.

This pattern appears in:

- `app.js` for `index.html`
- inline scripts in `terminal.html`, `security.html`, `jet-bridge.html`, `ramp.html`, `runway.html`, `tower.html`, `flight-log.html`, `preflight.html`, `wiring.html`

### 4.2 Cockpit data flow

`index.html` plus `app.js` is the only page with a shared external controller.

Runtime flow:

1. `DOMContentLoaded` in `app.js`
2. Parallel fetches:
   - `data\copilot-instruments.json`
   - `data\security-threats.json`
   - `data\governance-controls.json`
   - `data\copilot-models.json`
3. Derived indexes are built:
   - `scannerIndex` from threat `instrumentId`
   - `governanceIndex` from governance control `id`
   - `_engineModels` derived from non-deprecated model catalog entries
4. `renderCockpit()`, `initFilters()`, `initSearch()`, and `handleDeepLink()` run
5. A detail blade opens for `#instrument-<instrumentId>` if present

### 4.3 Cross-page search flow

`search.js` is loaded on the site pages and builds a runtime search index by fetching:

- `data\copilot-instruments.json`
- `data\governance-controls.json`
- `data\copilot-models.json`
- `data\known-changelog-entries.json`

The overlay is opened by `Ctrl+K` or `Cmd+K`. Search results navigate by setting `window.location.href` to a generated deep link such as:

- `index.html#instrument-<id>`
- `tower.html#control=<id>`
- `runway.html#model-<id>`
- `flight-log.html`

### 4.4 Wiring flow

`wiring.html` uses `data\wiring-diagram.json` plus `data\copilot-instruments.json` to derive:

- filtered connection subsets
- Mermaid graph source
- zone summaries
- aggregate graph statistics

Node click targets are generated as cockpit deep links to `index.html#instrument-<id>`.

## 5. URL deep-link architecture

Deep links are part of the functional contract, not just navigation sugar.

| Page | Contract | Purpose |
| --- | --- | --- |
| `index.html` | `#instrument-<instrumentId>` | Open cockpit detail blade |
| `security.html` | `#scan=<instrumentId>` | Load a specific scanner item |
| `ramp.html` | `#instrument-<instrumentId>` | Open ramp blade |
| `runway.html` | `#model-<modelId>` | Open model blade |
| `tower.html` | `#control=<controlId>` | Highlight a governance control |
| `tower.html` | `#sovereign=<deploymentOptionId>` | Highlight a sovereignty option |

These hashes are also emitted by cross-page links from:

- `app.js` callouts inside cockpit detail blades
- `search.js` search results
- `wiring.html` node click links
- `flight-log.html` instrument links
- `security.html` "Open in Cockpit" links

## 6. `localStorage` architecture

Persistent browser state is intentionally small and page-scoped:

| Key | Shared or page-specific | Shape |
| --- | --- | --- |
| `cockpit-theme` | shared | string: `light` or `dark` |
| `cockpit-last-scan` | `security.html` | string instrument id |
| `cockpit-security-posture` | `security.html` | JSON object `{ [postureId]: boolean }` |
| `copilot-preflight` | `preflight.html` | JSON object `{ [itemId]: boolean }` |

Filter state on cockpit, runway, and flight log pages is not persisted.

## 7. Optional data-maintenance tooling

`tools\enrich\*` implements a model-catalog enrichment pipeline for `data\copilot-models.json`:

- `tools\enrich\harvest.py` fetches upstream sources into `tools\cache\`
- `tools\enrich\normalize.py` merges cached adapter output into a candidate file
- `tools\enrich\cache.py` manages cache paths and optional `.env` loading
- adapters live under `tools\enrich\adapters\`

Important design rule from the tooling itself: the pipeline is human-gated and should not write directly to `data\copilot-models.json`.

## 8. Architectural risks

| Risk | Why it matters |
| --- | --- |
| JSON file drift | Pages depend on exact key names and cross-file ids with minimal schema enforcement in browser code |
| External CDN/runtime dependencies | Fonts, Prism, Mermaid, Vercel scripts, and embedded YouTube iframes are external dependencies |
| Partial verification in catalogs | `data\copilot-models.json` and `data\security-frameworks.json` explicitly mark verification uncertainty |
| Static-only failure handling | Missing or malformed JSON degrades to page-level error text; there is no retry or recovery layer |
| Inline page scripts | Most pages embed logic in HTML files, so behavior changes are spread across many entry points |

## 9. Assumptions and uncertainty

- **Assumption:** The static-site architecture is intentional and there is no hidden build/deploy preprocessing step beyond what is visible in the repository.
- **Assumption:** All runtime HTML pages are expected to be served from the repository root so that relative `data\...` fetch paths resolve unchanged.
