# Data Catalog

## 1. Catalog overview

The site is data-driven. Browser code renders most content from `data\*.json` instead of hard-coding text into page templates.

| File | Current size of main collection | Consumed by | Verification flag |
| --- | ---: | --- | --- |
| `data\copilot-instruments.json` | 46 instruments | `app.js`, `security.html`, `ramp.html`, `search.js`, `wiring.html` | none |
| `data\copilot-models.json` | 21 models | `app.js`, `runway.html`, `tower.html`, `search.js` | `verificationRequired: true` |
| `data\governance-controls.json` | 20 controls | `app.js`, `tower.html`, `search.js` | `verificationRequired: false` |
| `data\security-threats.json` | 23 threats | `app.js`, `security.html` | none |
| `data\security-frameworks.json` | framework registries | `security.html` | `verificationRequired: true` |
| `data\sovereign-cloud.json` | 6 deployment options, 3 residual risks | `tower.html` | none |
| `data\terminal-guide.json` | 4 page sections | `terminal.html` | none |
| `data\jet-bridge-guide.json` | 5 page sections | `jet-bridge.html` | none |
| `data\preflight-checklist.json` | 6 categories, 22 items | `preflight.html` | none |
| `data\known-changelog-entries.json` | 35 entries | `flight-log.html`, `search.js` | none |
| `data\wiring-diagram.json` | 55 connections, 4 types | `wiring.html` | none |

## 2. File-by-file schema notes

### 2.1 `data\copilot-instruments.json`

Purpose:

- master content source for cockpit instruments
- source of zones and plans
- source of cross-page ids

Top-level keys:

- `version`
- `lastUpdated`
- `zones[]`
- `plans[]`
- `instruments[]`

Important nested structures inside `instruments[]`:

- identity: `id`, `symbol`, `name`, `zone`, `perspectives`, `status`
- availability: `planAvailability`, `ideSupport`, `flightMode`
- content: `description`, `shortDescription`, `capabilities`, `squawkCodes`, `proTip`
- relationships: `relatedInstruments`
- rich media: `mermaidDiagrams`, `codeExamples`, `terminalRecordings`, `videos`
- references: `links`
- quality markers: `lastVerified`, `confidenceScore`

Consumers:

- `app.js` renders cards, detail tabs, plan/IDE rows, related links, and cockpit callouts
- `ramp.html` filters on `perspectives.includes('ramp')`
- `security.html` joins `threat.instrumentId` to instrument metadata
- `wiring.html` uses zone and instrument metadata for graph labels and zone summaries
- `search.js` indexes instruments for global search

### 2.2 `data\copilot-models.json`

Purpose:

- authoritative model fleet for Runway
- derived engine-zone content for the cockpit page
- Tower flight plans
- search indexing

Top-level keys currently used by code:

- metadata: `$schema`, `version`, `lastUpdated`, `verificationRequired`, `verificationNotes`, `description`
- reference lists: `sources[]`, `capabilities[]`, `surfaces[]`, `plans[]`
- main content: `models[]`
- runway/tower content: `notams[]`, `copilotEngine`, `flightPlans[]`

Important nested structures inside `models[]`:

- identity: `id`, `displayName`, `provider`, `family`, `status`
- availability: `surfaceAvailability`, `planAvailability`, `modeAvailability`, `ideAvailability`
- evaluation: `strengths`, `limitations`, `capabilities`, `taskFit`, `taskArea`, `excelsAt`
- metadata: `contextWindow`, `releasedAt`, `knowledgeCutoff`, `pricingMultiplier`
- quality markers: `confidenceScore`, `lastVerified`, `verificationRequired`

Operational note:

- `app.js` derives cockpit engine-zone links from non-deprecated models and maps them into `_engineModels`
- `runway.html` treats `verificationRequired` as a UI banner condition

### 2.3 `data\governance-controls.json`

Purpose:

- governance registry for Tower
- search index for controls
- cockpit governance callout source
- wiring graph endpoints

Top-level keys:

- `version`
- `lastUpdated`
- `verificationRequired`
- `description`
- `sources`
- `controls[]`

Important keys in `controls[]`:

- `id`
- `category`
- `scope`
- `defaultState`
- `rolloutEffort`
- `governanceNote`
- `complianceRelevance[]`

### 2.4 `data\security-threats.json`

Purpose:

- scanner content for Security page
- cockpit security callout index

Top-level keys:

- `version`
- `lastUpdated`
- `description`
- `schema`
- `threats[]`

Each threat record binds to `copilot-instruments.json` by `instrumentId`.

Important fields:

- classification metadata: `classification`, `cia[]`, `blastRadius`
- scenario content: `threatModel`, `scenario`, `demo`, `countermeasures[]`
- compliance/security links: `mitigatesCWE[]`, `frameworks`

### 2.5 `data\security-frameworks.json`

Purpose:

- lookup registry for framework badges on the Security page

Top-level keys:

- `version`
- `lastUpdated`
- `verificationRequired`
- `description`
- `sources`
- `frameworks`
- `cwePattern`

Important behavior:

- Security page resolves `frameworks.owaspLLM[...]` and `frameworks.atlas[...]`
- CWE links are generated through `cwePattern.urlTemplate`

### 2.6 `data\sovereign-cloud.json`

Purpose:

- Tower sovereignty section

Important top-level arrays/objects:

- `sovereignPillars[]`
- `deploymentOptions[]`
- `providerStrategies[]`
- `residualRisks[]`
- `dataFlowDiagram`

`deploymentOptions[]` contains structured `dataFlows` objects for:

- `inMotion`
- `inUse`
- `atRest`

### 2.7 `data\terminal-guide.json`

Purpose:

- content source for `terminal.html`

Top-level page sections:

- `checkIn`
- `boardingPass`
- `firstFlight`
- `departures`

### 2.8 `data\jet-bridge-guide.json`

Purpose:

- content source for `jet-bridge.html`

Top-level page sections:

- `promptCraft`
- `contextManagement`
- `editMode`
- `agentPatterns`
- `nextSteps`

### 2.9 `data\preflight-checklist.json`

Purpose:

- checklist content and routing metadata for `preflight.html`

Top-level keys:

- `version`
- `lastUpdated`
- `description`
- `intro`
- `categories[]`

Each category includes:

- `id`
- `title`
- `icon`
- `perspective` (HTML path)
- `items[]`

Each item includes:

- `id`
- `label`
- `detail`

### 2.10 `data\known-changelog-entries.json`

Purpose:

- flight-log timeline
- search index for changelog records

Top-level keys:

- `version`
- `lastUpdated`
- `entryTypes`
- `entries[]`

Each entry includes:

- `id`
- `date`
- `type`
- `title`
- `description`
- `instruments[]`
- `zone`
- `source`

### 2.11 `data\wiring-diagram.json`

Purpose:

- graph and legend source for `wiring.html`

Top-level keys:

- `version`
- `lastUpdated`
- `description`
- `intro`
- `connectionTypes[]`
- `connections[]`
- `zoneDescriptions`

`connections[]` records use:

- `from`
- `to`
- `type`
- `label`

`from` and `to` may reference either instrument ids or governance control ids.

## 3. Cross-file relationships

| Relationship | Meaning |
| --- | --- |
| `security-threats.instrumentId -> copilot-instruments.instruments.id` | threat records resolve to instrument metadata |
| `known-changelog-entries.entries[].instruments[] -> copilot-instruments.instruments.id` | changelog entries link back to cockpit instruments |
| `copilot-instruments.relatedInstruments[] -> copilot-instruments.instruments.id` | cockpit related-link graph |
| `wiring-diagram.connections[].from/to -> instruments.id or governance-controls.id` | mixed graph of product features and admin controls |

## 4. Validation coverage

`tests\integrity.spec.js` validates:

- duplicate instrument ids
- duplicate model ids
- required instrument fields
- valid zone references
- valid changelog entry types
- valid wiring connection types
- cross-file foreign-key integrity

This is the repository's strongest automated guard against data drift.

## 5. Data maintenance workflow

Model data has an additional optional maintenance path:

- `tools\enrich\harvest.py` caches upstream payloads under `tools\cache\`
- `tools\enrich\normalize.py` produces a merged candidate catalog
- final updates are expected to be applied manually

No equivalent enrichment pipeline exists in the repository for the other JSON catalogs.

## 6. Assumptions and uncertainty

- **Assumption:** Counts listed above are repository-source counts, not deployment-time counts after any external preprocessing.
- `data\copilot-models.json` and `data\security-frameworks.json` explicitly flag verification uncertainty inside the files; preserve that uncertainty when using their contents operationally.
