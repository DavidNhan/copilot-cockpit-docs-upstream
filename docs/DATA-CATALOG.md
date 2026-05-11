# Data Catalog

## 1. Datenlandschaft in einem Satz

Alle produktiven Inhalte der Site liegen als statische JSON-Dateien unter `C:\temp\copilot-cockpit\data\`. Die wichtigste Wartungsfrage ist nicht "welche Seite zeigt das an?", sondern **welche IDs und Querverweise haengen daran?**

## 2. Hub-Kataloge mit hoher Kopplung

| Datei | Top-Level-Vertrag | Konsumenten | Kritische Kopplungen | Verifikationsstatus |
| --- | --- | --- | --- | --- |
| `copilot-instruments.json` | `zones[]`, `plans[]`, `instruments[]` | Cockpit, Ramp, Security, Wiring, Search | Instrument-IDs werden von Changelog, Wiring, `relatedInstruments`, Search und Deep Links genutzt. | keine globale `verificationRequired`-Flagge |
| `copilot-models.json` | `capabilities[]`, `surfaces[]`, `plans[]`, `models[]`, `notams[]`, `copilotEngine`, `flightPlans[]` | Runway, Tower, Cockpit-EICAS, Search | Modell-IDs und Verfuegbarkeiten wirken in Runway, Search und Cockpit-Bridge. | `verificationRequired: true` |
| `governance-controls.json` | `sources`, `controls[]` | Tower, Cockpit, Search, Wiring-Integritaet | Control-IDs sind Hash-Ziele und duerfen in `wiring-diagram.json` vorkommen. | `verificationRequired: false` |
| `known-changelog-entries.json` | `entryTypes`, `entries[]` | Flight Log, Search, Integritaetstests | `entries[].instruments[]` muessen auf bekannte Instrumente zeigen. | keine globale Flagge |
| `wiring-diagram.json` | `connectionTypes[]`, `connections[]`, `zoneDescriptions` | Wiring, Integritaetstests | `connections[].from/to` muessen Instrumente oder Controls adressieren. | keine globale Flagge |

Diese Dateien sollten vor jeder strukturellen Aenderung zusammen mit [TESTING-GUIDE.md](TESTING-GUIDE.md) betrachtet werden.

## 3. Fachkataloge mit mittlerer Kopplung

| Datei | Rolle | Konsumenten | Wartungshinweis |
| --- | --- | --- | --- |
| `security-threats.json` | Threat-Modelle, Gegenmassnahmen und Demo-Snippets pro Instrument | Security, Cockpit | `threats[].instrumentId` muss auf ein existierendes Instrument verweisen. |
| `security-frameworks.json` | Framework-/CWE-Mapping fuer Security | Security | Viele Eintraege sind explizit `verified: false`; keine staerkeren Aussagen als die Datei selbst machen. |
| `sovereign-cloud.json` | Deploymentoptionen, Restrisiken und Mermaid-Daten fuer Tower | Tower | Enthaelt strukturierte Inhalte **und** Diagrammquelle; Syntax- und Inhaltsfehler wirken direkt in der UI. |

## 4. Page-spezifische Kataloge mit geringer Kopplung

| Datei | Konsument | Stabiler Vertrag | Pflegefalle |
| --- | --- | --- | --- |
| `terminal-guide.json` | `terminal.html` | `checkIn`, `boardingPass`, `firstFlight`, `departures` | Linkziele in `departures` muessen auf echte Seiten zeigen. |
| `jet-bridge-guide.json` | `jet-bridge.html` | `promptCraft`, `contextManagement`, `editMode`, `agentPatterns`, `nextSteps` | Intra-page-Struktur ist relativ lokal; kaputte Arrays wirken aber sofort auf das Rendering. |
| `preflight-checklist.json` | `preflight.html` | `intro`, `categories[]` | Fortschritt lebt in `localStorage`, nicht in der Datei selbst. |

## 5. Verifizierte Integritaetsregeln aus `tests\integrity.spec.js`

Die folgenden Regeln sind **harte Datenvertraege**, nicht nur redaktionelle Empfehlungen:

| Vertrag | Quelle |
| --- | --- |
| jede `entries[].instruments[]`-Referenz zeigt auf ein existierendes Instrument | `known-changelog-entries.json` + `copilot-instruments.json` |
| jede Wiring-Kante referenziert ein existierendes Instrument oder eine existierende Control | `wiring-diagram.json` + `copilot-instruments.json` + `governance-controls.json` |
| jede `relatedInstruments`-Referenz zeigt auf ein existierendes Instrument | `copilot-instruments.json` |
| Instrument-IDs sind eindeutig | `copilot-instruments.json` |
| Modell-IDs sind eindeutig | `copilot-models.json` |
| jedes Instrument besitzt `id`, `symbol`, `name`, `zone`, `status` | `copilot-instruments.json` |
| jedes Instrument nutzt eine definierte Zone | `copilot-instruments.json` |
| jeder Changelog-Typ ist in `entryTypes` definiert | `known-changelog-entries.json` |
| jeder Wiring-Typ ist in `connectionTypes` definiert | `wiring-diagram.json` |

## 6. Dateispezifische Wartungshinweise

### 6.1 `copilot-instruments.json`

Relevanz:

- primaerer Inhalts- und ID-Hub
- einzige Quelle fuer Cockpit-Zonen und Instrumentkarten
- Referenzquelle fuer Ramp, Security, Wiring und Search

Pflegehinweise:

1. Neue Instrumente brauchen stabile `id`, `zone` und `status`.
2. `relatedInstruments` duerfen nur existierende IDs verwenden.
3. Aenderungen an `planAvailability`, `perspectives` oder `flightMode` wirken direkt auf Filter- und Sichtbarkeitslogik.
4. Aenderungen an governance-relevanten Instrumenten koennen Cockpit-Callouts nach `tower.html#control=<id>` beeinflussen.

### 6.2 `copilot-models.json`

Relevanz:

- treibt Runway-Filter, Departure Board, Detail-Blade, NOTAMs und Flight Plans
- liefert Zusatzdaten fuer Tower und EICAS-Cluster im Cockpit

Pflegehinweise:

1. Datei ist selbst als **nicht final verifiziert** markiert; behandle Modellmetadaten entsprechend vorsichtig.
2. `models[].id` ist Deep-Link-Ziel fuer `#model-<id>`.
3. `status: deprecated` wirkt sichtbar in Runway-Filtern und Status-LEDs.
4. `planAvailability`, `surfaceAvailability` und `provider` haben direkte UI-Folgen.

### 6.3 `governance-controls.json`

Relevanz:

- steuert Tower-Control-Liste und Compliance-Chips
- liefert IDs fuer `#control=<id>`
- dient als moeglicher Endpunkt im Wiring-Graph

Pflegehinweise:

1. `controls[].id` muss stabil bleiben; Hash-Links und Tests haengen daran.
2. `sources` definieren den Legendeninhalt fuer Tower.
3. `verificationRequired: false` ist ein harter Unterschied zu Modellen und Frameworks; nicht angleichen ohne Quellbeleg.

### 6.4 `security-threats.json`

Relevanz:

- steuert Security-Scanner, Risikoszenarien und Demo-Snippets
- speist in `app.js` einen Scanner-Index fuer Cockpit-Callouts

Pflegehinweise:

1. `instrumentId` ist der wichtigste Vertrag.
2. Mermaid-Inhalte unter `threatModel.diagram` muessen renderbar bleiben.
3. `mitigatesCWE[]` und `frameworks` sollten nur Werte verwenden, die zur Framework-Datei passen; das ist logisch notwendig, auch wenn nicht alles per Test abgesichert ist.

### 6.5 `known-changelog-entries.json`

Pflegehinweise:

1. `entryTypes` ist der Typ-Kanon fuer `entries[].type`.
2. Instrument-Links im Flight Log werden direkt aus `entries[].instruments[]` gebaut.
3. Search indexiert die Datei mit; Titel und Beschreibung beeinflussen damit auch Suchtreffer.

### 6.6 `wiring-diagram.json`

Pflegehinweise:

1. Jede Kante braucht gueltige `from`, `to` und `type` Werte.
2. `connectionTypes[]` definiert nicht nur Labels, sondern die erlaubten Kantenklassen.
3. Die Seite erzeugt klickbare Mermaid-Knoten nach `index.html#instrument-<id>`.

## 7. Datenquellen ausserhalb von `data\`

| Quelle | Rolle |
| --- | --- |
| `tools\enrich\README.md` | beschreibt die Offline-Anreicherung fuer Modellkataloge |
| `tools\enrich\sources.yml` | registriert Upstream-Quellen fuer Modellanreicherung |
| `vercel.json` | bestimmt Cache-Verhalten fuer JSON-Auslieferung |

Wichtig: Laut `tools\enrich\README.md` ist die Enrichment-Pipeline **nicht** als direkter Writer nach `data\` gedacht. Modellkataloge bleiben damit statische Artefakte des Repos.

## 8. Aenderungsentscheidungen nach Katalogtyp

| Wenn du aenderst ... | Denke zusaetzlich an ... |
| --- | --- |
| ID-Felder | Search, Hashes, Wiring, Changelog, `relatedInstruments`, Playwright |
| Status-/Plan-/Provider-Felder | Filterlogik, Sichtbarkeit, LED-/Badge-Zustaende |
| Mermaid-Strings | Rendering in Security, Tower oder Wiring |
| `verificationRequired` | Dokumentationssprache und Banner/Warnings in der UI |
| Link-/URL-Felder | externe Zielgueltigkeit und Review-Qualitaet |

## 9. Annahmen

1. **JSON ist der kanonische Vertragsstand.** Wenn Freitext-Kommentare oder README-Passagen aeltere Zaehler nennen, sollte fuer technische Wartung immer die aktuelle JSON-Struktur priorisiert werden.
2. **Framework-Konsistenz ist fachlich erwartet, aber nicht vollstaendig testseitig erzwungen.** Gerade fuer `security-frameworks.json` braucht es deshalb Review-Disziplin zusaetzlich zur vorhandenen Testbasis.
