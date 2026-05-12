# Testing Guide

## 1. Test stack

The repository uses Playwright only.

| File | Role |
| --- | --- |
| `package.json` | defines `npm test` as `npx playwright test` |
| `playwright.config.js` | configures `tests\` as the suite directory and starts a local static server |
| `tests\*.spec.js` | end-to-end and integrity specs |

Playwright configuration details currently relevant to contributors:

- `testDir: './tests'`
- `baseURL: 'http://localhost:3000'`
- browser project: Chromium
- local server command: `python3 -m http.server 3000 --bind 127.0.0.1`

## 2. How tests are intended to run

Primary command:

```bash
npm test
```

Target a single spec:

```bash
npx playwright test tests\runway.spec.js
```

Prerequisites implied by repository configuration:

- Node.js and npm
- Playwright browser dependencies
- `python3` available on the local machine for the static test server

## 3. Current execution status for this documentation pass

- **Assumption:** Test coverage below is taken from repository source.
- `npm test` could not be executed in the documentation-generation environment because `pwsh.exe` was missing, so current runtime pass/fail status was not established here.

## 4. Test inventory

There are 11 Playwright spec files in `tests\`.

| Spec file | Focus |
| --- | --- |
| `tests\cockpit.spec.js` | Cockpit page boot, zone rendering, detail blade, deep links, theme persistence, filters, media/code tabs |
| `tests\terminal.spec.js` | Terminal page landmarks, plan cards, IDE setup cards, exercises, departures, nav state |
| `tests\security.spec.js` | Security page boot, scanner flows, Mermaid threat diagrams, framework links, deep links, posture-score persistence, cockpit bridge |
| `tests\jet-bridge.spec.js` | Prompt craft, context management, edit-mode cards, agent patterns, next-step links |
| `tests\ramp.spec.js` | Ramp card rendering, blade behavior, deep links, backdrop/Escape close, nav promotion |
| `tests\runway.spec.js` | Runway boot, filter behavior, departure board, model blade, topology, NOTAMs, engine section, cockpit bridge |
| `tests\tower.spec.js` | Tower boot, framework chips, governance controls, sovereignty section, flight plans, deep-link highlighting |
| `tests\flight-log.spec.js` | Changelog timeline rendering, filters, entry content, nav, theme behavior |
| `tests\preflight.spec.js` | Checklist rendering, progress, `localStorage` persistence, reset flow, nav presence |
| `tests\wiring.spec.js` | Wiring graph render, filters, legend, zone map, stats, nav presence |
| `tests\integrity.spec.js` | Node-side JSON cross-reference and schema-integrity checks |

## 5. Coverage themes

### 5.1 Rendering and boot coverage

Nearly every page has boot tests that assert:

- page loads without console/page errors
- main content landmarks render
- active navigation state is correct

### 5.2 URL hash contract coverage

Deep-link behavior is explicitly covered for:

- cockpit: `#instrument-<id>`
- security: `#scan=<id>`
- ramp: `#instrument-<id>`
- runway: `#model-<id>`
- tower: `#control=<id>` and `#sovereign=<id>`

### 5.3 `localStorage` coverage

Persistence is explicitly covered for:

- `cockpit-theme`
- `cockpit-last-scan`
- `cockpit-security-posture`
- `copilot-preflight`

### 5.4 Data integrity coverage

`tests\integrity.spec.js` is especially important because it validates JSON relationships without a browser:

- changelog instrument ids resolve
- wiring endpoints resolve
- related instrument ids resolve
- duplicate ids are rejected
- zone references are validated
- model ids are unique

## 6. Approximate suite size

Source inspection shows **223 declared tests** across the current spec files.

Breakdown by file:

| Spec file | Declared tests |
| --- | ---: |
| `tests\cockpit.spec.js` | 29 |
| `tests\terminal.spec.js` | 17 |
| `tests\security.spec.js` | 37 |
| `tests\jet-bridge.spec.js` | 17 |
| `tests\ramp.spec.js` | 15 |
| `tests\runway.spec.js` | 31 |
| `tests\tower.spec.js` | 25 |
| `tests\flight-log.spec.js` | 15 |
| `tests\preflight.spec.js` | 13 |
| `tests\wiring.spec.js` | 14 |
| `tests\integrity.spec.js` | 9 |

## 7. Relevant spec coverage by feature area

| Feature area | Primary specs |
| --- | --- |
| Cockpit detail blade and cross-links | `tests\cockpit.spec.js`, `tests\security.spec.js`, `tests\tower.spec.js`, `tests\runway.spec.js` |
| Global page navigation | most page-specific specs |
| Search-linked deep links | indirectly covered through page hash contracts; no dedicated search overlay spec currently |
| Security posture persistence | `tests\security.spec.js` |
| Pre-flight persistence and reset | `tests\preflight.spec.js` |
| Wiring graph topology render | `tests\wiring.spec.js` |
| Data catalog consistency | `tests\integrity.spec.js` |

## 8. Known gaps

| Gap | Impact |
| --- | --- |
| No dedicated test file for `search.js` overlay behavior | global search keyboard and ranking behavior can regress without direct coverage |
| No visual-regression or screenshot baseline tests | styling/layout regressions may pass functional tests |
| No accessibility-focused audit in current suite | ARIA and keyboard coverage are partial and feature-specific |
| No schema validator beyond custom integrity checks | malformed but syntactically valid JSON can still break runtime behavior if keys change unexpectedly |
| No deployment smoke test for Vercel headers/caching | caching regressions are not covered by Playwright |

## 9. Recommended contributor workflow

1. Run `npm test` before changing behavior.
2. Update or add the relevant JSON/catalog data.
3. Update affected page controller logic.
4. Update the corresponding `tests\*.spec.js` file.
5. Re-run `npm test`.

## 10. Assumptions and uncertainty

- **Assumption:** The declared-test count is a source-derived count of `test(...)` calls and does not include any dynamically generated tests; none were observed in current source.
