# Contributing

## 1. Change surfaces

Most repository changes fall into one of four categories:

| Change type | Primary files |
| --- | --- |
| Content/catalog update | `data\*.json` |
| Page behavior update | `index.html`, page-specific `*.html`, `app.js`, `search.js` |
| Shared styling update | `styles.css` |
| Test update | `tests\*.spec.js`, optionally `playwright.config.js` |

## 2. Repository-specific rules of thumb

### 2.1 Prefer editing JSON for content changes

If text appears on a page and is already sourced from a JSON file, update the JSON source rather than hard-coding markup.

Examples:

- Terminal text -> `data\terminal-guide.json`
- Jet Bridge text -> `data\jet-bridge-guide.json`
- Pre-flight items -> `data\preflight-checklist.json`
- Changelog entries -> `data\known-changelog-entries.json`

### 2.2 Preserve stable ids

Do not rename ids casually:

- instrument ids drive deep links, search, threat joins, wiring endpoints, related links, and changelog references
- control ids drive Tower deep links, cockpit governance callouts, search, and wiring
- model ids drive Runway deep links and cockpit engine-zone links

If an id must change, update all dependent files and links in the same change.

### 2.3 Keep page behavior aligned with data shape

Browser code expects exact key names. When changing schema-like JSON keys, update:

1. the page controller consuming the key
2. any search/indexing logic using the key
3. integrity and page specs that assume the key

## 3. Change workflow

1. Identify the affected page(s) and data file(s).
2. Edit the relevant JSON and/or page script.
3. Update related deep links or `localStorage` handling if ids changed.
4. Update the relevant Playwright spec file.
5. Run `npm test`.

## 4. File mapping for common tasks

| Task | Files to inspect first |
| --- | --- |
| Add or edit a cockpit instrument | `data\copilot-instruments.json`, `app.js`, `tests\cockpit.spec.js`, optionally `tests\integrity.spec.js` |
| Add or edit a model | `data\copilot-models.json`, `runway.html`, `app.js`, `tower.html`, `tests\runway.spec.js`, `tests\integrity.spec.js` |
| Add or edit a governance control | `data\governance-controls.json`, `tower.html`, `app.js`, `search.js`, `tests\tower.spec.js`, `tests\integrity.spec.js` |
| Add or edit a threat | `data\security-threats.json`, `data\security-frameworks.json`, `security.html`, `tests\security.spec.js` |
| Add or edit a wiring relationship | `data\wiring-diagram.json`, `wiring.html`, `tests\wiring.spec.js`, `tests\integrity.spec.js` |
| Add or edit a pre-flight item | `data\preflight-checklist.json`, `preflight.html`, `tests\preflight.spec.js` |

## 5. Deep-link and persistence checklist

When changing ids or page contracts, verify all affected links and storage:

| Contract | Files commonly affected |
| --- | --- |
| `#instrument-<id>` | `app.js`, `ramp.html`, `flight-log.html`, `wiring.html`, `search.js`, tests |
| `#scan=<id>` | `security.html`, cockpit security callouts, tests |
| `#model-<id>` | `runway.html`, `app.js`, `search.js`, tests |
| `#control=<id>` / `#sovereign=<id>` | `tower.html`, `app.js`, `search.js`, tests |
| `localStorage['cockpit-theme']` | all page controllers with theme toggles |
| `localStorage['cockpit-last-scan']` | `security.html`, `tests\security.spec.js` |
| `localStorage['cockpit-security-posture']` | `security.html`, `tests\security.spec.js` |
| `localStorage['copilot-preflight']` | `preflight.html`, `tests\preflight.spec.js` |

## 6. Working with the enrichment tooling

For model-catalog maintenance:

- read `tools\enrich\README.md` first
- add/adjust sources in `tools\enrich\sources.yml`
- add adapters under `tools\enrich\adapters\`
- use `harvest.py` and `normalize.py` to create reviewed candidate data

Do not introduce direct writes from the enrichment pipeline into `data\copilot-models.json` unless you intentionally change the documented human-gated workflow.

## 7. Documentation and tests

Update documentation when you change:

- page architecture or data flow
- deep-link or `localStorage` contracts
- deployment behavior (`vercel.json`)
- test tooling or coverage model

Minimum test expectation for code changes:

- update the page-specific spec
- keep `tests\integrity.spec.js` passing when you change shared ids or relationships

## 8. Known contribution risks

| Risk | How to avoid it |
| --- | --- |
| Breaking hidden cross-file ids | search for the id across `data\`, page scripts, and `tests\` before renaming |
| Updating JSON without tests | pair content/behavior changes with spec updates |
| Assuming model/framework catalogs are fully verified | respect the files' verification flags and notes |
| Editing page text in HTML when JSON is authoritative | update the catalog source instead |

## 9. Assumptions and uncertainty

- **Assumption:** Contributors are expected to work within the current static-site model rather than introducing a framework or build system for routine changes.
