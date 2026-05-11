# Data Catalog

## 1. Kataloguebersicht

Alle produktiven Inhaltsdaten liegen unter `C:\temp\copilot-cockpit\data\`.

| Datei | Primaere Collections/Objekte | Hauptkonsumenten | Verifikationshinweis |
| --- | --- | --- | --- |
| `copilot-instruments.json` | `zones[]`, `plans[]`, `instruments[]` | Cockpit, Ramp, Security, Wiring, Search | Kein explizites `verificationRequired`; Referenzen werden durch `tests\integrity.spec.js` abgesichert |
| `copilot-models.json` | `sources[]`, `capabilities[]`, `surfaces[]`, `plans[]`, `models[]`, `notams[]`, `flightPlans[]`, `copilotEngine` | Runway, Tower, Cockpit, Search | `verificationRequired: true` |
| `governance-controls.json` | `sources`, `controls[]` | Tower, Cockpit, Search | `verificationRequired: false` |
| `sovereign-cloud.json` | `sovereignPillars[]`, `deploymentOptions[]`, `providerStrategies[]`, `residualRisks[]`, `dataFlowDiagram` | Tower | Keine explizite Verifikationsflagge |
| `security-threats.json` | `schema`, `threats[]` | Security, Cockpit | Keine explizite Verifikationsflagge |
| `security-frameworks.json` | `sources`, `frameworks`, `cwePattern` | Security | `verificationRequired: true` |
| `terminal-guide.json` | `checkIn`, `boardingPass`, `firstFlight`, `departures` | Terminal | Keine explizite Verifikationsflagge |
| `jet-bridge-guide.json` | `promptCraft`, `contextManagement`, `editMode`, `agentPatterns`, `nextSteps` | Jet Bridge | Keine explizite Verifikationsflagge |
| `known-changelog-entries.json` | `entryTypes`, `entries[]` | Flight Log, Search, Integrity-Tests | Keine explizite Verifikationsflagge |
| `preflight-checklist.json` | `intro`, `categories[]` | Pre-Flight | Keine explizite Verifikationsflagge |
| `wiring-diagram.json` | `connectionTypes[]`, `connections[]`, `zoneDescriptions` | Wiring, Integrity-Tests | Keine explizite Verifikationsflagge |

## 2. Dateispezifische Vertrage

### 2.1 `copilot-instruments.json`

Pfad: `C:\temp\copilot-cockpit\data\copilot-instruments.json`

Technischer Vertrag:

- `zones[]`: 8 definierte Zonen
- `plans[]`: 5 Plaene (`free`, `pro`, `pro-plus`, `business`, `enterprise`)
- `instruments[]`: kanonische Instrumentdefinitionen fuer das Cockpit

Wichtige Felder pro Instrument:

- `id`
- `symbol`
- `name`
- `zone`
- `status`
- `perspectives`
- `planAvailability`
- optionale Felder wie `capabilities`, `ideSupport`, `relatedInstruments`, `codeExamples`, `mermaidDiagrams`, `terminalRecordings`

Abhaengigkeiten:

- `security-threats.json.threats[].instrumentId`
- `known-changelog-entries.json.entries[].instruments[]`
- `wiring-diagram.json.connections[].from|to`

### 2.2 `copilot-models.json`

Pfad: `C:\temp\copilot-cockpit\data\copilot-models.json`

Technischer Vertrag:

- `$schema`: zeigt auf `./schema/copilot-models.schema.json`
- `verificationRequired: true`
- `models[]`: derzeit testseitig indirekt als 21 sichtbare Modelle abgesichert (`tests\runway.spec.js`)
- `notams[]`: testseitig als 5 Eintraege abgesichert
- `flightPlans[]`: testseitig als 5 Karten abgesichert

Beobachtete Upstream-Quellen im Datensatz:

- GitHub Docs - Model comparison
- GitHub Docs - Supported models
- GitHub Docs - Copilot plans

Offline-Herkunft:

- `tools\enrich\README.md`
- `tools\enrich\sources.yml`

### 2.3 `governance-controls.json`

Pfad: `C:\temp\copilot-cockpit\data\governance-controls.json`

Technischer Vertrag:

- `verificationRequired: false`
- `sources`: 6 Framework-Quellen (`soc2`, `iso27001`, `gdpr`, `hipaa`, `fedramp`, `euAiAct`)
- `controls[]`: testseitig als 20 Controls abgesichert

Jede Control enthaelt mindestens:

- `id`
- `category`
- `scope`
- `defaultState`
- `rolloutEffort`
- `governanceNote`
- `complianceRelevance[]`

### 2.4 `sovereign-cloud.json`

Pfad: `C:\temp\copilot-cockpit\data\sovereign-cloud.json`

Technischer Vertrag:

- `sovereignPillars[]`: testseitig 3 Eintraege
- `deploymentOptions[]`: testseitig 6 Optionen
- `providerStrategies[]`: testseitig 4 Strategien
- `residualRisks[]`: testseitig 3 Risiken
- `dataFlowDiagram`: Mermaid-Quelltext fuer Tower

### 2.5 `security-threats.json`

Pfad: `C:\temp\copilot-cockpit\data\security-threats.json`

Technischer Vertrag gemaess eingebettetem `schema`:

- `instrumentId`
- `classification`
- `cia[]`
- `threatModel`
- `scenario`
- `demo`
- `countermeasures[]`
- `blastRadius`
- `mitigatesCWE[]`

Die Datei ist direkt mit `copilot-instruments.json` gekoppelt.

### 2.6 `security-frameworks.json`

Pfad: `C:\temp\copilot-cockpit\data\security-frameworks.json`

Technischer Vertrag:

- `verificationRequired: true`
- `sources.owasp-llm`
- `sources.mitre-atlas`
- `sources.cwe`
- `frameworks.owaspLLM`
- `frameworks.atlas`
- `cwePattern.urlTemplate`

Diese Datei ist bewusst als teilweise unbestaetigt markiert und muss als Annahme behandelt werden, solange `verified: false` bzw. `lastVerified: null` gesetzt ist.

### 2.7 `terminal-guide.json`

Pfad: `C:\temp\copilot-cockpit\data\terminal-guide.json`

Testseitig abgesicherte Struktur:

- 5 Plan-Karten
- 6 IDE-Karten
- 3 Exercise-Karten
- 5 Departure-Links

### 2.8 `jet-bridge-guide.json`

Pfad: `C:\temp\copilot-cockpit\data\jet-bridge-guide.json`

Testseitig abgesicherte Struktur:

- 6 Prompt-Techniken
- 3 Participant-Karten
- 3 Variable-Karten
- 3 Edit-Workflows
- 5 Agent-Patterns
- 5 Next-Step-Links

### 2.9 `known-changelog-entries.json`

Pfad: `C:\temp\copilot-cockpit\data\known-changelog-entries.json`

Technischer Vertrag:

- `entryTypes`: 6 Typen (`departure`, `cleared`, `upgrade`, `notam`, `grounded`, `turbulence`)
- `entries[]`: Timeline-Eintraege mit `id`, `date`, `type`, `title`, `description`, `instruments[]`, `zone`, `source`

Integritaetsvertrag:

- Jeder Typ in `entries[].type` muss in `entryTypes` definiert sein.
- Jede Instrument-Referenz in `entries[].instruments[]` muss in `copilot-instruments.json` existieren.

### 2.10 `preflight-checklist.json`

Pfad: `C:\temp\copilot-cockpit\data\preflight-checklist.json`

Testseitig abgesicherte Struktur:

- 6 Kategorien
- Kategorien enthalten `perspective`-Links auf HTML-Seiten
- Item-Zustaende werden nicht im JSON selbst, sondern in `localStorage['copilot-preflight']` gespeichert

### 2.11 `wiring-diagram.json`

Pfad: `C:\temp\copilot-cockpit\data\wiring-diagram.json`

Technischer Vertrag:

- `connectionTypes[]`: 4 Typen (`context`, `powers`, `governs`, `extends`)
- `connections[]`: Kanten mit `from`, `to`, `type`, `label`
- `zoneDescriptions`: optionale Textueberschreibung pro Zone

Integritaetsvertrag:

- `connections[].type` muss in `connectionTypes[].id` enthalten sein.
- `connections[].from` und `connections[].to` muessen auf bekannte Instrumente oder Controls zeigen.

## 3. Datenquellen ausserhalb von `data\`

| Quelle | Pfad | Rolle |
| --- | --- | --- |
| Model-Enrichment-Registry | `C:\temp\copilot-cockpit\tools\enrich\sources.yml` | Definiert Upstream-Quellen fuer Modellanreicherung |
| Model-Enrichment-Dokumentation | `C:\temp\copilot-cockpit\tools\enrich\README.md` | Beschreibt Harvest-/Normalize-Vertrag und Schreibverbot direkt nach `data\` |
| Deployment-Header | `C:\temp\copilot-cockpit\vercel.json` | Bestimmt Cache-Verhalten fuer `data\*.json` |

## 4. Testbezug

Zentrale Datenvertraege werden durch `C:\temp\copilot-cockpit\tests\integrity.spec.js` abgesichert:

- gueltige Changelog-zu-Instrument-Referenzen
- gueltige Wiring-Endpunkte
- gueltige `relatedInstruments`
- keine doppelten Instrument-IDs
- Pflichtfelder pro Instrument
- gueltige Zonen
- gueltige Changelog-Typen
- gueltige Wiring-Typen
- keine doppelten Model-IDs

## 5. Annahmen

1. **Instrumentanzahl wird nicht fest fixiert:** Mehrere Freitexte im Repository nennen unterschiedliche Summen. Fuer diesen Katalog wird deshalb der Strukturvertrag dokumentiert, nicht eine aus Prosa abgeleitete exakte Anzahl.
2. **Model- und Framework-Frische ist teilweise offen:** `copilot-models.json` und `security-frameworks.json` markieren selbst, dass Inhalte noch nicht voll verifiziert sind. Diese Unsicherheit ist Teil des Datenvertrags und kein Dokumentationsfehler.
