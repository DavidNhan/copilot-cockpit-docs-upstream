# Operations

## 1. Deployment model

The site is deployed as static content from the repository root.

`vercel.json` defines:

- `"framework": null`
- empty `"buildCommand"`
- `"outputDirectory": "."`

Operational consequence: deployment is file-copy style. There is no compile, bundle, or asset-generation step in the repository deployment path.

## 2. Cache behavior

`vercel.json` applies cache headers to selected asset classes:

| Path pattern | Cache policy |
| --- | --- |
| `/media/(.*)` | `public, max-age=31536000, immutable` |
| `/(.*).css` | `public, max-age=3600, must-revalidate` |
| `/(.*).js` | `public, max-age=3600, must-revalidate` |
| `/data/(.*)` | `public, max-age=3600, must-revalidate` |

Operational implication:

- JSON catalog changes may remain cached for up to one hour unless cache is bypassed or invalidated
- media assets are treated as versioned immutable content

## 3. Local operation

Because the site is static, any static server rooted at the repository root is sufficient.

The repository's existing automated-test path assumes:

```bash
python3 -m http.server 3000 --bind 127.0.0.1
```

through Playwright's `webServer` hook.

## 4. External runtime dependencies

The site is not fully self-contained at runtime. Current pages reference:

| Dependency type | Source |
| --- | --- |
| Fonts | `fonts.googleapis.com`, `fonts.gstatic.com` |
| Syntax highlighting | `cdn.jsdelivr.net` Prism assets |
| Diagrams | `cdn.jsdelivr.net` Mermaid assets |
| Analytics | `/_vercel/insights/script.js`, `/_vercel/speed-insights/script.js` |
| Video embeds | `www.youtube-nocookie.com` if instrument/video data is present |

Operational implication: the core HTML and JSON are local/static, but some presentation and analytics features depend on third-party or platform-hosted scripts.

## 5. Content update operations

### 5.1 Standard content updates

Most content updates are direct edits to:

- `data\copilot-instruments.json`
- `data\copilot-models.json`
- `data\governance-controls.json`
- `data\security-threats.json`
- `data\security-frameworks.json`
- `data\sovereign-cloud.json`
- page-guide JSON files

### 5.2 Model-catalog enrichment workflow

`tools\enrich\` is the operational path for refreshing `data\copilot-models.json`.

Workflow intent:

1. `harvest.py` fetches upstream provider/docs data into `tools\cache\`
2. `normalize.py` produces a merged candidate
3. a human reviews and manually applies final catalog changes

The tooling explicitly documents that it should not auto-write to `data\copilot-models.json`.

## 6. Failure modes

| Failure mode | User-visible result | Primary recovery action |
| --- | --- | --- |
| Missing or broken JSON file | page container replaced with `DATA LINK LOST` message | restore valid JSON and redeploy |
| Broken cross-file ids | missing callouts, broken links, incomplete graph/search output | run integrity tests and repair ids |
| CDN script outage | missing diagrams, missing syntax highlighting, or degraded fonts | restore upstream dependency or vendor locally |
| Stale cached JSON | deployment serves old catalog values for up to one hour | invalidate cache or wait for TTL |
| Invalid `localStorage` payloads | page-specific persistence issues | clear affected storage keys |

## 7. Key operational risks

### 7.1 Data verification risk

`data\copilot-models.json` and `data\security-frameworks.json` explicitly mark unresolved verification work. Treat those files as partially verified data sources.

### 7.2 Static error handling risk

There is no central retry, logging backend, or feature-flag layer in the repository. A malformed JSON file can break a page immediately at runtime.

### 7.3 Browser-state drift

Deep-link ids and persisted `localStorage` ids depend on current catalog keys. Renaming ids without compatibility handling can break:

- existing bookmarks
- search result links
- stored scanner/checklist state
- cockpit callouts

### 7.4 External dependency risk

Mermaid, Prism, fonts, Vercel analytics, and video embeds are external dependencies. The application remains static, but not fully offline-capable.

## 8. Deployment checklist

1. Validate edited JSON for syntax and cross-file ids.
2. Run Playwright suite.
3. Review `verificationRequired` flags in updated catalogs.
4. Confirm that hash-based links still resolve:
   - cockpit instrument links
   - scanner links
   - runway model links
   - tower control/sovereign links
5. Deploy static files.
6. Verify live caching behavior for changed `.json`, `.js`, and `.css` files.

## 9. Assumptions and uncertainty

- **Assumption:** Vercel is the primary deployment target because `vercel.json` is present and complete.
- **Assumption:** No additional CI/CD deployment logic outside the repository is required to serve the site correctly.
