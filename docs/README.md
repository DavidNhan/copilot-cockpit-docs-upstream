# Copilot Cockpit Documentation

This repository is a static, multi-page documentation site implemented with root-level HTML files, shared browser scripts, and JSON catalogs under `data\`. There is no build step. Pages fetch JSON directly at runtime and render client-side in the browser.

## Repository summary

| Surface | Path(s) | Notes |
| --- | --- | --- |
| Entry pages | `index.html`, `terminal.html`, `security.html`, `jet-bridge.html`, `ramp.html`, `runway.html`, `tower.html`, `flight-log.html`, `preflight.html`, `wiring.html` | 10 root HTML pages |
| Shared runtime | `app.js`, `search.js`, `styles.css` | `app.js` drives the cockpit page, `search.js` adds global search, `styles.css` styles all pages |
| Data catalogs | `data\*.json` | 11 JSON files; pages fetch these directly |
| Tests | `tests\*.spec.js`, `playwright.config.js` | Playwright end-to-end suite plus JSON integrity checks |
| Deployment | `vercel.json` | Static output from repository root; no build command |
| Optional tooling | `tools\enrich\*` | Human-gated model-catalog enrichment pipeline |

## Current content inventory

| Catalog | Current count | Source file |
| --- | ---: | --- |
| Cockpit instruments | 46 | `data\copilot-instruments.json` |
| Cockpit zones | 8 | `data\copilot-instruments.json` |
| Models | 21 | `data\copilot-models.json` |
| Governance controls | 20 | `data\governance-controls.json` |
| Security threats | 23 | `data\security-threats.json` |
| Changelog entries | 35 | `data\known-changelog-entries.json` |
| Wiring connections | 55 | `data\wiring-diagram.json` |
| Pre-flight checklist items | 22 | `data\preflight-checklist.json` |

## Runtime model

1. The browser loads a root HTML page.
2. The page script fetches one or more JSON files from `data\`.
3. The script renders DOM sections from the fetched data.
4. Shared behaviors apply across pages:
   - theme persistence through `localStorage['cockpit-theme']`
   - cross-page search through `search.js`
   - deep links through URL hash fragments such as `#instrument-...`, `#scan=...`, `#model-...`

## Documentation map

| File | Purpose |
| --- | --- |
| `README.md` | Entry point for this documentation set |
| `ARCHITECTURE.md` | Page/component architecture, runtime topology, and data flow |
| `API-REFERENCE.md` | Internal browser-facing contracts: JSON endpoints, hash fragments, and `localStorage` keys |
| `DATA-CATALOG.md` | Catalog-by-catalog schema and cross-file relationships |
| `TESTING-GUIDE.md` | Test stack, execution model, and spec coverage |
| `OPERATIONS.md` | Local operation, deployment, dependencies, and operational risks |
| `CONTRIBUTING.md` | Change workflow and repository-specific contribution guidance |

## Local execution

Use any static file server rooted at the repository root. Existing test configuration uses Playwright with a local Python web server on port `3000`.

```bash
npm test
```

That command maps to `npx playwright test` and relies on the Playwright `webServer` defined in `playwright.config.js`.

## Assumptions and uncertainty

- **Assumption:** This documentation describes the repository source as present during this pass, not the live deployment.
- `data\copilot-models.json` and `data\security-frameworks.json` include explicit verification flags and notes; treat those catalog claims as partially verified until the files' own `verificationRequired` flags are false.
- Automated tests could not be executed in the documentation-generation environment because `pwsh.exe` was not installed. The test design and coverage documented here come from repository source, not from a successful local run in this environment.
