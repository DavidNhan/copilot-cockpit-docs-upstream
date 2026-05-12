# Contributing

## Working principles

1. Prefer updating JSON catalogs over hard-coding page text.
2. Treat IDs as stable contracts.
3. Keep page behavior, hashes, and `localStorage` in sync with data changes.
4. Update tests when behavior or data contracts change.
5. Preserve explicit uncertainty when a catalog marks itself partially verified.

## Where changes usually belong

| Change type | Primary files |
| --- | --- |
| Content update | `data/*.json` |
| Cockpit behavior | `app.js`, `index.html`, `tests/cockpit.spec.js` |
| Global search behavior | `search.js`, affected page specs |
| Non-cockpit page behavior | page-local HTML script plus that page's spec |
| Shared styling | `styles.css` |
| Deployment behavior | `vercel.json`, `OPERATIONS.md` |

## Stable contract checklist

Before renaming an ID, check every dependent surface:

| ID type | Common dependents |
| --- | --- |
| Instrument ID | cockpit deep links, ramp deep links, security threats, Wiring graph, changelog links, search index, related instruments, stored scan state |
| Control ID | tower deep links, cockpit governance callouts, Wiring graph, search index |
| Model ID | runway deep links, NOTAM links, tower flight plans, cockpit engine-zone links, search index |

If an ID must change, update all references in the same change.

## Change recipes

### Add or update an instrument

1. Edit `data/copilot-instruments.json`.
2. Check whether the change affects:
   - cockpit rendering in `app.js`
   - ramp filtering by `perspectives`
   - security joins by `instrumentId`
   - Wiring graph labels or links
3. Update `tests/cockpit.spec.js` and `tests/integrity.spec.js` if needed.

### Add or update a governance control

1. Edit `data/governance-controls.json`.
2. Verify Tower rendering and cockpit governance callouts still make sense.
3. If the control is meant to map to a cockpit instrument, ensure the IDs intentionally align.
4. Update `tests/tower.spec.js` and, if relationships changed, `tests/integrity.spec.js`.

### Add or update a model

1. Edit `data/copilot-models.json`.
2. Verify Runway and Tower consumers:
   - model blade
   - NOTAM links
   - flight plans
   - cockpit engine-zone links
3. Preserve `verificationRequired` and `verificationNotes` if the data is still partial.
4. Update `tests/runway.spec.js`; consider integrity coverage if you add new ID relationships.

### Add or update a threat

1. Edit `data/security-threats.json`.
2. Confirm `instrumentId` resolves to an instrument.
3. If you add framework references, verify matching entries exist in `data/security-frameworks.json`.
4. Update `tests/security.spec.js`.

### Add or update guide content

| Page | Source catalog |
| --- | --- |
| Terminal | `data/terminal-guide.json` |
| Jet Bridge | `data/jet-bridge-guide.json` |
| Pre-Flight | `data/preflight-checklist.json` |
| Flight Log | `data/known-changelog-entries.json` |
| Wiring | `data/wiring-diagram.json` |
| Tower sovereignty | `data/sovereign-cloud.json` |

Prefer editing the catalog rather than the HTML markup when content already comes from data.

## Hash and storage contracts to preserve

| Contract | Files commonly affected |
| --- | --- |
| `#instrument-<id>` | `app.js`, `ramp.html`, `flight-log.html`, `wiring.html`, `search.js`, tests |
| `#scan=<id>` | `security.html`, cockpit security callouts, tests |
| `#model-<id>` | `runway.html`, `search.js`, tests |
| `#control=<id>` and `#sovereign=<id>` | `tower.html`, cockpit governance callouts, `search.js`, tests |
| `cockpit-theme` | almost every page controller |
| `cockpit-last-scan` | `security.html`, `tests/security.spec.js` |
| `cockpit-security-posture` | `security.html`, `tests/security.spec.js` |
| `copilot-preflight` | `preflight.html`, `tests/preflight.spec.js` |

## Maintainability notes

| Current pattern | Contribution implication |
| --- | --- |
| Theme logic is duplicated across pages | if theme behavior changes, update multiple HTML files, not just one helper |
| Most pages use inline scripts | behavior changes live beside markup, so page diffs can be large |
| Search indexes only instruments, controls, models, and changelog entries | adding a new searchable content type requires `search.js` work, not only data changes |
| Integrity checks cover major joins but not every relationship | do not assume new model or framework references are automatically validated |

## Minimum validation expectations

1. Run `npm test`.
2. Manually verify the relevant hash route if you changed any ID-based navigation.
3. Confirm affected `localStorage` behavior if you changed persistence.
4. Update these docs when architecture, contracts, testing scope, or operations materially change.

## Explicit assumptions

- Contributors are expected to work within the existing static-site model unless a larger architectural change is intentionally being made.
- `tools/enrich/` remains a human-gated workflow for model data unless the repository explicitly changes that rule.
