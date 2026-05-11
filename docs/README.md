# Copilot Cockpit - Technische Repository-Dokumentation

## Geltungsbereich

Diese Dokumentation beschreibt den technischen Stand des Quell-Repositories unter `C:\temp\copilot-cockpit`. Sie wurde bewusst ausserhalb des Quell-Repositories abgelegt und schreibt nur nach `C:\temp\copilot-cockpit-docs-final\docs`.

## Systemueberblick

`copilot-cockpit` ist eine statische Multi-Page-Webanwendung ohne Build-Schritt und ohne serverseitige Laufzeitlogik. Die Laufzeit besteht aus HTML-Einstiegspunkten, gemeinsam genutztem CSS/JavaScript und JSON-Katalogen unter `data\`.

| Bereich | Dateipfade | Zweck |
| --- | --- | --- |
| Einstiegspunkte | `C:\temp\copilot-cockpit\index.html`, `terminal.html`, `security.html`, `jet-bridge.html`, `ramp.html`, `runway.html`, `tower.html`, `flight-log.html`, `preflight.html`, `wiring.html` | Statische Seiten pro Perspektive |
| Gemeinsame UI-Logik | `C:\temp\copilot-cockpit\app.js`, `search.js`, `styles.css` | Cockpit-Rendering, globale Suche, Styling |
| Datenkataloge | `C:\temp\copilot-cockpit\data\*.json` | Inhaltsquellen fuer alle Perspektiven |
| Tests | `C:\temp\copilot-cockpit\tests\*.spec.js` | Playwright-End-to-End- und Integritaetstests |
| Betrieb/Deployment | `C:\temp\copilot-cockpit\vercel.json` | Header- und Caching-Regeln fuer statisches Hosting |
| Offline-Datenanreicherung | `C:\temp\copilot-cockpit\tools\enrich\*` | Erfasst Upstream-Quellen fuer `data\copilot-models.json`, schreibt nicht direkt in `data\` |

## Seiten- und Datenzuordnung

| Route | Primare Datenquellen | Primare Skripte |
| --- | --- | --- |
| `/` oder `/index.html` | `data\copilot-instruments.json`, optional `data\security-threats.json`, `data\governance-controls.json`, `data\copilot-models.json` | `app.js`, `search.js` |
| `/terminal.html` | `data\terminal-guide.json` | Inline-Skript, `search.js` |
| `/jet-bridge.html` | `data\jet-bridge-guide.json` | Inline-Skript, `search.js` |
| `/ramp.html` | `data\copilot-instruments.json` | Inline-Skript, `search.js` |
| `/runway.html` | `data\copilot-models.json` | Inline-Skript, `search.js` |
| `/tower.html` | `data\governance-controls.json`, `data\copilot-models.json`, `data\sovereign-cloud.json` | Inline-Skript, `search.js` |
| `/security.html` | `data\copilot-instruments.json`, `data\security-threats.json`, `data\security-frameworks.json` | Inline-Skript, `search.js` |
| `/flight-log.html` | `data\known-changelog-entries.json` | Inline-Skript, `search.js` |
| `/preflight.html` | `data\preflight-checklist.json` | Inline-Skript, `search.js` |
| `/wiring.html` | `data\wiring-diagram.json`, `data\copilot-instruments.json` | Inline-Skript, `search.js` |

## Hash-Kontrakte

Die Anwendung nutzt URL-Hashes als stabilen Deep-Link-Vertrag zwischen Seiten und Komponenten.

| Vertrag | Produzent | Konsument | Bedeutung |
| --- | --- | --- | --- |
| `#instrument-<id>` | `app.js`, `ramp.html`, `wiring.html`, `flight-log.html`, `search.js` | Cockpit (`app.js`), Ramp (`ramp.html`) | Oeffnet ein Instrument oder verlinkt auf dessen Detailansicht |
| `#model-<id>` | `runway.html`, `search.js`, Cockpit-EICAS-Links in `app.js` | Runway (`runway.html`) | Oeffnet das Model-Detailblatt |
| `#scan=<id>` | `security.html`, Cockpit-Callout in `app.js` | Security (`security.html`) | Waehlt einen Threat-Scanner-Eintrag |
| `#control=<id>` | `tower.html`, Cockpit-Callout in `app.js`, `search.js` | Tower (`tower.html`) | Hebt eine Governance-Control hervor |
| `#sovereign=<id>` | `tower.html` | Tower (`tower.html`) | Hebt eine Sovereign-Cloud-Option hervor |

## Persistente Client-Zustaende

| Speicher | Key | Seiten |
| --- | --- | --- |
| `localStorage` | `cockpit-theme` | Alle Perspektiven |
| `localStorage` | `cockpit-last-scan` | `security.html` |
| `localStorage` | `cockpit-security-posture` | `security.html` |
| `localStorage` | `copilot-preflight` | `preflight.html` |

## Testbezug

Die automatische Testabdeckung liegt in `C:\temp\copilot-cockpit\tests\`. Es existieren 11 Playwright-Spezifikationen mit insgesamt 222 Testfaellen. Integritaetsvertraege fuer Datenreferenzen liegen in `tests\integrity.spec.js`.

## Annahmen

1. **Frischer Testlauf nicht belegbar:** `npm test` konnte in dieser Umgebung nicht ausgefuehrt werden, weil `pwsh.exe` fehlt. Diese Dokumentation referenziert daher die eingecheckten Testdateien und Konfigurationen, nicht einen frischen gruenen Lauf.
2. **Zaehlangaben aus Daten statt aus Prosa:** Einige Freitexte im Repository nennen unterschiedliche Instrument-Anzahlen. Fuer technische Aussagen sind die JSON-Kataloge und die Testvertraege kanonisch, nicht Marketing-/Einleitungstexte in HTML oder README.
