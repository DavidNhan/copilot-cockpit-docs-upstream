# Copilot Cockpit documentation set

This documentation describes the repository as it exists in source: a static, client-rendered site with 10 root HTML pages, shared runtime assets in `app.js`, `search.js`, and `styles.css`, and JSON catalogs under `data/`. There is no build step and no server-side application in this repository.

## Quick facts

| Area | Current state |
| --- | --- |
| Site shape | 10 root HTML pages: 7 core perspectives plus Flight Log, Pre-Flight, and Wiring |
| Shared runtime | `app.js` for the cockpit page, `search.js` for global command-palette search, `styles.css` for shared styling and theme tokens |
| Data model | 11 JSON catalogs under `data/`, fetched directly by the browser |
| Primary state contracts | URL hashes and `localStorage` |
| Test stack | Playwright only, 222 declared tests across 11 spec files |
| Deployment shape | Static root deployment via `vercel.json`; no build command |

## Current catalog inventory

| Catalog | Count | Notes |
| --- | ---: | --- |
| Instruments | 46 | Master content source in `data/copilot-instruments.json` |
| Zones | 8 | Defined alongside instruments |
| Plans | 5 | Free, Pro, Pro+, Business, Enterprise |
| Models | 21 | `data/copilot-models.json`; file explicitly marks itself partially verified |
| Governance controls | 20 | `data/governance-controls.json` |
| Security threats | 22 | `data/security-threats.json` |
| Changelog entries | 35 | `data/known-changelog-entries.json` |
| Wiring connections | 55 | `data/wiring-diagram.json` |
| Pre-flight categories | 6 | 22 checklist items total |

## What matters operationally

1. Each page loads one or more JSON files at runtime and renders client-side.
2. Cross-page navigation depends on stable IDs such as instrument IDs, control IDs, and model IDs.
3. Persistent browser state is intentionally small:
   - `cockpit-theme`
   - `cockpit-last-scan`
   - `cockpit-security-posture`
   - `copilot-preflight`
4. Deep links are part of the functional contract, not just convenience:
   - `#instrument-<id>`
   - `#scan=<id>`
   - `#model-<id>`
   - `#control=<id>`
   - `#sovereign=<id>`

## Documentation map

| File | Purpose |
| --- | --- |
| `README.md` | Entry point and high-level operating model |
| `ARCHITECTURE.md` | Page topology, runtime responsibilities, and Mermaid system diagram |
| `API-REFERENCE.md` | Browser-facing contracts: fetches, hashes, `localStorage`, and page behavior |
| `DATA-CATALOG.md` | JSON file schemas, foreign keys, verification flags, and maintenance notes |
| `TESTING-GUIDE.md` | Playwright setup, coverage scope, and known gaps |
| `OPERATIONS.md` | Deployment, caching, external dependencies, failure modes, and risks |
| `CONTRIBUTING.md` | Change workflow and repository-specific guardrails |

## Read this set in order

1. Start with `ARCHITECTURE.md` to understand how pages, shared runtime files, and catalogs fit together.
2. Use `API-REFERENCE.md` when changing behavior, IDs, hashes, or `localStorage`.
3. Use `DATA-CATALOG.md` when editing content under `data/`.
4. Use `TESTING-GUIDE.md` and `OPERATIONS.md` before shipping.

## Explicit uncertainty

- `data/copilot-models.json` sets `verificationRequired: true` and includes `verificationNotes`; treat model-specific claims as partially verified unless that flag is cleared.
- This repository contains Vercel configuration and Vercel Insights scripts, so these docs describe a Vercel-compatible deployment path. If another host is used, `OPERATIONS.md` still applies for the static-site behavior, but platform-specific observability or cache behavior may differ.
