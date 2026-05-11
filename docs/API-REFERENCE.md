# API-Reference

## 1. API-Oberflaeche des Repositories

Dieses Repository bietet keine serverseitige JSON- oder RPC-API. Die technische API-Oberflaeche besteht aus:

1. statischen HTML-Routen,
2. statischen JSON-Endpunkten unter `/data/`,
3. URL-Hash-Kontrakten,
4. `localStorage`-Keys,
5. wenigen globalen Browser-Funktionen.

## 2. HTML-Routen

| Route | Dateipfad | Zweck |
| --- | --- | --- |
| `/` | `C:\temp\copilot-cockpit\index.html` | Cockpit-Hauptansicht |
| `/index.html` | `C:\temp\copilot-cockpit\index.html` | Alias fuer Cockpit |
| `/terminal.html` | `C:\temp\copilot-cockpit\terminal.html` | Einstieg / Plan / IDE / Erstnutzung |
| `/jet-bridge.html` | `C:\temp\copilot-cockpit\jet-bridge.html` | Prompt- und Agent-Muster |
| `/ramp.html` | `C:\temp\copilot-cockpit\ramp.html` | Agenten-/MCP-Perspektive |
| `/runway.html` | `C:\temp\copilot-cockpit\runway.html` | Modellkatalog |
| `/tower.html` | `C:\temp\copilot-cockpit\tower.html` | Governance und Sovereign Cloud |
| `/security.html` | `C:\temp\copilot-cockpit\security.html` | Threat Scanner |
| `/flight-log.html` | `C:\temp\copilot-cockpit\flight-log.html` | Changelog-Timeline |
| `/preflight.html` | `C:\temp\copilot-cockpit\preflight.html` | Checkliste |
| `/wiring.html` | `C:\temp\copilot-cockpit\wiring.html` | Verbindungsgraph |

## 3. JSON-Endpunkte unter `/data/`

Alle Daten werden per `GET /data/<datei>.json` geladen. Es existiert keine Schreib-API.

| Endpunkt | Primaere Konsumenten | Top-Level-Vertrag |
| --- | --- | --- |
| `/data/copilot-instruments.json` | `app.js`, `ramp.html`, `security.html`, `wiring.html`, `search.js` | `version`, `lastUpdated`, `zones[]`, `plans[]`, `instruments[]` |
| `/data/copilot-models.json` | `runway.html`, `tower.html`, `app.js`, `search.js` | `$schema`, `version`, `lastUpdated`, `verificationRequired`, `sources[]`, `capabilities[]`, `surfaces[]`, `plans[]`, `models[]`, `notams[]`, `copilotEngine`, `flightPlans[]` |
| `/data/governance-controls.json` | `tower.html`, `app.js`, `search.js` | `version`, `lastUpdated`, `verificationRequired`, `sources`, `controls[]` |
| `/data/sovereign-cloud.json` | `tower.html` | `version`, `lastUpdated`, `sovereignPillars[]`, `deploymentOptions[]`, `providerStrategies[]`, `residualRisks[]`, `dataFlowDiagram` |
| `/data/security-threats.json` | `security.html`, `app.js` | `version`, `lastUpdated`, `schema`, `threats[]` |
| `/data/security-frameworks.json` | `security.html` | `version`, `lastUpdated`, `verificationRequired`, `sources`, `frameworks`, `cwePattern` |
| `/data/terminal-guide.json` | `terminal.html` | `version`, `lastUpdated`, `checkIn`, `boardingPass`, `firstFlight`, `departures` |
| `/data/jet-bridge-guide.json` | `jet-bridge.html` | `version`, `lastUpdated`, `promptCraft`, `contextManagement`, `editMode`, `agentPatterns`, `nextSteps` |
| `/data/known-changelog-entries.json` | `flight-log.html`, `search.js`, `tests\integrity.spec.js` | `version`, `lastUpdated`, `entryTypes`, `entries[]` |
| `/data/preflight-checklist.json` | `preflight.html` | `version`, `lastUpdated`, `intro`, `categories[]` |
| `/data/wiring-diagram.json` | `wiring.html`, `tests\integrity.spec.js` | `version`, `lastUpdated`, `intro`, `connectionTypes[]`, `connections[]`, `zoneDescriptions` |

## 4. Hash-Kontrakte

### 4.1 Cockpit-Instrumente

- Muster: `#instrument-<instrumentId>`
- Produzenten:
  - `app.js` beim Oeffnen des Detailpanels
  - `ramp.html` beim Oeffnen des Ramp-Blades
  - `wiring.html` in Mermaid-Click-Links
  - `flight-log.html` in Instrument-Links
  - `search.js` in globalen Suchtreffern
- Konsumenten:
  - `app.js`
  - `ramp.html`

### 4.2 Modelle

- Muster: `#model-<modelId>`
- Produzenten:
  - `runway.html`
  - `search.js`
  - `app.js` fuer EICAS-Links nach Runway
- Konsument:
  - `runway.html`

### 4.3 Security-Scanner

- Muster: `#scan=<instrumentId>`
- Produzenten:
  - `security.html`
  - Cockpit-Callout in `app.js`
- Konsument:
  - `security.html`

### 4.4 Governance und Sovereign Cloud

- Muster:
  - `#control=<controlId>`
  - `#sovereign=<optionId>`
- Produzenten:
  - `tower.html`
  - Cockpit-Callout in `app.js`
  - `search.js` fuer Controls
- Konsument:
  - `tower.html`

## 5. Browser-Persistenz

| Key | Typ | Semantik |
| --- | --- | --- |
| `cockpit-theme` | String (`light` oder `dark`) | Globales Theme ueber alle Seiten |
| `cockpit-last-scan` | String (`instrumentId`) | Letzter aktiver Security-Scan |
| `cockpit-security-posture` | JSON-Objekt | Checkbox-Zustand fuer Security-Posture |
| `copilot-preflight` | JSON-Objekt | Checkbox-Zustand fuer Preflight |

## 6. Globale JavaScript-Funktionen

| Funktion | Quelle | Zweck |
| --- | --- | --- |
| `toggleTheme()` | `app.js` | Theme-Wechsel auf der Cockpit-Seite |
| `copyCode(button)` | `app.js` | Kopiert Codeblock-Inhalt |
| `window.openGlobalSearch()` | `search.js` | Oeffnet die globale Suchpalette |

Hinweis: Die meisten Perspektivseiten kapseln ihre Funktionen im Inline-Skript und exportieren keine globale API.

## 7. Cache- und Auslieferungsvertrag

`vercel.json` definiert fuer API-aehnliche Datenzugriffe:

- `/data/*` -> `Cache-Control: public, max-age=3600, must-revalidate`
- `/*.js` -> `Cache-Control: public, max-age=3600, must-revalidate`
- `/*.css` -> `Cache-Control: public, max-age=3600, must-revalidate`

Damit ist die "API" semantisch read-only und dateibasiert.

## 8. Testbezug pro Vertrag

| Vertrag | Testdateien |
| --- | --- |
| `#instrument-<id>` | `tests\cockpit.spec.js`, `tests\ramp.spec.js`, `tests\flight-log.spec.js` |
| `#model-<id>` | `tests\runway.spec.js` |
| `#scan=<id>` | `tests\security.spec.js` |
| `#control=<id>`, `#sovereign=<id>` | `tests\tower.spec.js` |
| Cross-JSON-Referenzen | `tests\integrity.spec.js` |

## 9. Annahmen

1. **Kein externer Schreibzugriff:** Da nur statische Dateien sichtbar sind, wird angenommen, dass Vercel oder ein anderes Hosting keine verdeckte Write-API fuer Inhaltsmutationen bereitstellt.
2. **JSON ist kanonisch:** Wenn HTML-Text und JSON-Werte abweichen, gilt fuer technische Integration der JSON-Endpunkt als Vertragsquelle.
