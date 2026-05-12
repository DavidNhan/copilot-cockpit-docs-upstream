# API Reference

This repository does not expose a server-side API. The operative "API" is the set of browser-facing contracts used by the static pages:

1. JSON files under `data\`
2. URL hash fragments
3. `localStorage` keys
4. page-level rendering/error conventions

## 1. JSON resource endpoints

All fetches are relative to the repository root.

| Resource | Consumed by | Purpose |
| --- | --- | --- |
| `data\copilot-instruments.json` | `app.js`, `security.html`, `ramp.html`, `search.js`, `wiring.html` | Master instrument, zone, and plan catalog |
| `data\copilot-models.json` | `app.js`, `runway.html`, `tower.html`, `search.js` | Model catalog, surfaces, plans, NOTAMs, flight plans, engine summary |
| `data\governance-controls.json` | `app.js`, `tower.html`, `search.js` | Governance controls and framework mappings |
| `data\security-threats.json` | `app.js`, `security.html` | Threat records keyed by `instrumentId` |
| `data\security-frameworks.json` | `security.html` | OWASP/ATLAS/CWE registry used to resolve framework ids to links |
| `data\sovereign-cloud.json` | `tower.html` | Sovereignty pillars, deployment options, provider strategies, residual risks |
| `data\terminal-guide.json` | `terminal.html` | Terminal page content |
| `data\jet-bridge-guide.json` | `jet-bridge.html` | Jet Bridge tutorial content |
| `data\preflight-checklist.json` | `preflight.html` | Checklist categories and items |
| `data\known-changelog-entries.json` | `flight-log.html`, `search.js` | Changelog timeline and search entries |
| `data\wiring-diagram.json` | `wiring.html` | Connection graph, legend, zone descriptions |

## 2. Resource shape summary

### 2.1 `data\copilot-instruments.json`

Top-level keys:

- `version`
- `lastUpdated`
- `zones[]`
- `plans[]`
- `instruments[]`

Important contracts:

- `zones[].id` is used as a rendering and CSS-routing key
- `instruments[].id` is the primary cross-file foreign key
- `instruments[].zone` must match a zone id
- `instruments[].planAvailability` uses booleans and `"limited"`
- `instruments[].perspectives` drives page filtering such as Ramp

### 2.2 `data\copilot-models.json`

Top-level keys in current source include:

- `$schema`
- `version`
- `lastUpdated`
- `verificationRequired`
- `verificationNotes`
- `sources[]`
- `capabilities[]`
- `surfaces[]`
- `plans[]`
- `models[]`

Additional page-specific structures used by `runway.html` and `tower.html`:

- `notams[]`
- `copilotEngine`
- `flightPlans[]`

### 2.3 `data\governance-controls.json`

Top-level keys:

- `version`
- `lastUpdated`
- `verificationRequired`
- `description`
- `sources`
- `controls[]`

Control ids also participate in:

- cockpit governance callouts
- tower deep links
- wiring graph endpoints
- search results

### 2.4 `data\security-threats.json`

Top-level keys:

- `version`
- `lastUpdated`
- `description`
- `schema`
- `threats[]`

Each `threats[]` record uses:

- `instrumentId`
- `classification`
- `cia[]`
- `threatModel`
- `scenario`
- `demo`
- `countermeasures[]`
- `blastRadius`
- `mitigatesCWE[]`
- `frameworks`

## 3. URL hash contracts

### 3.1 Cockpit

**Format:** `index.html#instrument-<instrumentId>`

Behavior:

- `app.js` opens the detail blade for the resolved instrument id
- opening a blade pushes the hash
- closing the blade clears the hash back to the path
- browser back/forward is handled through `history.pushState` and `popstate`

### 3.2 Security

**Format:** `security.html#scan=<instrumentId>`

Behavior:

- scanner initializes to hash value if present
- otherwise it falls back to `localStorage['cockpit-last-scan']`
- otherwise it falls back to the first threat record
- changing scan updates the hash with `history.replaceState`

### 3.3 Ramp

**Format:** `ramp.html#instrument-<instrumentId>`

Behavior:

- if the id is present in the ramp-filtered subset of instruments, the ramp blade opens
- closing the blade clears the hash back to the path

### 3.4 Runway

**Format:** `runway.html#model-<modelId>`

Behavior:

- model blade opens from the hash on load
- opening a model writes the hash with `history.replaceState`
- closing the blade clears the hash

### 3.5 Tower

**Formats:**

- `tower.html#control=<controlId>`
- `tower.html#sovereign=<deploymentOptionId>`

Behavior:

- matching rows/options receive a `highlight` class
- matching elements are scrolled into view
- no persistent selection state is stored outside the hash

## 4. `localStorage` contracts

### 4.1 Shared theme key

| Key | Value type | Used by |
| --- | --- | --- |
| `cockpit-theme` | string | All page controllers that implement theme toggle |

Allowed values:

- `light`
- `dark`

Behavior:

- if absent, pages default to dark theme
- pages that use Mermaid re-render diagrams when the theme changes

### 4.2 Security page keys

| Key | Value type | Example | Purpose |
| --- | --- | --- | --- |
| `cockpit-last-scan` | string | `mcp` | restore last scanned control |
| `cockpit-security-posture` | JSON object | `{"audit-logs":true,"byok":false}` | persist posture checklist |

`cockpit-security-posture` is parsed with `JSON.parse(localStorage.getItem(...) || '{}')`; malformed JSON would throw unless corrected in storage.

### 4.3 Pre-Flight key

| Key | Value type | Example | Purpose |
| --- | --- | --- | --- |
| `copilot-preflight` | JSON object | `{"github-account":true,"plan-selected":true}` | persist checklist completion |

`preflight.html` wraps load in a `try/catch` and falls back to `{}` if stored JSON is invalid.

## 5. Search contracts

`search.js` builds in-memory entries with these output targets:

| Item type | URL pattern |
| --- | --- |
| instrument | `index.html#instrument-<id>` |
| control | `tower.html#control=<id>` |
| model | `runway.html#model-<id>` |
| changelog | `flight-log.html` |

Keyboard contract:

- `Ctrl+K` / `Cmd+K`: open or close global search overlay
- `Escape`: close overlay
- Arrow keys: move selection
- `Enter`: navigate to selected result

Cockpit-local search contract in `app.js`:

- `/` focuses the cockpit filter input when an input element is not already focused
- `Ctrl+K` also focuses the cockpit search input on the cockpit page

## 6. Rendering and error contracts

Common behavior across page controllers:

- fetch failure replaces the main content container with a `DATA LINK LOST` message
- HTML inserted from JSON values is escaped before rendering
- Mermaid rendering is lazy or post-render, depending on page

Known exception:

- `wiring.html` and `tower.html` use Mermaid with `securityLevel: 'loose'`

## 7. Cross-file foreign-key contracts

| Foreign key | Source file | Target file |
| --- | --- | --- |
| `instrumentId` | `data\security-threats.json` | `data\copilot-instruments.json` |
| `controls[].id` links | `data\governance-controls.json` | cockpit/tower/search/wiring consumers |
| `connections[].from` / `connections[].to` | `data\wiring-diagram.json` | instrument ids and governance control ids |
| `entries[].instruments[]` | `data\known-changelog-entries.json` | `data\copilot-instruments.json` |
| `relatedInstruments[]` | `data\copilot-instruments.json` | `data\copilot-instruments.json` |

These relationships are enforced by `tests\integrity.spec.js`.

## 8. Assumptions and uncertainty

- **Assumption:** Relative JSON paths are stable because the site is always hosted from the repository root.
- **Assumption:** There is no public compatibility promise for these contracts outside the repository itself; treat them as internal implementation interfaces.
