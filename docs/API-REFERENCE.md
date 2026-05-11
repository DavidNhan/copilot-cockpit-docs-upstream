# API Reference

## 1. Was in diesem Repo "API" bedeutet

Es gibt **keine Backend-API**. Die technische Oberflaeche des Systems besteht aus:

1. HTML-Routen
2. JSON-Endpunkten unter `data\`
3. DOM-Entry-Points
4. URL-Hash-Kontrakten
5. externen CDN- oder Vercel-Skripten

## 2. Seiten- und Routenreferenz

| Route | Quelle | DOM-Roots / Einstiegspunkte | Datenquellen | Hauptinteraktionen |
|---|---|---|---|---|
| `/` oder `/index.html` | `index.html` + `app.js` | `#cockpit-grid`, `#cockpit-legend`, `#detail-panel`, `#search-input` | `copilot-instruments.json` plus optionale Threat-, Governance- und Modelldaten | Zonen-Grid, Filter, lokale Suche, Blade, `#instrument-<id>` |
| `/terminal.html` | `terminal.html` | `#terminal-main`, `#terminal-plans`, `#terminal-ides`, `#terminal-exercises`, `#terminal-departures` | `terminal-guide.json` | statische Content-Karten, Navigationslinks |
| `/jet-bridge.html` | `jet-bridge.html` | `#jet-bridge-main`, `#jb-techniques`, `#jb-participants`, `#jb-variables`, `#jb-workflows`, `#jb-patterns` | `jet-bridge-guide.json` | Content-Sections und Pattern-Karten |
| `/ramp.html` | `ramp.html` | `#ramp-main`, `#ramp-grid`, `#ramp-blade`, `#ramp-blade-body` | `copilot-instruments.json` | Ramp-Filterung, Blade, `#instrument-<id>` |
| `/runway.html` | `runway.html` | `#runway-main`, `#verification-banner`, `#departure-board`, `#model-blade`, `#topology-diagram`, `#notam-list` | `copilot-models.json` | Provider-/Status-/Plan-Filter, Blade, NOTAM-Links, `#model-<id>` |
| `/security.html` | `security.html` | `#security-main`, `#luggage-lane`, `#scanner-content`, `#posture-checklist` | `copilot-instruments.json`, `security-threats.json`, `security-frameworks.json` | Scanner, Arrow-Key-Navigation, Posture-Persistenz, `#scan=<id>` |
| `/tower.html` | `tower.html` | `#tower-main`, `#framework-list`, `#control-list`, `#sovereign-options`, `#flight-plans` | `governance-controls.json`, `copilot-models.json`, `sovereign-cloud.json` | Highlighting, Sovereign-Optionen, Flight Plans, `#control=` / `#sovereign=` |
| `/flight-log.html` | `flight-log.html` | `#log-stats`, `#log-timeline` | `known-changelog-entries.json` | Timeline-Filter und Instrument-Backlinks |
| `/preflight.html` | `preflight.html` | `#preflight-main`, `#progress-fill`, `#preflight-categories`, `#reset-btn` | `preflight-checklist.json` | Checkbox-Persistenz und Fortschritt |
| `/wiring.html` | `wiring.html` | `#wiring-main`, `#wiring-filters`, `#wiring-diagram`, `#wiring-legend`, `#wiring-zones`, `#wiring-stats` | `wiring-diagram.json`, `copilot-instruments.json` | Typfilter, Mermaid-Graph, Klick in Cockpit-Deep-Links |

## 3. Wichtige JS-Entry-Points

| Datei | Rolle | Technischer Vertrag |
|---|---|---|
| `app.js` | zentrale Cockpit-Runtime | erwartet `#cockpit-grid`, `#cockpit-legend`, `#detail-panel`, `#search-input` in `index.html` |
| `search.js` | globale Suche | kann auf mehreren Seiten geladen werden und baut einen eigenen Overlay-Index |
| page-lokale Inline-Skripte | seitenweises Rendering | erwarten jeweils spezifische Container-IDs der Seite |

### `app.js` im Detail

| Bereich | Relevante Aufgaben |
|---|---|
| Bootstrap | `DOMContentLoaded`, `Promise.all(...)`, tolerante Zusatzdaten |
| Rendering | `renderCockpit`, `renderZone`, `renderEngineZone`, `renderFmsZone`, `renderLegend` |
| Blade | `openDetailPanel`, `closeDetailPanel`, `renderDetailTabs` |
| Suche/Filter | `initFilters`, `applyFilters`, `initSearch` |
| Navigation | `handleDeepLink`, `popstate`, `Escape`, Klick auf gedimmten Hauptbereich |
| Theme | `toggleTheme`, `loadSavedTheme` |

### `search.js` im Detail

| Typ | Quelle | Ziel-URL |
|---|---|---|
| `instrument` | `copilot-instruments.json` | `index.html#instrument-<id>` |
| `control` | `governance-controls.json` | `tower.html#control=<id>` |
| `model` | `copilot-models.json` | `runway.html#model-<id>` |
| `changelog` | `known-changelog-entries.json` | `flight-log.html` |

## 4. `fetch()`-Matrix

| Konsument | Fetches | Pflicht / Soft-Fail | Zweck |
|---|---|---|---|
| `app.js` | `copilot-instruments.json`, `security-threats.json`, `governance-controls.json`, `copilot-models.json` | Instrumente Pflicht, Rest Soft-Fail | Cockpit-Grid plus Enrichment fuer Security, Tower und EICAS |
| `search.js` | `copilot-instruments.json`, `governance-controls.json`, `copilot-models.json`, `known-changelog-entries.json` | Instrumente faktisch Kern, Rest Soft-Fail | globaler Suchindex |
| `terminal.html` | `terminal-guide.json` | Pflicht | Onboarding-Inhalte |
| `jet-bridge.html` | `jet-bridge-guide.json` | Pflicht | Prompt- und Agent-Guide |
| `ramp.html` | `copilot-instruments.json` | Pflicht | Ramp-spezifischer Ausschnitt aus dem Instrumentkatalog |
| `runway.html` | `copilot-models.json` | Pflicht | Model Board, Blade, Topology, NOTAMs |
| `security.html` | `copilot-instruments.json`, `security-threats.json`, `security-frameworks.json` | alle Pflicht | Threat-Scanner und Framework-Mappings |
| `tower.html` | `governance-controls.json`, `copilot-models.json`, `sovereign-cloud.json` | alle Pflicht | Controls, Sovereign Cloud, Flight Plans |
| `flight-log.html` | `known-changelog-entries.json` | Pflicht | Timeline und Stats |
| `preflight.html` | `preflight-checklist.json` | Pflicht | Checklist-Content |
| `wiring.html` | `wiring-diagram.json`, `copilot-instruments.json` | beide Pflicht | Graph und Zonen-Metadaten |

## 5. Hash- und URL-Kontrakte

| Format | Konsument | Wirkung |
|---|---|---|
| `#instrument-<id>` | Cockpit | oeffnet die Detail-Blade via `history.pushState` |
| `#instrument-<id>` | Ramp | oeffnet die Ramp-Blade |
| `#model-<id>` | Runway | oeffnet die Model Blade |
| `#scan=<id>` | Security | waehlt einen Threat im Scanner |
| `#control=<id>` | Tower | hebt ein Governance-Control hervor und scrollt dorthin |
| `#sovereign=<id>` | Tower | hebt eine Sovereign-Option hervor und scrollt dorthin |

## 6. Relevante Interaktionsvertraege

### Cockpit

| Interaktion | Effekt |
|---|---|
| Klick auf `.instrument` | Blade oeffnen |
| `Escape` | Blade schliessen |
| Klick auf nicht-instrumentierten Hauptbereich | Blade schliessen |
| Filterbuttons | Karten dimmen oder aktivieren |
| Sucheingabe oder `/` | lokale Suche fokussieren |

### Global Search

| Interaktion | Effekt |
|---|---|
| `Ctrl+K` / `Cmd+K` | Overlay oeffnen |
| Pfeiltasten | Treffer wechseln |
| `Enter` | Zielroute oeffnen |
| `Escape` | Overlay schliessen |

### Runway

| Interaktion | Effekt |
|---|---|
| Provider-/Status-Chips | `departure-row`-Elemente dimmen |
| Auswahl eines Plans | sichtbares/gedimmtes Set neu berechnen |
| Klick auf Modellzeile, NOTAM-Link oder Alternative-Link | Blade oeffnen bzw. Modell wechseln |

### Security

| Interaktion | Effekt |
|---|---|
| Klick auf Luggage-Item | Scanner neu rendern und Hash setzen |
| Pfeil links/rechts | naechsten Scan waehlen |
| Posture-Checkbox | `cockpit-security-posture` aktualisieren und Score neu berechnen |

### Tower

| Interaktion | Effekt |
|---|---|
| `#control=` | `.control-row.highlight` setzen |
| `#sovereign=` | `.sovereign-option.highlight` setzen |
| Framework-Chips | externer Link zu Standards |

### Wiring

| Interaktion | Effekt |
|---|---|
| Filterbutton | Mermaid-Source neu generieren |
| Klick auf Mermaid-Knoten | Sprung zu `index.html#instrument-<id>` |

## 7. Fehlerverhalten

Die meisten Seiten folgen bei Pflichtdaten demselben Muster:

```html
<p style="color:#ff4444;padding:40px;">DATA LINK LOST - ...</p>
```

Das ist kein globales Error-Framework, sondern eine seitenlokale Fallback-Ausgabe.

## 8. Externe Laufzeitabhaengigkeiten

| Typ | Verwendet fuer |
|---|---|
| Mermaid CDN | Diagramme in Cockpit, Runway, Security, Tower, Wiring |
| Prism CDN | Syntaxhervorhebung in der Cockpit-Blade |
| Google Fonts | `JetBrains Mono` |
| `/_vercel/insights/script.js` und `/_vercel/speed-insights/script.js` | Metriken auf mehreren Seiten |

## 9. Was diese Referenz nicht behauptet

1. Keine Aussage ueber serverseitige APIs, weil es keine gibt.
2. Keine Garantie, dass alle fachlichen Modelldaten aktuell sind; der Katalog markiert sich selbst teilweise als verifikationspflichtig.
3. Keine automatische Sync-Zusage zwischen Doku-Repo und Quell-Repo; Aenderungen muessen bewusst nachgezogen werden.

Weiterfuehrend: [`ARCHITECTURE.md`](ARCHITECTURE.md), [`DATA-CATALOG.md`](DATA-CATALOG.md)
