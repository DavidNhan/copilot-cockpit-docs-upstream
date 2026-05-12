# Operations

## Deployment model

The repository is deployable as static files directly from the repository root.

`vercel.json` currently sets:

- `"framework": null`
- `"buildCommand": ""`
- `"outputDirectory": "."`

That means deployment is file-serving only. There is no compile, bundling, or asset-generation step in the repository itself.

## Cache behavior

`vercel.json` applies these headers:

| Path pattern | Cache-Control |
| --- | --- |
| `/media/(.*)` | `public, max-age=31536000, immutable` |
| `/(.*).css` | `public, max-age=3600, must-revalidate` |
| `/(.*).js` | `public, max-age=3600, must-revalidate` |
| `/data/(.*)` | `public, max-age=3600, must-revalidate` |

Operational consequences:

- JSON, JS, and CSS changes can remain stale for up to 1 hour.
- Media files are treated as versioned immutable assets.
- A data-only hotfix may appear delayed to end users unless cache is bypassed or invalidated.

## Local operation

Any static server rooted at the repository root will work.

The repository's existing test path expects:

```bash
python3 -m http.server 3000 --bind 127.0.0.1
```

Playwright starts that server automatically through `playwright.config.js`.

## Runtime dependencies outside the repo

| Dependency | Source | If unavailable |
| --- | --- | --- |
| Fonts | Google Fonts | visual fallback only |
| Syntax highlighting | jsDelivr Prism assets | cockpit code tab loses highlighting |
| Diagram rendering | jsDelivr Mermaid asset | diagram sections fail or degrade |
| Analytics | `/_vercel/insights/script.js`, `/_vercel/speed-insights/script.js` | no impact on core content |
| Embedded video | `youtube-nocookie.com` | media embeds fail |

The application is static, but not fully self-contained or offline-complete.

## Runtime operational responsibilities

| Surface | Responsibility |
| --- | --- |
| `data/*.json` | keep keys valid, IDs stable, and cross-file references intact |
| page controllers | keep theme behavior, hash routing, and rendering aligned with catalog shapes |
| `search.js` | update when introducing new searchable content types or changing deep-link shapes |
| tests | keep page specs and `tests/integrity.spec.js` aligned with changed contracts |

## Failure modes and recovery

| Failure mode | Typical symptom | Recovery |
| --- | --- | --- |
| primary JSON fetch fails | page shows `DATA LINK LOST` or breaks during boot | restore valid JSON, redeploy, and re-test |
| cross-file ID drift | missing callouts, broken Wiring graph links, broken deep links | search for the changed ID, repair references, run integrity tests |
| Mermaid CDN failure | blank or unrendered diagrams | restore CDN access or vendor Mermaid locally |
| stale cache | page shows old models, controls, or changelog data | wait for TTL or invalidate cache |
| malformed `cockpit-security-posture` storage | Security page posture section can throw during parse | clear the key in browser storage |
| malformed `copilot-preflight` storage | checklist resets to empty state | clear or rewrite the key; page already falls back safely |

## Known operational risks

| Risk | Impact | Why it exists |
| --- | --- | --- |
| Duplicated theme logic | medium maintainability risk | most pages implement their own theme boot and toggle inline |
| Partial model-catalog verification | medium content trust risk | `data/copilot-models.json` explicitly warns that parts of the file are still partial |
| Minimal runtime schema enforcement | medium runtime risk | browser code assumes expected keys rather than validating full schemas |
| Security page storage parsing is brittle | medium UX risk | `cockpit-security-posture` parsing is not wrapped in `try/catch` |
| External CDN dependence | medium availability risk | Mermaid, Prism, and fonts are not local assets |
| Search overlay lacks direct automated coverage | medium regression risk | core pages are tested, but `search.js` itself is not deeply exercised |

## Operational checklist for content or behavior changes

1. Validate JSON syntax.
2. Check whether any ID changed.
3. Run `npm test`.
4. Manually verify the affected hash routes.
5. Review any file with `verificationRequired: true`.
6. Deploy static files.
7. Spot-check cache freshness for changed `.json`, `.js`, and `.css` assets.

## Explicit assumptions

- Vercel is the primary supported host because the repository contains a concrete `vercel.json` and Vercel Insights scripts.
- No CI workflow is present in the repository root `.github/workflows` path, so any production deployment automation likely lives outside this source tree or is managed manually.
