# Testing guide

## Test stack

The repository uses Playwright only.

| File | Role |
| --- | --- |
| `package.json` | exposes `npm test` as `npx playwright test` |
| `playwright.config.js` | sets `tests/`, base URL, Chromium project, and a local static `webServer` |
| `tests/*.spec.js` | page-level end-to-end coverage plus JSON integrity checks |

Key Playwright settings currently in force:

- `testDir: './tests'`
- `baseURL: 'http://localhost:3000'`
- `reporter: 'list'`
- Chromium-only project
- local server command: `python3 -m http.server 3000 --bind 127.0.0.1`

## How to run the suite

Run everything:

```bash
npm test
```

Run a single spec:

```bash
npx playwright test tests/runway.spec.js
```

Environment prerequisites implied by the repo:

- Node.js and npm
- Playwright browser binaries
- `python3` on `PATH` for the local static server

First-time setup in a clean workspace is therefore:

```bash
npm install
npx playwright install chromium
```

## Current suite size

The current test source declares **222 tests across 11 spec files**.

| Spec file | Declared tests | Coverage summary |
| --- | ---: | --- |
| `tests/cockpit.spec.js` | 29 | cockpit boot, zones, blade behavior, deep links, theme, filters, code and media tabs, cockpit-local search |
| `tests/terminal.spec.js` | 17 | terminal structure, plan cards, IDE cards, exercises, departures |
| `tests/security.spec.js` | 37 | scanner behavior, Mermaid threat diagrams, framework links, `localStorage`, cockpit bridge, print button |
| `tests/jet-bridge.spec.js` | 17 | prompt craft, context management, edit workflows, agent patterns, next steps |
| `tests/ramp.spec.js` | 15 | ramp cards, blade lifecycle, deep links, metaphor key |
| `tests/runway.spec.js` | 31 | filters, departure board, model blade, topology, NOTAMs, engine, cockpit bridge |
| `tests/tower.spec.js` | 25 | governance controls, frameworks, sovereignty views, flight plans, deep links |
| `tests/flight-log.spec.js` | 15 | timeline render, filters, entry content, nav, theme |
| `tests/preflight.spec.js` | 13 | checklist render, progress, `localStorage`, reset |
| `tests/wiring.spec.js` | 14 | graph render, filters, legend, zones, stats |
| `tests/integrity.spec.js` | 9 | cross-file data integrity and duplicate-ID checks |

## Observed baseline during this documentation pass

One full run was executed after installing npm dependencies and Playwright Chromium.

| Result | Count |
| --- | ---: |
| Passed | 216 |
| Failed | 6 |

All 6 observed failures are in `tests/cockpit.spec.js` and center on cockpit filter or search expectations:

1. `Page Load › shows instrument count in header after filter interaction`
2. `Filters › flight mode filter dims non-matching instruments`
3. `Filters › plan filter works`
4. `Filters › status filter shows only GA or Preview`
5. `Search › filters instruments by name`
6. `Search › / focuses search when page has focus`

This is useful context for contributors: the repository currently has a known non-green baseline in the cockpit filter and search area.

## Coverage themes

### 1. Page boot and rendering

Most page specs assert:

- no page errors or unexpected console errors,
- expected landmarks render,
- active navigation state is correct,
- data-backed cards, rows, or timeline entries are present.

### 2. Deep-link coverage

The suite explicitly covers:

| Contract | Covered in |
| --- | --- |
| `#instrument-<id>` on cockpit | `tests/cockpit.spec.js` |
| `#scan=<id>` on security | `tests/security.spec.js` |
| `#instrument-<id>` on ramp | `tests/ramp.spec.js` |
| `#model-<id>` on runway | `tests/runway.spec.js` |
| `#control=<id>` and `#sovereign=<id>` on tower | `tests/tower.spec.js` |

### 3. Persistence coverage

The suite directly exercises:

- `cockpit-theme`
- `cockpit-last-scan`
- `cockpit-security-posture`
- `copilot-preflight`

### 4. Rich-rendering coverage

The suite checks that Mermaid-based sections render to SVG on:

- Security
- Runway
- Tower
- Wiring

It also checks Prism-based code highlighting in the cockpit Code tab.

### 5. Data integrity coverage

`tests/integrity.spec.js` validates:

- duplicate instrument IDs,
- duplicate model IDs,
- required instrument fields,
- valid zone references,
- valid changelog entry types,
- valid wiring connection types,
- cross-file reference integrity for changelog, wiring, and related instruments.

## Coverage limits and known gaps

| Gap | Why it matters |
| --- | --- |
| No dedicated spec for the global `search.js` overlay | command-palette ranking, result composition, and keyboard navigation can regress without direct detection |
| No visual-regression baseline | layout and styling regressions can pass functional checks |
| No explicit accessibility audit | keyboard and ARIA behavior are only partially covered |
| No runtime corruption tests for `cockpit-security-posture` | malformed storage JSON can still break Security page state |
| No assertion of Vercel headers or cache behavior | deployment-time caching issues are outside the current suite |
| No integrity checks for model `flightPlans` or `notams` references | some model cross-links are trusted at runtime rather than validated centrally |

## Coverage scope by risk area

| Risk area | Current protection |
| --- | --- |
| Broken page boot | strong page-level coverage |
| Broken hash routes | strong for cockpit, security, ramp, runway, and tower |
| Broken theme persistence | moderate, covered on cockpit, security, and flight log |
| Broken cross-catalog IDs | moderate, covered by `integrity.spec.js` for major joins |
| Broken global search overlay | weak, no dedicated spec |
| Cache and deployment regressions | weak, not exercised in Playwright |

## Suggested contributor workflow

1. Run `npm test`.
2. Change JSON or runtime code.
3. Update the closest page-specific spec.
4. If IDs or relationships changed, review `tests/integrity.spec.js`.
5. Re-run `npm test`.

## Explicit assumption

- The test counts above are source-derived counts of `test(...)` declarations in the current repository state. No dynamically generated tests were observed.
