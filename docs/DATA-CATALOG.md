# Data Catalog

## 1. Uebersicht

Alle fachlichen Inhalte des Projekts liegen unter `C:\temp\copilot-cockpit\data\`. Die JSON-Dateien fungieren als **Content-Kataloge**, nicht nur als Konfiguration.

| Datei | Kernzweck | Hauptkonsumenten |
|---|---|---|
| `copilot-instruments.json` | Canonical Catalog fuer Instrumente und Zonen | `app.js`, `ramp.html`, `wiring.html`, `search.js` |
| `copilot-models.json` | Modellkatalog fuer Runway/Tower | `runway.html`, `tower.html`, `app.js`, `search.js` |
| `governance-controls.json` | Governance- und Compliance-Registry | `tower.html`, `app.js`, `search.js` |
| `security-threats.json` | Threat Models pro Instrument | `security.html`, `app.js` |
| `security-frameworks.json` | OWASP-/ATLAS-/CWE-Referenzdaten | `security.html` |
| `terminal-guide.json` | Onboarding-Inhalte | `terminal.html` |
| `jet-bridge-guide.json` | Prompt-/Context-/Agent-Guides | `jet-bridge.html` |
| `preflight-checklist.json` | Rollout-Checkliste | `preflight.html` |
| `known-changelog-entries.json` | Release-Historie | `flight-log.html`, `search.js` |
| `sovereign-cloud.json` | Data Residency und Sovereignty | `tower.html` |
| `wiring-diagram.json` | Feature-Beziehungsgraph | `wiring.html` |

## 2. Dateidetails

### 2.1 `data/copilot-instruments.json`

**Rolle:** Zentraler Katalog fuer Cockpit-Instrumente.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `zones`
- `plans`
- `instruments`

**Struktur**

- `zones[]`: 8 Cockpit-Zonen (`pfd`, `nd`, `glareshield`, `eicas`, `pedestal`, `overhead`, `side`, `fms`)
- `plans[]`: Plan-Tiers (`free`, `pro`, `pro-plus`, `business`, `enterprise`)
- `instruments[]`: Hauptobjekte mit `id`, `symbol`, `name`, `zone`, `status`, `planAvailability`, `capabilities`, `relatedInstruments`, `mermaidDiagrams`, `codeExamples`

**Technische Bedeutung**

- Hub fuer Cross-References in Changelog, Wiring und Security
- bestimmt Karten im Cockpit und Ramp
- liefert Zonen- und Symbolmetadaten fuer Wiring

**Beispiel**

```json
{
  "id": "agent-mode",
  "symbol": "AGT",
  "name": "Agent Mode",
  "zone": "pfd",
  "status": "ga"
}
```

### 2.2 `data/copilot-models.json`

**Rolle:** Modellkatalog fuer `runway.html` und Teile von `tower.html`.

**Top-Level-Keys**

- `$schema`
- `version`
- `lastUpdated`
- `verificationRequired`
- `verificationNotes`
- `description`
- `sources`
- `capabilities`
- `surfaces`
- `plans`
- `models`

**Struktur**

- `capabilities[]`: u. a. `reasoning`, `code`, `long-context`, `vision`, `tool-use`, `speed`
- `surfaces[]`: `chat`, `inline`, `edits`, `agent`, `coding-agent`, `code-review`
- `plans[]`: Plan-Matrix
- `models[]`: eigentliche Modelldefinitionen mit Provider, Availability und Metadaten

**Technische Bedeutung**

- fuellt Runway-Departure-Board und Model-Blade
- liefert Tower-Flight-Plans
- speist Cockpit-EICAS, falls Modell-Enrichment verfuegbar ist

**Wichtig**

- `verificationRequired: true`
- Runway-Tests erwarten aktuell **21 gerenderte Departure Rows**

**Beispiel**

```json
{
  "id": "gpt-4-1",
  "displayName": "GPT-4.1",
  "provider": "OpenAI",
  "surfaceAvailability": {
    "chat": true,
    "inline": true,
    "agent": true
  }
}
```

### 2.3 `data/governance-controls.json`

**Rolle:** Governance-Registry fuer Tower.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `verificationRequired`
- `description`
- `sources`
- `controls`

**Struktur**

- `sources`: Framework-Lexikon fuer `soc2`, `iso27001`, `gdpr`, `hipaa`, `fedramp`, `euAiAct`
- `controls[]`: Controls mit `id`, `category`, `scope`, `defaultState`, `rolloutEffort`, `governanceNote`, `complianceRelevance`

**Technische Bedeutung**

- Tower rendert daraus Framework-Chips und Control-Liste
- `app.js` baut daraus `governanceIndex`
- `search.js` indexiert Controls fuer die globale Suche

**Wichtig**

- `verificationRequired: false`
- Tower-Tests erwarten aktuell **20 gerenderte Controls**

### 2.4 `data/security-threats.json`

**Rolle:** Adversarial View auf sicherheitsrelevante Instrumente.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `description`
- `schema`
- `threats`

**Struktur pro Threat**

- `instrumentId`
- `classification`
- `cia`
- `threatModel`
- `scenario`
- `demo`
- `countermeasures`
- `blastRadius`
- `mitigatesCWE`
- `frameworks`

**Technische Bedeutung**

- treibt den X-Ray-Scanner
- liefert Mermaid-Diagramme, Before/After-Demos und CWE-/Framework-Links
- `app.js` extrahiert daraus `scannerIndex`

**Beispiel**

```json
{
  "instrumentId": "content-exclusion",
  "classification": "preventive",
  "cia": ["confidentiality"],
  "blastRadius": "high"
}
```

### 2.5 `data/security-frameworks.json`

**Rolle:** Registry fuer externe Sicherheitsklassifikationen.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `verificationRequired`
- `description`
- `sources`
- `frameworks`
- `cwePattern`

**Struktur**

- `frameworks.owaspLLM`
- `frameworks.atlas`
- `cwePattern.urlTemplate`

**Technische Bedeutung**

- `security.html` baut daraus Framework-Chips mit externen Links
- IDs aus `security-threats.json` werden hier aufgeloest

**Wichtig**

- `verificationRequired: true`
- Datei kennzeichnet sich selbst explizit als **manuell zu verifizieren**

### 2.6 `data/terminal-guide.json`

**Rolle:** Fachinhalte fuer `terminal.html`.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `description`
- `checkIn`
- `boardingPass`
- `firstFlight`
- `departures`

**Inhalt**

- 5 Plan-Karten
- 6 IDE-Karten
- 3 erste Uebungen
- 5 Weiterleitungsziele

**Technische Bedeutung**

- reines Content-Modell fuer page-lokales Rendering
- keine Cross-References auf andere JSON-Dateien notwendig

### 2.7 `data/jet-bridge-guide.json`

**Rolle:** Lern- und Pattern-Katalog fuer `jet-bridge.html`.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `description`
- `promptCraft`
- `contextManagement`
- `editMode`
- `agentPatterns`
- `nextSteps`

**Inhalt**

- 6 Prompt-Techniken
- 3 Participants (`@workspace`, `@vscode`, `@terminal`)
- 3 Variablen (`#file`, `#selection`, `#codebase`)
- 3 Edit-Workflows
- 5 Agent-Patterns

**Technische Bedeutung**

- strikt in Sections gruppiertes Content-Schema
- Inhalte werden 1:1 in Karten und Beispielblöcke ueberfuehrt

### 2.8 `data/preflight-checklist.json`

**Rolle:** Persistente Rollout-/Onboarding-Checkliste.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `description`
- `intro`
- `categories`

**Struktur**

- 6 Kategorien
- jede Kategorie besitzt `id`, `title`, `icon`, `perspective`, `items[]`

**Technische Bedeutung**

- `perspective` verlinkt in andere HTML-Seiten
- `items[].id` wird als LocalStorage-Key fuer Checklistenstatus verwendet

**Beispiel**

```json
{
  "id": "tower",
  "perspective": "tower.html",
  "items": [
    { "id": "org-policy-reviewed", "label": "Organization Copilot policy reviewed" }
  ]
}
```

### 2.9 `data/known-changelog-entries.json`

**Rolle:** Changelog und Historie der Copilot-Features.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `entryTypes`
- `entries`

**Struktur**

- `entryTypes`: `departure`, `cleared`, `upgrade`, `notam`, `grounded`, `turbulence`
- `entries[]`: `id`, `date`, `type`, `title`, `description`, `instruments[]`, `zone`, `source`

**Technische Bedeutung**

- fuellt `flight-log.html`
- wird von `search.js` als eigener Suchtyp indexiert
- verlinkt ueber `instruments[]` zurueck in das Cockpit

**Wichtig**

- README nennt **35 Eintraege**
- `integrity.spec.js` validiert Instrument-Referenzen und Entry-Typen

### 2.10 `data/sovereign-cloud.json`

**Rolle:** Datenmodell fuer Tower-Unterbereich "Sovereign Cloud & Data Residency".

**Top-Level-Keys**  
im sichtbaren Dateibereich:

- `version`
- `lastUpdated`
- `description`
- `sovereignPillars`
- `deploymentOptions`

Weitere Bereiche werden durch Tower-Tests und Rendering benoetigt:

- Provider-Strategien
- Residual Risks
- Data-Flow-Matrix

**Inhalt**

- 3 Sovereignty Pillars
- 6 Deployment Options
- 4 Provider Strategy Cards
- 3 Residual Risk Cards

**Technische Bedeutung**

- liefert Tower-Diagramm, Optionskarten, Matrix und Highlight-Ziele fuer `#sovereign=...`

### 2.11 `data/wiring-diagram.json`

**Rolle:** Graphmodell fuer `wiring.html`.

**Top-Level-Keys**

- `version`
- `lastUpdated`
- `description`
- `intro`
- `connectionTypes`
- `connections`

**Struktur**

- `connectionTypes[]`: `context`, `powers`, `governs`, `extends`
- `connections[]`: `from`, `to`, `type`, `label`

**Technische Bedeutung**

- wird mit `copilot-instruments.json` gemerged
- erzeugt Mermaid-Source fuer Knotengruppen nach Zone
- Click-Ziele springen ins Cockpit

**Wichtig**

- README beschreibt **55 Verbindungen**
- `integrity.spec.js` validiert Connection-Typen und Endpunkte

## 3. Datenbeziehungen

### Hub-Datei

`copilot-instruments.json` ist die wichtigste Referenzdatei. Sie wird benutzt von:

- `security-threats.json` via `instrumentId`
- `known-changelog-entries.json` via `instruments[]`
- `wiring-diagram.json` via `from`/`to`
- `search.js` fuer globale Suche

### Modell- und Governance-Bezug

- `copilot-models.json` wird sowohl fachlich in `runway.html` als auch administrativ in `tower.html` verwendet.
- `governance-controls.json` definiert IDs, die zugleich in Deep Links und in Wiring-Kanten auftauchen.

## 4. Datenqualitaet und Testabdeckung

`tests/integrity.spec.js` validiert derzeit:

- Changelog -> Instrument-Referenzen
- Wiring-Endpunkte
- `relatedInstruments`
- Instrument-ID-Duplikate
- Pflichtfelder
- gueltige Zonen
- Changelog-Typen
- Wiring-Typen
- Modell-ID-Duplikate

Nicht alle JSON-Dateien werden dadurch gleich stark abgesichert. Besonders inhaltliche Guide-Dateien (`terminal-guide.json`, `jet-bridge-guide.json`) sind vor allem durch Seiten-Renderingstests abgedeckt.
