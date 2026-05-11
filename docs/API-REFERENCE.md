# API-Reference

## 1. Was in diesem Repo "API" bedeutet

Es gibt **keine serverseitige API**. Die technische API-Flaeche dieses Repositories besteht aus:

1. statischen HTML-Routen
2. statischen JSON-Endpunkten unter `data\`
3. URL-Hash-Kontrakten
4. `localStorage`-Keys
5. wenigen globalen Browser-Funktionen

Fuer Architekturkontext siehe [ARCHITECTURE.md](ARCHITECTURE.md).

## 2. HTML-Routen und ihre Vertraege

| Route | Quelldatei | Renderer | Gelesene Daten | Deep Links |
| --- | --- | --- | --- | --- |
| `/` und `/index.html` | `index.html` | `app.js` + `search.js` | `copilot-instruments.json`, optional `security-threats.json`, `governance-controls.json`, `copilot-models.json` | konsumiert `#instrument-<id>` |
| `/terminal.html` | `terminal.html` | Inline-Skript + `search.js` | `terminal-guide.json` | keine page-lokalen Hash-Kontrakte belegt |
| `/jet-bridge.html` | `jet-bridge.html` | Inline-Skript + `search.js` | `jet-bridge-guide.json` | keine page-lokalen Hash-Kontrakte belegt |
| `/ramp.html` | `ramp.html` | Inline-Skript + `search.js` | `copilot-instruments.json` | konsumiert und erzeugt `#instrument-<id>` |
| `/runway.html` | `runway.html` | Inline-Skript + `search.js` | `copilot-models.json` | konsumiert und erzeugt `#model-<id>` |
| `/tower.html` | `tower.html` | Inline-Skript + `search.js` | `governance-controls.json`, `copilot-models.json`, `sovereign-cloud.json` | konsumiert `#control=<id>` und `#sovereign=<id>` |
| `/security.html` | `security.html` | Inline-Skript + `search.js` | `copilot-instruments.json`, `security-threats.json`, `security-frameworks.json` | konsumiert und erzeugt `#scan=<id>` |
| `/flight-log.html` | `flight-log.html` | Inline-Skript + `search.js` | `known-changelog-entries.json` | erzeugt Links nach `index.html#instrument-<id>` |
| `/preflight.html` | `preflight.html` | Inline-Skript + `search.js` | `preflight-checklist.json` | keine Hash-Kontrakte belegt; nutzt `localStorage` fuer Fortschritt |
| `/wiring.html` | `wiring.html` | Inline-Skript + `search.js` | `wiring-diagram.json`, `copilot-instruments.json` | erzeugt Links nach `index.html#instrument-<id>` |

## 3. JSON-Endpunkte

| Endpunkt | Primaere Konsumenten | Kernvertrag | Wartungshinweis |
| --- | --- | --- | --- |
| `data\copilot-instruments.json` | Cockpit, Ramp, Security, Wiring, Search | `zones[]`, `plans[]`, `instruments[]` | Instrument-IDs sind globale Referenzanker. |
| `data\copilot-models.json` | Runway, Tower, Cockpit, Search | `models[]`, `plans[]`, `surfaces[]`, `capabilities[]`, `notams[]`, `flightPlans[]` | Datei markiert sich selbst als verifikationspflichtig. |
| `data\governance-controls.json` | Tower, Cockpit, Search, Wiring-Integritaet | `sources`, `controls[]` | Control-IDs werden fuer Hashes und Graph-Kanten genutzt. |
| `data\sovereign-cloud.json` | Tower | `sovereignPillars[]`, `deploymentOptions[]`, `providerStrategies[]`, `residualRisks[]`, `dataFlowDiagram` | Enthaelt auch Mermaid-Quelltext. |
| `data\security-threats.json` | Security, Cockpit | `schema`, `threats[]` | `threats[].instrumentId` muss auf Instrumente zeigen. |
| `data\security-frameworks.json` | Security | `sources`, `frameworks`, `cwePattern` | Datei markiert sich explizit als manuell zu verifizieren. |
| `data\terminal-guide.json` | Terminal | `checkIn`, `boardingPass`, `firstFlight`, `departures` | Single-page-Katalog ohne querreferenzierte IDs. |
| `data\jet-bridge-guide.json` | Jet Bridge | `promptCraft`, `contextManagement`, `editMode`, `agentPatterns`, `nextSteps` | Page-lokal; geringe Kopplung. |
| `data\known-changelog-entries.json` | Flight Log, Search, Integritaetstests | `entryTypes`, `entries[]` | `entries[].instruments[]` referenzieren Instrument-IDs. |
| `data\preflight-checklist.json` | Pre-Flight | `intro`, `categories[]` | Fortschritt liegt nicht in JSON, sondern in `localStorage`. |
| `data\wiring-diagram.json` | Wiring, Integritaetstests | `connectionTypes[]`, `connections[]`, `zoneDescriptions` | `connections[].from/to` duerfen Instrumente oder Controls adressieren. |

Fuer Details zu Katalogkopplungen siehe [DATA-CATALOG.md](DATA-CATALOG.md).

## 4. URL-Hash-Kontrakte

### 4.1 `#instrument-<id>`

| Aspekt | Stand |
| --- | --- |
| Konsumenten | `index.html` (`app.js`), `ramp.html` |
| Produzenten | `app.js`, `ramp.html`, `wiring.html`, `flight-log.html`, `search.js` |
| Zweck | Oeffnet ein Instrument bzw. springt in dessen Detailansicht |

### 4.2 `#model-<id>`

| Aspekt | Stand |
| --- | --- |
| Konsument | `runway.html` |
| Produzenten | `runway.html`, `search.js`, `app.js` (EICAS-Links) |
| Zweck | Oeffnet die Model-Detailansicht auf der Runway-Seite |

### 4.3 `#scan=<id>`

| Aspekt | Stand |
| --- | --- |
| Konsument | `security.html` |
| Produzenten | `security.html`, `app.js` |
| Zweck | Aktiviert den Scanner-Eintrag fuer ein Instrument |

### 4.4 `#control=<id>` und `#sovereign=<id>`

| Hash | Konsument | Produzenten | Zweck |
| --- | --- | --- | --- |
| `#control=<id>` | `tower.html` | `tower.html`, `search.js`, `app.js` | Hebt eine Governance-Control hervor |
| `#sovereign=<id>` | `tower.html` | `tower.html` | Hebt eine Sovereign-Cloud-Option hervor |

## 5. Browser-Persistenz

| Key | Typ | Verwendet von | Bedeutung |
| --- | --- | --- | --- |
| `cockpit-theme` | String (`light` / `dark`) | Cockpit und mehrere Perspektivseiten | globales Theme |
| `cockpit-last-scan` | String | `security.html` | zuletzt aktiver Scan |
| `cockpit-security-posture` | JSON-Objekt | `security.html` | Security-Posture-Checkboxzustand |
| `copilot-preflight` | JSON-Objekt | `preflight.html` | Fortschritt der Checkliste |

## 6. Globale JavaScript-Oberflaeche

| Funktion | Quelle | Rolle |
| --- | --- | --- |
| `toggleTheme()` | `app.js` | Theme-Toggle auf der Cockpit-Seite |
| `copyCode(button)` | `app.js` | Kopiert Codeblock-Inhalte aus der Detailansicht |
| `window.openGlobalSearch()` | `search.js` | Oeffnet die globale Suchpalette |

Wichtig: Die meisten Perspektivseiten exportieren **keine** globale API. Ihr Verhalten lebt im jeweiligen Inline-Skript.

## 7. History- und Navigationsverhalten

| Datei | Verifizierter Umgang mit Browser-History |
| --- | --- |
| `app.js` | nutzt `history.pushState` und `popstate` fuer Cockpit-Detailpanel |
| `runway.html` | nutzt `history.replaceState` fuer `#model-<id>` |
| `security.html` | aktualisiert Hash ueber `history.replaceState` |
| `ramp.html` | aktualisiert `window.location.hash`, setzt beim Schliessen per `history.replaceState` zurueck |
| `tower.html` | liest Hashes fuer Highlighting; direkte Hash-Manipulation wird in Tests abgesichert |

## 8. Cache- und Auslieferungsvertrag

`vercel.json` definiert folgende Header:

| Pfadklasse | Cache-Control |
| --- | --- |
| `/media/*` | `public, max-age=31536000, immutable` |
| `/*.css` | `public, max-age=3600, must-revalidate` |
| `/*.js` | `public, max-age=3600, must-revalidate` |
| `/data/*` | `public, max-age=3600, must-revalidate` |

Folge: Die "API" ist read-only und dateibasiert, aber nicht ungecached.

## 9. Nicht explizit belegt

1. **Serverseitige Rewrite-/Routing-Regeln.** Im Repo ist nur statisches Root-Serving belegt; weitergehende Plattformlogik ist nicht sichtbar.
2. **Schreibende Schnittstellen ausserhalb des Repos.** Da keine Write-API im Code existiert, geht diese Doku von read-only Hosting aus; falls externe Systeme Inhalte erzeugen, sind sie hier nicht dokumentiert.
