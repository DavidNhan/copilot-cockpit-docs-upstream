# Testing Guide

## 1. Teststrategie des Repositories

Das Quell-Repo nutzt **nur Playwright** als automatisierte Testbasis. Es gibt in `package.json` keine separaten Build- oder Lint-Skripte; die technische Absicherung konzentriert sich damit auf:

1. Seiten-Boot und Rendering
2. Hash-/Deep-Link-Vertraege
3. Datenintegritaet zwischen JSON-Katalogen
4. einige Integrationsbruecken zwischen Perspektiven

## 2. Verifizierte Runner-Konfiguration

| Einstellung | Quelle | Wert |
| --- | --- | --- |
| Standardbefehl | `package.json` | `npm test` -> `npx playwright test` |
| Testverzeichnis | `playwright.config.js` | `./tests` |
| Browser | `playwright.config.js` | `chromium` |
| Base URL | `playwright.config.js` | `http://localhost:3000` |
| Lokaler Server | `playwright.config.js` | `python3 -m http.server 3000 --bind 127.0.0.1` |
| Reporter | `playwright.config.js` | `list` |
| Retries in CI | `playwright.config.js` | `2` |

## 3. Standardbefehle

Aus `C:\temp\copilot-cockpit`:

```bash
npm test
```

Einzelspezifikation:

```bash
npx playwright test tests/tower.spec.js
```

Falls Chromium lokal noch nicht installiert ist:

```bash
npx playwright install chromium
```

## 4. Suite-Inventar

Die Testfallzahlen unten sind direkt aus den eingecheckten `test(`-Aufrufen abgeleitet.

| Spezifikation | Testfaelle | Fokus |
| --- | ---: | --- |
| `tests\cockpit.spec.js` | 29 | Cockpit-Grid, Detailpanel, Theme, Instrument-Deep-Links |
| `tests\flight-log.spec.js` | 15 | Changelog-Rendering, Instrument-Links |
| `tests\integrity.spec.js` | 9 | JSON-Referenzen, Pflichtfelder, Duplikate, Zonen, Typen |
| `tests\jet-bridge.spec.js` | 17 | Jet-Bridge-Seitenstruktur und Guide-Rendering |
| `tests\preflight.spec.js` | 13 | Checkliste, Progress-State, Persistenz |
| `tests\ramp.spec.js` | 15 | Ramp-Rendering und `#instrument-<id>` |
| `tests\runway.spec.js` | 31 | Modellkatalog, Blade, Topologie, NOTAMs, `#model-<id>` |
| `tests\security.spec.js` | 37 | Security-Scanner, Threat-Links, Persistenz, Cockpit-Bridge |
| `tests\terminal.spec.js` | 17 | Terminal-Seitenstruktur und Guide-Rendering |
| `tests\tower.spec.js` | 25 | Controls, Sovereign Cloud, Flight Plans, `#control=`, `#sovereign=` |
| `tests\wiring.spec.js` | 14 | Wiring-Graph, Diagramm, Links |
| **Summe** | **222** |  |

## 5. Welche Aenderung welche Tests triggert

| Geaenderter Bereich | Relevante Spezifikationen |
| --- | --- |
| `index.html`, `app.js` | `cockpit.spec.js`, plus Bruecken nach `security.spec.js`, `tower.spec.js`, `runway.spec.js` |
| `search.js` | betroffene Zielseiten-Smokes, Deep-Link-Verhalten, Suchziele manuell mitpruefen |
| `data\copilot-instruments.json` | `cockpit.spec.js`, `ramp.spec.js`, `security.spec.js`, `wiring.spec.js`, `flight-log.spec.js`, `integrity.spec.js` |
| `data\copilot-models.json` | `runway.spec.js`, `tower.spec.js`, `integrity.spec.js` |
| `data\governance-controls.json` | `tower.spec.js`, `integrity.spec.js`, Cockpit-Governance-Bridge |
| `data\security-threats.json` / `data\security-frameworks.json` | `security.spec.js`, Cockpit-Security-Bridge |
| `data\known-changelog-entries.json` | `flight-log.spec.js`, `integrity.spec.js`, Search-Ziele |
| `data\wiring-diagram.json` | `wiring.spec.js`, `integrity.spec.js` |
| `data\preflight-checklist.json` | `preflight.spec.js` |
| `data\terminal-guide.json` | `terminal.spec.js` |
| `data\jet-bridge-guide.json` | `jet-bridge.spec.js` |

## 6. Datenintegritaet als eigener Testlayer

`tests\integrity.spec.js` ist der wichtigste nicht-visuelle Vertragstest. Er prueft:

- gueltige Instrument-Referenzen aus dem Changelog
- gueltige Endpunkte in Wiring-Kanten
- gueltige `relatedInstruments`
- eindeutige Instrument- und Modell-IDs
- Pflichtfelder pro Instrument
- gueltige Zonen
- gueltige Changelog-Typen
- gueltige Wiring-Typen

Wenn eine Datenaenderung mehrere Seiten treffen kann, sollte `integrity.spec.js` immer zu den ersten Revalidierungen gehoeren.

## 7. Lokal vorausgesetzte Umgebung

| Voraussetzung | Warum |
| --- | --- |
| Node.js / npm | benoetigt fuer Playwright-Runner |
| `python3` auf PATH | Playwright startet den lokalen Static Server genau damit |
| installierter Chromium-Browser fuer Playwright | einziges konfiguriertes Browserprojekt |

Der lokale Testpfad ist also **statisches Serving plus Browser-Automation**, nicht Build + Test.

## 8. Was ein Fehlschlag typischerweise bedeutet

| Fehlbild | Naheliegende Ursache |
| --- | --- |
| Hash-Tests schlagen fehl | ID-Drift, geaenderte History-/Hash-Logik oder falscher Linkaufbau |
| Wiring-/Integrity-Tests schlagen fehl | kaputte Referenzen zwischen JSON-Dateien |
| Runway-/Tower-Counts schlagen fehl | Modell-/Control-Katalog oder Filter-/Renderlogik geaendert |
| Security-Tests schlagen fehl | Inkonsistenz zwischen Instrumenten, Threats und Frameworks |
| Theme-/Persistenztests schlagen fehl | `localStorage`-Key oder Toggle-Logik geaendert |

## 9. Nicht abgedeckte oder nur indirekt abgedeckte Bereiche

| Bereich | Status |
| --- | --- |
| `tools\enrich\*` | keine eigene Test-Suite im Repo sichtbar |
| Deployment auf Vercel | keine separate Deploy-Testdefinition im Repo sichtbar |
| Demo-Aufnahme-Workflow | kein Teil der Playwright-Suite |
| Docs in diesem Ziel-Repo | keine repo-lokalen Docs-Checks im Quell-Repo deklariert |

## 10. Review-Empfehlung fuer Maintainer

1. Starte bei Datenaenderungen mit `integrity.spec.js` und der betroffenen Seitenspezifikation.
2. Bei Hash- oder Suchaenderungen immer mindestens einen realen Deep Link pruefen.
3. Bei Aenderungen an `verificationRequired`-Katalogen nicht nur UI, sondern auch Formulierung in Docs und Banner-Texte reviewen.

Weiterfuehrend: [API-REFERENCE.md](API-REFERENCE.md), [DATA-CATALOG.md](DATA-CATALOG.md), [OPERATIONS.md](OPERATIONS.md)
