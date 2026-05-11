# Testing Guide

## 1. Test-Stack

Das Repository verwendet ausschliesslich Playwright.

| Konfiguration | Quelle | Wert |
| --- | --- | --- |
| Test-Runner | `C:\temp\copilot-cockpit\package.json` | `npm test` -> `npx playwright test` |
| Testverzeichnis | `C:\temp\copilot-cockpit\playwright.config.js` | `./tests` |
| Browserprojekt | `C:\temp\copilot-cockpit\playwright.config.js` | `chromium` |
| Base URL | `C:\temp\copilot-cockpit\playwright.config.js` | `http://localhost:3000` |
| Lokaler Server | `C:\temp\copilot-cockpit\playwright.config.js` | `python3 -m http.server 3000 --bind 127.0.0.1` |
| Reporter | `C:\temp\copilot-cockpit\playwright.config.js` | `list` |

## 2. Testbefehle

Aus dem Quell-Repository `C:\temp\copilot-cockpit`:

```bash
npm test
```

Einzelspezifikation:

```bash
npx playwright test tests/tower.spec.js
```

Browserinstallation bei neuer Umgebung:

```bash
npx playwright install chromium
```

## 3. Spezifikationsinventar

Die folgenden Zahlen stammen aus den eingecheckten Testdateien (`test(`-Vorkommen):

| Spezifikation | Dateipfad | Testfaelle |
| --- | --- | ---: |
| Cockpit | `C:\temp\copilot-cockpit\tests\cockpit.spec.js` | 29 |
| Flight Log | `C:\temp\copilot-cockpit\tests\flight-log.spec.js` | 15 |
| Integritaet | `C:\temp\copilot-cockpit\tests\integrity.spec.js` | 9 |
| Jet Bridge | `C:\temp\copilot-cockpit\tests\jet-bridge.spec.js` | 17 |
| Pre-Flight | `C:\temp\copilot-cockpit\tests\preflight.spec.js` | 13 |
| Ramp | `C:\temp\copilot-cockpit\tests\ramp.spec.js` | 15 |
| Runway | `C:\temp\copilot-cockpit\tests\runway.spec.js` | 31 |
| Security | `C:\temp\copilot-cockpit\tests\security.spec.js` | 37 |
| Terminal | `C:\temp\copilot-cockpit\tests\terminal.spec.js` | 17 |
| Tower | `C:\temp\copilot-cockpit\tests\tower.spec.js` | 25 |
| Wiring | `C:\temp\copilot-cockpit\tests\wiring.spec.js` | 14 |
| **Summe** |  | **222** |

## 4. Was die Tests absichern

### 4.1 Seiten-Rendering

Alle Perspektivseiten pruefen mindestens:

- seitenweiter Boot ohne relevante JS-Fehler
- zentrale Landmarken und aktive Navigationslinks
- rendering der aus JSON geladenen Kernelemente

### 4.2 Hash-Kontrakte

| Vertrag | Testdateien |
| --- | --- |
| `#instrument-<id>` | `cockpit.spec.js`, `ramp.spec.js` |
| `#model-<id>` | `runway.spec.js` |
| `#scan=<id>` | `security.spec.js` |
| `#control=<id>`, `#sovereign=<id>` | `tower.spec.js` |
| Instrument-Deep-Links aus Changelog/Wiring | `flight-log.spec.js`, `wiring.spec.js` |

### 4.3 Client-Persistenz

| Key | Testdateien |
| --- | --- |
| `cockpit-theme` | `cockpit.spec.js`, `security.spec.js`, `flight-log.spec.js` |
| `cockpit-last-scan` | `security.spec.js` |
| `cockpit-security-posture` | `security.spec.js` |
| `copilot-preflight` | `preflight.spec.js` |

### 4.4 Datenintegritaet

`tests\integrity.spec.js` ist der wichtigste technische Datenvertrag. Abgesichert werden:

- Referenzen von Changelog-Eintraegen auf Instrumente
- Referenzen von Wiring-Kanten auf Instrumente/Controls
- `relatedInstruments`
- Pflichtfelder pro Instrument
- gueltige Zonen
- gueltige Entry- und Connection-Typen
- doppelte IDs in Instrumenten- und Modellkatalog

## 5. Testdurchfuehrung in dieser Umgebung

Ein frischer Lauf wurde versucht, konnte aber nicht abgeschlossen werden:

- Aufruf: `npm test`
- Ergebnis: nicht ausgefuehrt
- Ursache: `pwsh.exe` / PowerShell Core fehlt in der aktuellen Umgebung

Folge fuer diese Dokumentation:

- Die Teststruktur und die Vertragsabdeckung sind belastbar dokumentiert.
- Ein aktueller PASS/FAIL-Status des kompletten Suites ist **nicht** belegbar.

## 6. Praktische Hinweise fuer lokale Reproduktion

1. Stelle sicher, dass `python3` verfuegbar ist, da Playwright den lokalen Static-Server darueber startet.
2. Stelle sicher, dass Playwright Chromium installiert hat.
3. Fuehre Tests im Repository-Root `C:\temp\copilot-cockpit` aus.
4. Bei Deep-Link-Fehlern zuerst die Hash-Vertraege und die Datenreferenzen in `tests\integrity.spec.js` pruefen.

## 7. Nicht durch Playwright abgedeckte Bereiche

Folgende Bereiche sind zwar vorhanden, aber nicht als eigenstaendige Produktions-Tests im Repository erkennbar:

- `tools\enrich\*` als Offline-Pipeline
- GitHub-Workflow `C:\temp\copilot-cockpit\.github\workflows\record-demos.yml`
- Shell-Skript `C:\temp\copilot-cockpit\tools\record-demo.sh`

Diese Bereiche sollten separat verifiziert werden, wenn sie geaendert werden.

## 8. Annahmen

1. **Die 222 Testfaelle sind der aktuelle Stand der eingecheckten Suite.** Diese Zahl stammt aus den Testdateien selbst, nicht aus einem Live-Runner-Report.
2. **Umgebungsfehler sind von Repository-Fehlern zu trennen.** Das hier beobachtete Problem (`pwsh.exe` fehlt) blockiert die Ausfuehrung, sagt aber nichts ueber die inhaltliche Korrektheit der Tests oder der Anwendung aus.
