# Testing Guide

## 1. Teststrategie in einem Satz

Das Quell-Repo setzt praktisch komplett auf **Playwright als Integrations- und Seitenvertragstest**, inklusive Browser-Interaktion und datengetriebener Integritaetspruefung.

## 2. Test-Setup

| Bereich | Stand |
|---|---|
| Framework | `@playwright/test` |
| Konfiguration | `playwright.config.js` |
| Testverzeichnis | `tests\` |
| Standardkommando | `npm test` |
| Browser-Projekt | `chromium` |
| Reporter | `list` |
| Base URL | `http://localhost:3000` |
| Webserver | `python3 -m http.server 3000 --bind 127.0.0.1` |

### Was das praktisch bedeutet

1. Getestet wird die Site so, wie sie deployt wird: **als statische Ausgabe**.
2. Ein Node-App-Server existiert nicht.
3. Datenfehler sind genauso wichtig wie UI-Fehler, weil JSON direkt gerendert wird.

## 3. Suite-Struktur

| Spec | Schwerpunkt | Testtyp |
|---|---|---|
| `cockpit.spec.js` | Cockpit-Grid, Blade, Filter, Suche, Theme, Media | Regression |
| `terminal.spec.js` | Onboarding-Content | Smoke/Regression |
| `jet-bridge.spec.js` | Prompt- und Agent-Guide | Smoke/Regression |
| `ramp.spec.js` | Ramp-Perspektive und Blade | Regression |
| `runway.spec.js` | Modellkatalog, Filter, Blade, Topology, NOTAMs | Regression |
| `security.spec.js` | X-Ray Scanner, Framework-Chips, Posture, Cockpit-Bruecke | Regression |
| `tower.spec.js` | Frameworks, Controls, Sovereign Cloud, Flight Plans | Regression |
| `flight-log.spec.js` | Timeline und Filter | Smoke/Regression |
| `preflight.spec.js` | Checklist, Fortschritt, Persistenz | Regression |
| `wiring.spec.js` | Mermaid-Graph, Filter, Legende, Stats | Regression |
| `integrity.spec.js` | Referenzen, Duplikate, Gueltigkeit von Datenbeziehungen | Struktur-/Datenvertrag |

## 4. Aktueller Umfang

Die aktuelle Suite summiert sich auf **222 Tests** und stimmt damit mit der Root-README des Quell-Repos ueberein.

| Spec | Anzahl |
|---|---:|
| `cockpit.spec.js` | 29 |
| `flight-log.spec.js` | 15 |
| `integrity.spec.js` | 9 |
| `jet-bridge.spec.js` | 17 |
| `preflight.spec.js` | 13 |
| `ramp.spec.js` | 15 |
| `runway.spec.js` | 31 |
| `security.spec.js` | 37 |
| `terminal.spec.js` | 17 |
| `tower.spec.js` | 25 |
| `wiring.spec.js` | 14 |
| **Gesamt** | **222** |

## 5. Smoke vs. Regression

Das Repo trennt die Suite nicht technisch in zwei Playwright-Projekte. Fuer den Alltag ist die folgende **arbeitspraktische** Trennung sinnvoll:

### Smoke

Ziel: schnell erkennen, ob eine Seite noch bootet und ihre Kernlandmarks rendert.

Typische Smoke-Kandidaten:

- jeweils der erste "page load" / "page structure"-Block pro Spec
- `integrity.spec.js`, wenn nur JSON-Referenzen betroffen sind
- die betroffene Seitenspec nach einer kleinen lokalen Aenderung

### Regression

Ziel: Interaktionen, Deep Links, Persistenz, Diagramme und Cross-Page-Bruecken mitpruefen.

Typische Regression:

- komplette betroffene Seitenspec
- `cockpit.spec.js`, wenn `app.js`, `search.js`, Navigation oder Theme betroffen ist
- mehrere Seitenspecs, wenn ein Hub-Katalog geaendert wurde

## 6. Was die Suite gut absichert

| Bereich | Beispiele |
|---|---|
| Seiten-Boot | Hauptcontainer, aktive Navigation, fehlende JS-Fehler |
| Deep Links | `#instrument-`, `#model-`, `#scan=`, `#control=`, `#sovereign=` |
| Persistenz | Theme, Preflight-Status, Security-Posture, letzter Scan |
| Datengetriebene UI | Zahlen von Rows, Chips, Kategorien, Optionen |
| Diagramme | Mermaid-Rendering in Security, Runway, Tower, Wiring |
| Cross-Repo-Logik | Integritaet von IDs und Referenzen ueber mehrere JSON-Dateien |

## 7. Was weniger stark abgesichert ist

| Bereich | Einschraenkung |
|---|---|
| Redaktionelle Qualitaet | Texte koennen inhaltlich schwach sein, obwohl DOM-Rendering gruen bleibt |
| Externe Quellen | CDN-Verfuegbarkeit und externe Standard-Links werden nicht als eigener Vertrag isoliert getestet |
| Wiederholte Seitenteile | Da Navigation und Theme mehrfach implementiert sind, bleiben semantische Abweichungen moeglich |
| Fachliche Aktualitaet | Tests pruefen Struktur und UI-Verhalten, nicht Produktwahrheit |

## 8. Aenderung -> empfohlene Validierung

| Aenderung | Mindestens ausfuehren | Warum |
|---|---|---|
| `copilot-instruments.json` | `tests\integrity.spec.js`, `cockpit.spec.js`, plus betroffene Seitenspec(s) | Hub-Datei mit breiter Wirkung |
| `copilot-models.json` | `runway.spec.js`, `tower.spec.js`, optional `cockpit.spec.js`, `integrity.spec.js` | Runway/Tower und Cockpit-EICAS betroffen |
| `governance-controls.json` | `tower.spec.js`, `integrity.spec.js`, ggf. `cockpit.spec.js` | Deep Links und Search-Index betroffen |
| `security-threats.json` oder `security-frameworks.json` | `security.spec.js`, `integrity.spec.js`, ggf. `cockpit.spec.js` | Scanner und Security-Callouts betroffen |
| `wiring-diagram.json` | `wiring.spec.js`, `integrity.spec.js` | Graph und Referenzen betroffen |
| `terminal-guide.json` | `terminal.spec.js` | reiner Content, page-lokal |
| `jet-bridge-guide.json` | `jet-bridge.spec.js` | reiner Content, page-lokal |
| `preflight-checklist.json` | `preflight.spec.js`, `integrity.spec.js` falls IDs referenziert werden | Persistenz und Fortschritt |
| `app.js` | `cockpit.spec.js` plus angrenzende Seitenspecs | zentrale Runtime |
| `search.js` | `cockpit.spec.js`, dazu manuelle Plausibilisierung auf weiteren Seiten | globale Suche wirkt seitenuebergreifend |
| Navigation / Theme / Layout | mehrere Seitenspecs, mindestens eine pro Perspektivenfamilie | Logik ist dupliziert |

## 9. Nützliche Kommandos

```bash
npm test
npx playwright test tests/runway.spec.js
npx playwright test tests/security.spec.js
npx playwright test tests/integrity.spec.js
npx playwright test --grep "page load"
```

## 10. Validierungsreihenfolge fuer typische Changes

### Datenaenderung

1. `tests/integrity.spec.js`
2. direkt betroffene Seitenspec
3. bei Hub-Daten: angrenzende Seitenspecs

### UI-/JS-Aenderung

1. betroffene Seitenspec
2. bei gemeinsamem Pattern: weitere Seiten mit derselben Logik
3. bei Navigation/Theme/Search: Cockpit plus mindestens eine Nicht-Cockpit-Seite

### Doku-Only in diesem Repo

Keine automatische Testpflicht in diesem Doku-Repo. Trotzdem sollten Zahlen, Dateinamen, Pfade und Aussagen gegen `C:\temp\copilot-cockpit` gegengeprueft werden.

Weiterfuehrend: [`DATA-CATALOG.md`](DATA-CATALOG.md), [`OPERATIONS.md`](OPERATIONS.md), [`CONTRIBUTING.md`](CONTRIBUTING.md)
