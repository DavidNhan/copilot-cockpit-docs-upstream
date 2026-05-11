# Testing Guide

## 1. Stack

Das Projekt verwendet **ausschliesslich Playwright** fuer automatisierte Tests.

| Bestandteil | Wert |
|---|---|
| Framework | `@playwright/test` |
| Konfigurationsdatei | `playwright.config.js` |
| Testverzeichnis | `tests\` |
| NPM-Script | `npm test` |
| Reporter | `list` |
| Browser-Projekt | `chromium` |

`package.json` definiert kein Build-, Lint- oder Dev-Script; die Tests sind der einzige automatisierte Standard-Entry-Point.

## 2. Startverhalten

`npm test` expandiert zu:

```bash
npx playwright test
```

Playwright startet dafuer automatisch einen statischen Webserver:

```js
webServer: {
    command: 'python3 -m http.server 3000 --bind 127.0.0.1',
    port: 3000,
    reuseExistingServer: !process.env.CI,
}
```

### Implikationen

1. Das Projekt wird wie eine **statische Website** getestet.
2. Ein Node- oder App-Server ist nicht erforderlich.
3. `python3` muss verfuegbar sein.

## 3. Playwright-Konfiguration

| Setting | Wert |
|---|---|
| `testDir` | `./tests` |
| `fullyParallel` | `true` |
| `forbidOnly` | in CI aktiv |
| `retries` | `2` in CI, sonst `0` |
| `workers` | `1` in CI, lokal Standard |
| `baseURL` | `http://localhost:3000` |
| `trace` | `on-first-retry` |
| `screenshot` | `only-on-failure` |

## 4. Coverage nach Spec-Datei

> Hinweis: Die Root-`README.md` nennt 222 Tests. Die aktuellen Spec-Dateien summieren sich jedoch auf **219 Tests**. Fuer die aktuelle Suite sollte daher die Testbasis in `tests\*.spec.js` als maßgeblich gelten.

| Spec | Fokus | Kernaussagen |
|---|---|---|
| `tests/cockpit.spec.js` | Cockpit-Grid, Blade, Filter, Suche, Theme | prueft Rendering, Zonen, Tabs, Deep Links, Media, Prism |
| `tests/flight-log.spec.js` | Flight Log | Timeline, Jahrgruppierung, Filter, Navigation, Theme |
| `tests/integrity.spec.js` | JSON-Integritaet | Node-only-Checks auf Referenzen und Duplikate |
| `tests/jet-bridge.spec.js` | Jet Bridge | Prompt-Cards, Kontextkarten, Edit-Workflows, Agent-Patterns |
| `tests/preflight.spec.js` | Pre-Flight | Kategorien, Checkboxen, LocalStorage, Fortschritt, Reset |
| `tests/ramp.spec.js` | Ramp | Karten, Blade, Hash-Navigation, Metaphern-Key |
| `tests/runway.spec.js` | Runway | Filterbar, Departure Board, Model Blade, Mermaid, NOTAMs |
| `tests/security.spec.js` | Security | X-Ray-Scanner, Threat-Diagramme, Framework-Chips, Posture |
| `tests/terminal.spec.js` | Terminal | Plaene, IDEs, Uebungen, Weiterleitungslinks |
| `tests/tower.spec.js` | Tower | Frameworks, Controls, Sovereignty, Flight Plans |
| `tests/wiring.spec.js` | Wiring | Mermaid-Graph, Filter, Legende, Zonen, Statistiken |

## 5. Testzahlen

| Spec | Anzahl |
|---|---:|
| `cockpit.spec.js` | 29 |
| `flight-log.spec.js` | 15 |
| `integrity.spec.js` | 9 |
| `jet-bridge.spec.js` | 17 |
| `preflight.spec.js` | 13 |
| `ramp.spec.js` | 15 |
| `runway.spec.js` | 30 |
| `security.spec.js` | 35 |
| `terminal.spec.js` | 17 |
| `tower.spec.js` | 25 |
| `wiring.spec.js` | 14 |
| **Gesamt** | **219** |

## 6. Was die Suite besonders gut absichert

### 6.1 Seitenverhalten

- Initial Rendering fast aller HTML-Seiten
- aktive Nav-Links
- Sichtbarkeit zentraler DOM-Strukturen
- Fehlerfreiheit im Browser-Console-Output

### 6.2 Interaktive UI-Muster

- Blade/Open-Close-Mechanik
- Filter-Logik
- Hash-basierte Deep Links
- Theme-Toggle und Persistenz
- LocalStorage-basierte Progress-/State-Logik

### 6.3 Datenintegritaet

`integrity.spec.js` prueft fachliche Referenzen direkt gegen Dateien im Dateisystem:

```js
for (const conn of wiring.connections) {
    if (!allIds.has(conn.from)) invalid.push(...);
    if (!allIds.has(conn.to)) invalid.push(...);
}
```

Das ist wichtig, weil das Repo stark von Cross-References zwischen JSON-Dateien lebt.

## 7. Beobachtbare Luecken

Die Tests sind stark, aber nicht vollkommen symmetrisch:

1. `integrity.spec.js` prueft nicht jede einzelne JSON-Datei mit demselben Tiefengrad.
2. Guide-Dateien wie `terminal-guide.json` und `jet-bridge-guide.json` werden primär ueber DOM-Rendering abgesichert.
3. Externe CDN-Abhaengigkeiten werden indirekt mitgetestet, aber nicht isoliert contract-basiert.
4. Wegen statischer Architektur gibt es keine separaten Unit-Tests fuer Renderfunktionen in Inline-Scripts.

## 8. Nützliche Einzelkommandos

```bash
# Gesamte Suite
npm test

# Einzelne Seite pruefen
npx playwright test tests/tower.spec.js
npx playwright test tests/security.spec.js
npx playwright test tests/runway.spec.js

# Nur Datenintegritaet
npx playwright test tests/integrity.spec.js
```

## 9. Teststrategie fuer Dokumentations- und Datenaenderungen

### Bei JSON-Aenderungen

Mindestens sinnvoll:

- `tests/integrity.spec.js`
- die betroffene Seitenspec, z. B.:
  - `tests/runway.spec.js` fuer `copilot-models.json`
  - `tests/security.spec.js` fuer `security-threats.json`
  - `tests/tower.spec.js` fuer `governance-controls.json` oder `sovereign-cloud.json`

### Bei HTML-/JS-Aenderungen

Mindestens sinnvoll:

- betroffene Seitenspec
- `tests/cockpit.spec.js`, falls `app.js`, `search.js` oder gemeinsame Navigation betroffen ist

## 10. Fazit

Die Test-Suite ist fuer ein rein statisches Repo ungewoehnlich tief: Sie sichert nicht nur Rendering, sondern auch **Deep-Link-Vertraege, Persistenz, Datenreferenzen und wichtige fachliche Integrationen**. Gerade fuer dieses Projekt ist das entscheidend, weil Inhalt und Laufzeitverhalten direkt aus JSON-Katalogen erzeugt werden.
