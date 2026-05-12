# Data catalog

The site is data-driven. Most page content is authored in JSON and rendered in the browser rather than hard-coded in HTML.

## Catalog inventory

| File | Main collection size | Primary consumers | Verification signal |
| --- | ---: | --- | --- |
| `data/copilot-instruments.json` | 46 instruments | cockpit, security, ramp, wiring, search | none |
| `data/copilot-models.json` | 21 models | cockpit, runway, tower, search | `verificationRequired: true` |
| `data/governance-controls.json` | 20 controls | cockpit, tower, search, wiring | `verificationRequired: false` |
| `data/security-threats.json` | 22 threats | cockpit, security | none |
| `data/security-frameworks.json` | registry object | security | `verificationRequired: true` |
| `data/sovereign-cloud.json` | 6 deployment options | tower | none |
| `data/terminal-guide.json` | 4 major sections | terminal | none |
| `data/jet-bridge-guide.json` | 5 major sections | jet bridge | none |
| `data/preflight-checklist.json` | 6 categories, 22 items | pre-flight | none |
| `data/known-changelog-entries.json` | 35 entries | flight log, search | none |
| `data/wiring-diagram.json` | 55 connections | wiring | none |

## Core schema shapes

### `data/copilot-instruments.json`

Top-level keys:

- `version`
- `lastUpdated`
- `zones[]`
- `plans[]`
- `instruments[]`

Observed instrument shape:

```json
{
  "id": "agent-mode",
  "symbol": "AGT",
  "name": "Agent Mode",
  "zone": "pfd",
  "perspectives": ["cockpit", "terminal"],
  "status": "ga",
  "statusHistory": [{ "status": "preview", "date": "2025-02" }],
  "description": "...",
  "shortDescription": "...",
  "planAvailability": { "free": "limited", "pro": true },
  "ideSupport": { "vscode": "ga", "jetbrains": "ga" },
  "flightMode": ["beginner", "advanced"],
  "capabilities": ["..."],
  "squawkCodes": [{ "code": "7600", "title": "...", "description": "..." }],
  "securityRelevance": { "relevant": true, "aspects": ["..."] },
  "proTip": "...",
  "links": { "docs": "...", "changelog": "...", "learn": "..." },
  "relatedInstruments": ["mcp"],
  "mermaidDiagrams": [],
  "codeExamples": [],
  "terminalRecordings": [],
  "videos": [],
  "lastVerified": "2026-04-14",
  "confidenceScore": 95
}
```

Important rules:

- `zone` must match `zones[].id`.
- `id` is the primary foreign key used across the site.
- `planAvailability` values are mixed: `true`, `false` or omitted, and `"limited"`.
- `capabilities` can be a string array or an object array; `app.js` handles both.
- `links` can be an object in instrument data; `app.js` also supports an array form.

### `data/copilot-models.json`

Top-level keys used by runtime code:

- metadata: `$schema`, `version`, `lastUpdated`, `verificationRequired`, `verificationNotes`, `description`
- lookup tables: `sources[]`, `capabilities[]`, `surfaces[]`, `plans[]`
- content: `models[]`, `notams[]`, `flightPlans[]`, `copilotEngine`

Observed model shape:

```json
{
  "id": "gpt-4-1",
  "displayName": "GPT-4.1",
  "provider": "OpenAI",
  "family": "GPT",
  "status": "ga",
  "tagline": "...",
  "strengths": ["..."],
  "limitations": [],
  "contextWindow": 1047576,
  "capabilities": { "reasoning": "partial", "code": "strong" },
  "surfaceAvailability": { "chat": true, "inline": true, "coding-agent": true },
  "planAvailability": { "free": true, "pro": true, "enterprise": true },
  "pricingMultiplier": 0.0,
  "releasedAt": "2025-04-14",
  "knowledgeCutoff": "2024-06-30",
  "inputModalities": ["image", "text", "file"],
  "outputModalities": ["text"],
  "modeAvailability": { "agent": true, "ask": true, "edit": true },
  "ideAvailability": { "github-com": true, "copilot-cli": true, "vscode": true },
  "taskArea": "...",
  "excelsAt": "...",
  "modelCardUrl": "...",
  "sourceLinks": ["..."],
  "confidenceScore": 70,
  "lastVerified": "2026-04-10",
  "verificationRequired": true
}
```

Important rules:

- `runway.html` treats `verificationRequired` as a visible warning banner.
- `app.js` derives cockpit engine-zone links from non-deprecated models only.
- `flightPlans[].recommendedModels[]` and `avoidModels[]` are expected to resolve to model IDs.
- `notams[].affectedModels[]` are expected to resolve to model IDs.

### `data/governance-controls.json`

Top-level keys:

- `version`
- `lastUpdated`
- `verificationRequired`
- `description`
- `sources`
- `controls[]`

Observed control shape:

```json
{
  "id": "feature-policies",
  "category": "policy",
  "scope": "org",
  "defaultState": "off",
  "rolloutEffort": "low",
  "governanceNote": "...",
  "complianceRelevance": ["SOC2-CC6.1", "ISO-A.5.15", "GDPR-Art25"]
}
```

Important rules:

- Control IDs double as Tower deep-link identifiers.
- Some control IDs intentionally align with instrument IDs so the cockpit can render governance callouts from `instrument.id`.
- Wiring endpoints may reference control IDs as well as instrument IDs.

### `data/security-threats.json`

Top-level keys:

- `version`
- `lastUpdated`
- `description`
- `schema`
- `threats[]`

Observed threat shape:

```json
{
  "instrumentId": "content-exclusion",
  "classification": "preventive",
  "cia": ["confidentiality"],
  "threatModel": { "title": "...", "diagram": "sequenceDiagram ..." },
  "scenario": { "severity": "critical", "likelihood": "high", "narrative": "..." },
  "demo": {
    "type": "before-after",
    "vulnerable": { "label": "...", "language": "bash", "filename": "...", "code": "..." },
    "hardened": { "label": "...", "language": "bash", "filename": "...", "code": "..." }
  },
  "countermeasures": ["..."],
  "blastRadius": "high",
  "mitigatesCWE": ["CWE-200"],
  "frameworks": { "owaspLLM": ["LLM02:2025"], "atlas": ["AML.T0057"] }
}
```

Important rules:

- `instrumentId` must resolve to an instrument ID.
- `frameworks` references are resolved against `data/security-frameworks.json`.
- Mermaid diagrams are stored as strings and rendered at runtime.

## Guide-style catalogs

| File | Top-level structure | Runtime expectation |
| --- | --- | --- |
| `data/terminal-guide.json` | `checkIn`, `boardingPass`, `firstFlight`, `departures` | each section renders cards or links in `terminal.html` |
| `data/jet-bridge-guide.json` | `promptCraft`, `contextManagement`, `editMode`, `agentPatterns`, `nextSteps` | each section renders tutorial cards in `jet-bridge.html` |
| `data/preflight-checklist.json` | `intro`, `categories[]` | categories include `perspective` links and `items[]` with checkbox IDs |
| `data/known-changelog-entries.json` | `entryTypes`, `entries[]` | `entries[].type` must resolve into `entryTypes`; `entries[].instruments[]` link back to cockpit instruments |
| `data/wiring-diagram.json` | `intro`, `connectionTypes[]`, `connections[]`, `zoneDescriptions` | `connections[].type` must resolve into `connectionTypes`; endpoints must resolve to instruments or controls |
| `data/sovereign-cloud.json` | `sovereignPillars[]`, `deploymentOptions[]`, `providerStrategies[]`, `residualRisks[]`, `dataFlowDiagram`, `sources` | Tower uses these sections directly for sovereignty pages and Mermaid rendering |
| `data/security-frameworks.json` | `sources`, `frameworks`, `cwePattern` | Security page resolves OWASP LLM, ATLAS, and CWE links from this registry |

## Cross-file foreign keys

| Source | Target | Enforced today |
| --- | --- | --- |
| `security-threats.threats[].instrumentId` | `copilot-instruments.instruments[].id` | yes, integrity spec |
| `known-changelog-entries.entries[].instruments[]` | `copilot-instruments.instruments[].id` | yes, integrity spec |
| `copilot-instruments.instruments[].relatedInstruments[]` | `copilot-instruments.instruments[].id` | yes, integrity spec |
| `wiring-diagram.connections[].from/to` | instrument IDs or control IDs | yes, integrity spec |
| `copilot-instruments.instruments[].zone` | `copilot-instruments.zones[].id` | yes, integrity spec |
| `known-changelog-entries.entries[].type` | `known-changelog-entries.entryTypes` | yes, integrity spec |
| `wiring-diagram.connections[].type` | `wiring-diagram.connectionTypes[].id` | yes, integrity spec |
| `copilot-models.flightPlans[].recommendedModels[]` | `copilot-models.models[].id` | not in integrity spec |
| `copilot-models.flightPlans[].avoidModels[]` | `copilot-models.models[].id` | not in integrity spec |
| `copilot-models.notams[].affectedModels[]` | `copilot-models.models[].id` | not in integrity spec |
| `security-threats.frameworks.*` | `security-frameworks.frameworks.*` | resolved at runtime, not in integrity spec |

## Validation coverage

`tests/integrity.spec.js` is the main automated data guard. It verifies:

- no duplicate instrument IDs,
- no duplicate model IDs,
- required instrument fields,
- valid instrument zone references,
- valid changelog entry types,
- valid wiring connection types,
- valid cross-file references for changelog entries, related instruments, and wiring endpoints.

What it does **not** currently verify:

- model `notams[].affectedModels[]`,
- model `flightPlans` model references,
- security framework ID lookups,
- uniqueness of checklist item IDs across categories,
- uniqueness of governance control IDs.

## Maintenance notes

1. Prefer editing JSON when page text already lives in a catalog.
2. Treat IDs as stable public-ish implementation keys because they appear in hashes, search results, and persisted browser state.
3. If a catalog contains `verificationRequired: true`, preserve that uncertainty in downstream documentation and UI messaging.
4. `tools/enrich/` exists only for model catalog maintenance; it is explicitly human-gated and does not auto-write into `data/copilot-models.json`.

## Explicit uncertainty

- Counts in this file reflect the repository source examined during this pass, not any separately hosted deployment.
- `data/copilot-models.json` and `data/security-frameworks.json` explicitly flag verification work in progress; do not present them as fully authoritative without a human review.
