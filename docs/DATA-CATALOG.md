# Data Catalog

## 1. Warum dieser Katalog wichtig ist

Die JSON-Dateien unter `C:\temp\copilot-cockpit\data` sind nicht bloss Konfiguration. Sie sind **die fachliche Quelle der Anwendung**. Wer Inhalte aendert, aendert damit direkt Rendering, Deep Links, Suche und oft auch Testverhalten.

## 2. Gesamtuebersicht

| Datei | Rolle | Wichtige Konsumenten | Kritische Pflegehinweise |
|---|---|---|---|
| `copilot-instruments.json` | Hauptkatalog fuer Zonen, Plaene, Instrumente | `app.js`, `ramp.html`, `security.html`, `wiring.html`, `search.js` | IDs sind Hub-Referenzen fuer mehrere andere Dateien |
| `copilot-models.json` | Modellkatalog fuer Runway und Teile von Tower/Cockpit | `runway.html`, `tower.html`, `app.js`, `search.js` | Datei markiert sich selbst als verifikationspflichtig |
| `governance-controls.json` | Governance-Registry | `tower.html`, `app.js`, `search.js` | Control-IDs muessen mit Deep Links stabil bleiben |
| `security-threats.json` | Threat-Katalog pro Instrument | `security.html`, `app.js` | `instrumentId` muss auf existierende Instrumente zeigen |
| `security-frameworks.json` | Registry fuer OWASP/ATLAS/CWE | `security.html` | Framework-Links und Summaries sind teils noch manuell zu verifizieren |
| `terminal-guide.json` | Content-Modell fuer Einstieg | `terminal.html` | rein redaktionell, aber DOM-Rendering erwartet definierte Sektionen |
| `jet-bridge-guide.json` | Content-Modell fuer Prompt- und Agent-Guide | `jet-bridge.html` | strukturierte Sections statt freies Rich-Content-Modell |
| `preflight-checklist.json` | Interaktive Checklist | `preflight.html` | `items[].id` ist Persistenzschluessel |
| `known-changelog-entries.json` | Release- und Event-Historie | `flight-log.html`, `search.js` | Entry-Typen und Instrument-Referenzen muessen konsistent sein |
| `sovereign-cloud.json` | Data-Residency- und Sovereignty-Modell | `tower.html` | mehrere Teilbereiche speisen unterschiedliche UI-Sektionen |
| `wiring-diagram.json` | Verbindungsgraph | `wiring.html` | `from`/`to` muessen existierende Instrumente referenzieren |

## 3. Datei-fuer-Datei-Referenz

### 3.1 `copilot-instruments.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `zones`, `plans`, `instruments` |
| Aktuelle Groesse | 8 Zonen, 5 Plaene, 46 Instrumente |
| Hauptfunktion | Canonical Catalog fuer Cockpit, Ramp, Wiring und Teile von Security/Search |
| Typische Referenzen | `relatedInstruments`, `planAvailability`, `securityRelevance`, `links`, `mermaidDiagrams`, `codeExamples` |

**Konsumenten**

- `app.js` rendert Cockpit, Blade, Filter und Ressourcen-Tab.
- `ramp.html` filtert auf Instrumente mit `perspectives.includes('ramp')`.
- `security.html` loest Symbol und Name fuer Threats auf.
- `wiring.html` nutzt Zone, Symbol und Name fuer Knoten.
- `search.js` indexiert Instrumente global.

**Pflegehinweise**

1. Instrument-IDs sind externe Referenzen fuer `security-threats.json`, `known-changelog-entries.json` und `wiring-diagram.json`.
2. Eine ID-Umbenennung ist nie lokal.
3. Kommentare oder Begleittexte mit alten Gesamtzahlen sind nicht massgeblich; der Katalog selbst ist die Quelle.

### 3.2 `copilot-models.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `$schema`, `version`, `lastUpdated`, `verificationRequired`, `verificationNotes`, `description`, `sources`, `capabilities`, `surfaces`, `plans`, `models` |
| Aktuelle Groesse | 3 Quellen, 6 Capabilities, 6 Surfaces, 5 Plaene, 21 Modelle |
| Hauptfunktion | fuellt Runway-Board, Model Blade, Tower-Flight-Plans und Cockpit-EICAS-Enrichment |
| Wichtige Felder | `planAvailability`, `surfaceAvailability`, `ideAvailability`, `status`, `taskFit`, `sourceLinks` |

**Besonderheit**

Die Datei markiert sich selbst mit `verificationRequired: true`. Dokumentation und Betrieb sollten das nicht weichzeichnen.

**Pflegehinweise**

1. Plan-/Surface-Aenderungen beeinflussen Runway-Filter und Blade-Matrizen.
2. Deprecation-Infos steuern Grounded-Darstellung und Alternativlinks.
3. Flight Plans in `tower.html` erwarten stabile Modell-IDs.

### 3.3 `governance-controls.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `verificationRequired`, `description`, `sources`, `controls` |
| Aktuelle Groesse | 6 Framework-Quellen, 20 Controls |
| Hauptfunktion | Governance-Liste und Framework-Legende im Tower |
| Wichtige Felder | `id`, `category`, `scope`, `defaultState`, `rolloutEffort`, `governanceNote`, `complianceRelevance` |

**Pflegehinweise**

1. `id` speist `#control=<id>`-Deep-Links.
2. `sources` sind direkt verlinkt; kaputte URLs sind sofort sichtbar.
3. `complianceRelevance` taucht als Badge in der UI auf.

### 3.4 `security-threats.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `description`, `schema`, `threats` |
| Aktuelle Groesse | 22 Threat-Eintraege |
| Hauptfunktion | Scanner-Inhalte, Threat Models, Demos, Countermeasures |
| Wichtige Felder | `instrumentId`, `classification`, `cia`, `threatModel`, `scenario`, `demo`, `countermeasures`, `blastRadius`, `mitigatesCWE`, `frameworks` |

**Pflegehinweise**

1. Jeder Eintrag braucht ein passendes Instrument in `copilot-instruments.json`.
2. Mermaid-Diagramme muessen lauffaehig bleiben oder landen als Text-Fallback.
3. `frameworks` und `mitigatesCWE` sind nur so gut wie `security-frameworks.json`.

### 3.5 `security-frameworks.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `verificationRequired`, `description`, `sources`, `frameworks`, `cwePattern` |
| Aktuelle Groesse | 6 OWASP-LLM-Eintraege, 6 ATLAS-Eintraege |
| Hauptfunktion | Link-Aufloesung fuer Framework-Chips in Security |
| Besonderheit | enthaelt explizite Hinweise, dass URLs/Titel/Summaries manuell verifiziert werden sollen |

**Pflegehinweise**

1. Diese Datei ist eine Registry, kein abgeschlossener Wahrheitsbeweis.
2. Fehlende IDs werden in der UI als kaputte Chips sichtbar.
3. `cwePattern.urlTemplate` steuert alle CWE-Links.

### 3.6 `terminal-guide.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `description`, `checkIn`, `boardingPass`, `firstFlight`, `departures` |
| Aktuelle Groesse | 5 Plaene, 6 IDEs, 3 Uebungen, 5 Ziele |
| Hauptfunktion | reine Content-Quelle fuer `terminal.html` |
| Risiko | wenig Cross-Referenzen, aber DOM-Abschnitte erwarten volle Sektionen |

### 3.7 `jet-bridge-guide.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `description`, `promptCraft`, `contextManagement`, `editMode`, `agentPatterns`, `nextSteps` |
| Aktuelle Groesse | 6 Techniken, 3 Participants, 3 Variables, 3 Workflows, 5 Patterns, 5 Ziele |
| Hauptfunktion | page-lokaler Lernkatalog fuer `jet-bridge.html` |
| Risiko | stark section-orientiertes Rendering, daher eher struktur- als typo-empfindlich |

### 3.8 `preflight-checklist.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `description`, `intro`, `categories` |
| Aktuelle Groesse | 6 Kategorien, 22 Items |
| Hauptfunktion | Checkliste mit lokaler Persistenz |
| Kritisch | `items[].id` wird direkt als Persistenz-ID verwendet |

### 3.9 `known-changelog-entries.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `entryTypes`, `entries` |
| Aktuelle Groesse | 6 Entry-Typen, 35 Eintraege |
| Hauptfunktion | Flight Log und globaler Suchindex |
| Kritisch | `entries[].instruments[]` muss auf existierende Instrumente zeigen |

### 3.10 `sovereign-cloud.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `description`, `sovereignPillars`, `deploymentOptions`, `residualRisks`, `providerStrategies` |
| Aktuelle Groesse | 3 Pillars, 6 Deployment Options, 3 Residual Risks, 4 Provider Strategies |
| Hauptfunktion | mehrere Tower-Sektionen auf einmal |
| Kritisch | Option-IDs treiben `#sovereign=<id>`-Deep-Links |

### 3.11 `wiring-diagram.json`

| Aspekt | Details |
|---|---|
| Top-Level-Keys | `version`, `lastUpdated`, `description`, `intro`, `connectionTypes`, `connections` |
| Aktuelle Groesse | 4 Verbindungstypen, 55 Verbindungen |
| Hauptfunktion | Mermaid-Graph und Statistik in `wiring.html` |
| Kritisch | `connectionTypes` steuern Filter und Kantenstil; `from`/`to` muessen auf Instrumente zeigen |

## 4. Datenbeziehungen, die man nicht uebersehen sollte

| Von | Nach | Art der Beziehung |
|---|---|---|
| `security-threats.json` | `copilot-instruments.json` | `instrumentId` |
| `known-changelog-entries.json` | `copilot-instruments.json` | `instruments[]` |
| `wiring-diagram.json` | `copilot-instruments.json` | `from`, `to` |
| `tower.html` | `copilot-models.json` | Flight Plans und Modellanzeigen |
| `security-threats.json` | `security-frameworks.json` | OWASP-/ATLAS-/CWE-Aufloesung |

## 5. Pflegerisiken nach Dateityp

| Dateityp | Typisches Risiko | Minimale Gegenmassnahme |
|---|---|---|
| Hub-Kataloge | ID-Bruch mit Seiteneffekten | betroffene Seitenspec plus `integrity.spec.js` |
| Registry-Dateien | kaputte externe Links oder fehlende Keys | Zielseite oeffnen und Badge-/Link-Rendering pruefen |
| Guide-Dateien | Layout bleibt technisch heil, wird aber inhaltlich inkonsistent | DOM-Render der Zielseite plus inhaltlicher Review |
| Diagrammdateien | Mermaid oder Querverweise brechen | Zielseite plus Deep-Link-/Graph-Check |

## 6. Pflege-Checkliste

1. Erst klaeren, ob die Aenderung eine **ID**, nur **Copy** oder **Struktur** betrifft.
2. Bei ID-Aenderungen immer nach Konsumenten in mehreren Seiten suchen.
3. Bei `verificationRequired`-Katalogen keine zu starken Gewissheiten in README oder Doku formulieren.
4. Nach JSON-Aenderungen mindestens die Zielseite und `tests\integrity.spec.js` einplanen.

Weiterfuehrend: [`API-REFERENCE.md`](API-REFERENCE.md), [`TESTING-GUIDE.md`](TESTING-GUIDE.md), [`OPERATIONS.md`](OPERATIONS.md)
