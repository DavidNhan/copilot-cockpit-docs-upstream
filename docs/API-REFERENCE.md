# API reference

This repository has no server-side API. The effective runtime API is the contract between:

1. root HTML pages,
2. shared browser scripts,
3. JSON catalogs under `data/`,
4. URL hash fragments, and
5. `localStorage`.

Use this file when you need to change behavior safely.

## Shared runtime module contracts

| Module | Contract surface | Consumers |
| --- | --- | --- |
| `app.js` | cockpit boot, derived indexes, filters, blade tabs, deep links, theme toggle, Mermaid and Prism hooks | `index.html` |
| `search.js` | global search index, overlay lifecycle, keyboard handling, deep-link generation | all root pages |
| page inline scripts | page-specific rendering, fetch lifecycles, theme restoration, page-local hashes and state | every non-cockpit page |

## Page fetch contracts

All fetches are relative to the repository root.

| Page | Required fetches | Optional or soft-failed fetches | Output contract |
| --- | --- | --- | --- |
| `index.html` via `app.js` | `data/copilot-instruments.json` | `data/security-threats.json`, `data/governance-controls.json`, `data/copilot-models.json` | cockpit grid, detail blade, callout indexes, engine-zone model links |
| `terminal.html` | `data/terminal-guide.json` | none | 4 guided sections |
| `security.html` | `data/copilot-instruments.json`, `data/security-threats.json`, `data/security-frameworks.json` | none | scanner, framework chips, posture checklist |
| `jet-bridge.html` | `data/jet-bridge-guide.json` | none | tutorial cards and next-step links |
| `ramp.html` | `data/copilot-instruments.json` | none | ramp subset and ramp blade |
| `runway.html` | `data/copilot-models.json` | none | model board, blade, topology, NOTAMs, engine summary |
| `tower.html` | `data/governance-controls.json`, `data/copilot-models.json`, `data/sovereign-cloud.json` | none | controls, sovereignty data, flight plans |
| `flight-log.html` | `data/known-changelog-entries.json` | none | grouped timeline and filters |
| `preflight.html` | `data/preflight-checklist.json` | none | checklist, progress, reset |
| `wiring.html` | `data/wiring-diagram.json`, `data/copilot-instruments.json` | none | graph, filters, legend, stats |

### Important fetch behavior differences

- `app.js` hard-fails if `copilot-instruments.json` fails, but treats threats, governance controls, and models as optional enrichments.
- `search.js` catches index build failure and falls back to an empty search dataset.
- Most inline page controllers replace the main content area with a `DATA LINK LOST` message on fetch failure.
- `security.html` loads data with `Promise.all(...then(r => r.json()))` and does not wrap that load in a page-level `try/catch`, so malformed or missing data can fail more abruptly than on other pages.

## Hash contracts

| Hash | Owner | Accepted values | Behavior |
| --- | --- | --- | --- |
| `#instrument-<instrumentId>` | `index.html` | instrument IDs from `data/copilot-instruments.json` | opens cockpit blade; `pushState` on open; hash cleared on close |
| `#scan=<instrumentId>` | `security.html` | threat-backed instrument IDs | loads selected scan; falls back to stored scan or first threat |
| `#instrument-<instrumentId>` | `ramp.html` | instrument IDs in the ramp subset | opens ramp blade; ignored if ID is not in the ramp-filtered set |
| `#model-<modelId>` | `runway.html` | model IDs from `data/copilot-models.json` | opens model blade; cleared on close |
| `#control=<controlId>` | `tower.html` | governance control IDs | highlights and scrolls to a control row |
| `#sovereign=<deploymentOptionId>` | `tower.html` | sovereignty deployment option IDs | highlights and scrolls to an option card |

### Hash producers

| Producer | Output |
| --- | --- |
| cockpit detail callouts in `app.js` | `security.html#scan=<id>`, `tower.html#control=<id>`, `runway.html` |
| `search.js` | cockpit, tower, runway, and flight-log links |
| `wiring.html` | `index.html#instrument-<id>` click targets in Mermaid graph |
| `flight-log.html` | instrument links back to the cockpit |
| `security.html` | `index.html#instrument-<id>` via "Open in Cockpit" |

## `localStorage` contracts

| Key | Owner | Value shape | Read behavior | Write behavior |
| --- | --- | --- | --- | --- |
| `cockpit-theme` | nearly every page | `"light"` or `"dark"` | page boot applies light theme if value is `"light"` | theme toggle writes `"light"` or `"dark"` |
| `cockpit-last-scan` | `security.html` | instrument ID string | used if `#scan=` is absent | updated when a new scan is loaded |
| `cockpit-security-posture` | `security.html` | JSON object `{ [postureId]: boolean }` | parsed on render and score update | each checkbox change rewrites the whole object |
| `copilot-preflight` | `preflight.html` | JSON object `{ [itemId]: boolean }` | parsed during checklist boot | each item toggle rewrites the whole object |

### Storage resilience notes

- `preflight.html` wraps checklist-state parsing in `try/catch` and falls back to `{}`.
- `security.html` does **not** wrap `cockpit-security-posture` parsing in `try/catch`, so malformed JSON can break posture rendering or score updates.
- Theme state is duplicated across many page controllers; there is no shared theme helper outside `app.js`.

## Search contracts

`search.js` creates in-memory search entries from four catalogs:

| Source | Indexed item type | URL emitted |
| --- | --- | --- |
| `data/copilot-instruments.json` | `instrument` | `index.html#instrument-<id>` |
| `data/governance-controls.json` | `control` | `tower.html#control=<id>` |
| `data/copilot-models.json` | `model` | `runway.html#model-<id>` |
| `data/known-changelog-entries.json` | `changelog` | `flight-log.html` |

### Search keyboard contract

| Key | Effect |
| --- | --- |
| `Ctrl+K` or `Cmd+K` | open or close the global overlay |
| `Escape` | close the overlay |
| `ArrowUp` and `ArrowDown` | change active result |
| `Enter` | navigate to the active result |

### Cockpit-local search contract

Inside `app.js`, the cockpit page also has a separate text filter:

- `#search-input` filters instrument cards client-side by simple text inclusion.
- `/` focuses `#search-input` when the active element is not an input.
- This is distinct from the global `search.js` overlay.

## Rendering conventions

| Convention | Current behavior |
| --- | --- |
| HTML escaping | page controllers escape JSON-derived text before inserting it |
| Mermaid rendering | lazy or post-render depending on page; several pages re-render diagrams on theme switch |
| Prism usage | cockpit code tab highlights visible code blocks when the Code tab opens |
| Error message | most pages render `DATA LINK LOST - <message>` in the primary content area |

### Mermaid security-level contract

| Page or runtime | Mermaid security level |
| --- | --- |
| `app.js` | `loose` |
| `tower.html` | `loose` |
| `wiring.html` | `loose` |
| `security.html` | explicit `mermaid.render(...)` path |

Because diagram content comes from repository-controlled JSON or inline markup, this is currently a maintainability choice rather than a direct user-input risk. If diagrams ever become user-authored or externally sourced, revisit this immediately.

## Cross-file ID contracts

| Source field | Target field | Used by |
| --- | --- | --- |
| `security-threats.threats[].instrumentId` | `copilot-instruments.instruments[].id` | Security page, cockpit X-Ray callout index |
| `governance-controls.controls[].id` | matching instrument IDs for selected controls | cockpit governance callout index, Tower deep links, search, Wiring |
| `known-changelog-entries.entries[].instruments[]` | `copilot-instruments.instruments[].id` | Flight Log links, search, integrity tests |
| `copilot-instruments.instruments[].relatedInstruments[]` | `copilot-instruments.instruments[].id` | cockpit related links |
| `wiring-diagram.connections[].from/to` | instrument IDs or governance control IDs | Wiring graph and integrity tests |
| `copilot-models.notams[].affectedModels[]` | model IDs | Runway NOTAM links |
| `copilot-models.flightPlans[].recommendedModels[]` and `.avoidModels[]` | model IDs | Tower flight plans |

The first five relationships are explicitly validated by `tests/integrity.spec.js`. The last two are runtime assumptions backed by page behavior, not by the integrity spec today.

## Explicit assumptions

- These contracts are internal repository contracts. There is no evidence in the codebase of an external compatibility guarantee for third-party consumers.
- Relative JSON paths are expected to remain valid because the site is served from the repository root or an equivalent static root.
