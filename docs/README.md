# Dokumentationsindex

Diese Dokumentation beschreibt das Quell-Repo `C:\temp\copilot-cockpit` aus Entwickler- und Betreiberperspektive. Fokus ist die **technische Realität**: Seiten, Daten, Fluesse, Tests und Betrieb.

## Schnellueberblick

| Thema | Kernaussage |
|---|---|
| App-Typ | Statische Multi-Page-Anwendung |
| Routing | Dateibasierte Navigation plus Hash-Deep-Links |
| Datenzugriff | Browser-`fetch()` auf `data\*.json` |
| Wiederverwendung | `styles.css`, `search.js`, Theme-Persistenz, kopierte Header/Footer |
| Testen | Playwright E2E plus Integritaetschecks gegen JSON |
| Deploy | Statisches Output-Verzeichnis `.` mit Vercel-Cache-Regeln |

## Leserpfade

### 1. Fuer neue Maintainer

1. [`../README.md`](../README.md)
2. [`ARCHITECTURE.md`](ARCHITECTURE.md)
3. [`DATA-CATALOG.md`](DATA-CATALOG.md)
4. [`TESTING-GUIDE.md`](TESTING-GUIDE.md)

### 2. Fuer Aenderungen an einer konkreten Seite

1. [`API-REFERENCE.md`](API-REFERENCE.md) - Route, DOM-Roots, Deep Links
2. [`DATA-CATALOG.md`](DATA-CATALOG.md) - welche JSON-Dateien die Seite liest
3. [`TESTING-GUIDE.md`](TESTING-GUIDE.md) - welche Specs danach Pflicht sind

### 3. Fuer Architektur- oder Refactoring-Entscheidungen

1. [`ARCHITECTURE.md`](ARCHITECTURE.md)
2. [`API-REFERENCE.md`](API-REFERENCE.md)
3. [`OPERATIONS.md`](OPERATIONS.md)

### 4. Fuer Content-Pflege und Release-Vorbereitung

1. [`DATA-CATALOG.md`](DATA-CATALOG.md)
2. [`OPERATIONS.md`](OPERATIONS.md)
3. [`CONTRIBUTING.md`](CONTRIBUTING.md)
4. [`TESTING-GUIDE.md`](TESTING-GUIDE.md)

## Dokumente im Detail

| Datei | Wofuer sie da ist | Besonders hilfreich wenn ... |
|---|---|---|
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Systembild, Runtime-Fluesse, Komponenten, Seitenschnitt | du verstehen willst, warum die Anwendung nicht wie eine SPA organisiert ist |
| [`API-REFERENCE.md`](API-REFERENCE.md) | Referenz fuer HTML-Routen, DOM-Roots, Hash-Kontrakte, Interaktionen | du wissen musst, wo eine Funktion technisch einhaengt |
| [`DATA-CATALOG.md`](DATA-CATALOG.md) | Vollstaendige JSON-Karte mit Konsumenten und Pflegehinweisen | du Daten aenderst oder neue Inhalte einhaengst |
| [`TESTING-GUIDE.md`](TESTING-GUIDE.md) | Struktur und Taktik der Playwright-Suite | du Aenderungen verifizieren oder Testumfang zuschneiden willst |
| [`OPERATIONS.md`](OPERATIONS.md) | Deployment, Cache-Verhalten, Content-Refresh, Risiken | du die Site betreibst oder ein Problem eingrenzt |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Arbeitsregeln fuer Doku- und Codebeitraege | du konsistent beitragen und sauber reviewen willst |

## Wichtige Grundsaetze

1. **Keine Backend-API erfinden.** Die API-Oberflaeche dieses Repos besteht aus HTML-Seiten, JSON-Dateien, Hash-Kontrakten und externen CDN-Skripten.
2. **Den Hub beachten.** `copilot-instruments.json` ist die wichtigste Referenzdatei fuer mehrere andere Kataloge.
3. **Verifikation nicht ueberspringen.** Teile des Modells- und Framework-Katalogs markieren sich selbst als noch nicht abschliessend verifiziert.
4. **Seiten lokal denken.** Viele Muster wiederholen sich, aber die meisten Perspektiven besitzen ihre eigene Renderlogik.

## Wo die Doku bewusst vorsichtig formuliert

| Thema | Warum vorsichtig? |
|---|---|
| Modell- und Surface-Aussagen | `copilot-models.json` markiert sich selbst als verifikationspflichtig. |
| Sicherheits-Framework-Mappings | `security-frameworks.json` enthaelt explizite Hinweise auf manuelle Nachpruefung. |
| Zaehlerstaende in Kommentaren | Einzelne Tests und Inhaltsdateien enthalten historische Zahlen, die nicht mehr den aktuellen Katalog abbilden. |
