# API-Reference

## 1. Einordnung

Das Repo besitzt **keine klassische HTTP-API mit Backend-Endpoints**. Die "API-Oberflaeche" besteht aus:

1. **statischen HTML-Endpunkten**
2. **statischen JSON-Endpunkten unter `/data/`**
3. **Hash-basierten Deep-Link-Kontrakten**
4. **externen CDN-Assets**

## 2. HTML-Endpunkte

| Endpoint | Datei | Zweck | Wichtige DOM-Roots |
|---|---|---|---|
| `/index.html` bzw. `/` | `index.html` | Haupt-Cockpit | `#cockpit-grid`, `#detail-panel`, `#search-input` |
| `/terminal.html` | `terminal.html` | Einstieg / Onboarding | `#terminal-main`, `#terminal-plans`, `#terminal-ides` |
| `/jet-bridge.html` | `jet-bridge.html` | Interaktionsmuster | `#jet-bridge-main`, `#jb-techniques`, `#jb-patterns` |
| `/ramp.html` | `ramp.html` | Agents / MCP | `#ramp-main`, `#ramp-grid`, `#ramp-blade` |
| `/runway.html` | `runway.html` | Modellkatalog | `#runway-main`, `#departure-board`, `#model-blade` |
| `/security.html` | `security.html` | Threat-Modelling | `#security-main`, `#luggage-lane`, `#scanner-content` |
| `/tower.html` | `tower.html` | Governance / Sovereignty | `#tower-main`, `#control-list`, `#sovereign-cloud-section` |
| `/flight-log.html` | `flight-log.html` | Changelog | `#log-stats`, `#log-timeline` |
| `/preflight.html` | `preflight.html` | Checklist | `#preflight-main`, `#preflight-categories` |
| `/wiring.html` | `wiring.html` | Verbindungsgraph | `#wiring-main`, `#wiring-diagram`, `#wiring-stats` |

## 3. JSON-Endpunkte

| Endpoint | Datei | Konsumenten | Pflicht / Optional |
|---|---|---|---|
| `/data/copilot-instruments.json` | `data\copilot-instruments.json` | `app.js`, `ramp.html`, `wiring.html`, `search.js`, `security.html` | Pflicht fuer Cockpit |
| `/data/copilot-models.json` | `data\copilot-models.json` | `runway.html`, `tower.html`, `app.js`, `search.js` | optional im Cockpit, Pflicht auf Runway/Tower |
| `/data/governance-controls.json` | `data\governance-controls.json` | `tower.html`, `app.js`, `search.js` | optional im Cockpit |
| `/data/security-threats.json` | `data\security-threats.json` | `security.html`, `app.js` | optional im Cockpit, Pflicht in Security |
| `/data/security-frameworks.json` | `data\security-frameworks.json` | `security.html` | Pflicht in Security |
| `/data/terminal-guide.json` | `data\terminal-guide.json` | `terminal.html` | Pflicht |
| `/data/jet-bridge-guide.json` | `data\jet-bridge-guide.json` | `jet-bridge.html` | Pflicht |
| `/data/preflight-checklist.json` | `data\preflight-checklist.json` | `preflight.html` | Pflicht |
| `/data/known-changelog-entries.json` | `data\known-changelog-entries.json` | `flight-log.html`, `search.js` | Pflicht |
| `/data/sovereign-cloud.json` | `data\sovereign-cloud.json` | `tower.html` | Pflicht |
| `/data/wiring-diagram.json` | `data\wiring-diagram.json` | `wiring.html` | Pflicht |

## 4. `fetch()`-Aufrufe nach Datei

### 4.1 `app.js`

```js
fetch('data/copilot-instruments.json'),
fetch('data/security-threats.json').catch(() => null),
fetch('data/governance-controls.json').catch(() => null),
fetch('data/copilot-models.json').catch(() => null)
```

**Bedeutung**

- `copilot-instruments.json` ist zwingend
- Threats / Controls / Models sind Soft-Fail-Enrichment

### 4.2 `search.js`

```js
fetch('data/copilot-instruments.json'),
fetch('data/governance-controls.json').catch(() => null),
fetch('data/copilot-models.json').catch(() => null),
fetch('data/known-changelog-entries.json').catch(() => null)
```

Die globale Suche erzeugt daraus vier Result-Typen:

- `instrument`
- `control`
- `model`
- `changelog`

### 4.3 Page-lokale Fetches

| Datei | Fetch | Zweck |
|---|---|---|
| `terminal.html` | `fetch('data/terminal-guide.json')` | Onboarding-Guide |
| `jet-bridge.html` | `fetch('data/jet-bridge-guide.json')` | Prompt-/Context-/Edit-Guide |
| `preflight.html` | `fetch('data/preflight-checklist.json')` | Checklist-Inhalt |
| `ramp.html` | `fetch('data/copilot-instruments.json')` | Filter auf `perspectives.includes('ramp')` |
| `runway.html` | `fetch('data/copilot-models.json')` | Model Board und Blade |
| `security.html` | drei parallele Fetches | Instrumente, Threats, Framework-Registry |
| `tower.html` | drei parallele Fetches | Controls, Modelle, Sovereignty |
| `flight-log.html` | `fetch('data/known-changelog-entries.json')` | Timeline |
| `wiring.html` | zwei parallele Fetches | Connections und Instrument-Metadaten |

## 5. Deep-Link-Vertrag

| Format | Beispiel | Konsument | Wirkung |
|---|---|---|---|
| `#instrument-<id>` | `index.html#instrument-agent-mode` | `app.js` | oeffnet Cockpit-Detail-Blade |
| `#instrument-<id>` | `ramp.html#instrument-mcp` | `ramp.html` | oeffnet Ramp-Blade |
| `#model-<id>` | `runway.html#model-gpt-4-1` | `runway.html` | oeffnet Model-Blade |
| `#scan=<id>` | `security.html#scan=agent-mode` | `security.html` | aktiviert Threat im X-Ray-Scanner |
| `#control=<id>` | `tower.html#control=usage-metrics` | `tower.html` | highlightet Governance Control |
| `#sovereign=<id>` | `tower.html#sovereign=byok-enterprise` | `tower.html` | highlightet Sovereign Option |

## 6. Cross-Page-Linking

Wichtige interne Linkbeziehungen:

- Cockpit-Detail -> Security: `security.html#scan=<instrumentId>`
- Cockpit-Detail -> Tower: `tower.html#control=<instrumentId>`
- Cockpit-Detail -> Runway: `runway.html`
- Wiring-Knoten -> Cockpit: `index.html#instrument-<id>`
- Flight-Log-Eintrag -> Cockpit: `index.html#instrument-<id>`
- Security-Scanner -> Cockpit: `index.html#instrument-<id>`

## 7. Externe Laufzeit-Endpoints

| Typ | URL-Muster | Verwendet in |
|---|---|---|
| Mermaid | `https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js` | `index.html`, `runway.html`, `security.html`, `tower.html` |
| Mermaid ESM | `https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs` | `wiring.html` |
| Prism | `https://cdn.jsdelivr.net/npm/prismjs@1/...` | `index.html` |
| Google Fonts | `https://fonts.googleapis.com/...JetBrains+Mono...` | alle Seiten |
| Vercel Insights | `/_vercel/insights/script.js` | mehrere Seiten |
| Vercel Speed Insights | `/_vercel/speed-insights/script.js` | mehrere Seiten |

## 8. Cache- und Deploy-Verhalten

`vercel.json` definiert:

- `outputDirectory: "."`
- `.css` und `.js`: `max-age=3600, must-revalidate`
- `/data/*`: `max-age=3600, must-revalidate`
- `/media/*`: `max-age=31536000, immutable`

Das heisst fuer API-Konsumenten im weiteren Sinn:

1. Daten werden direkt statisch ausgeliefert.
2. JSON-Aenderungen sind innerhalb kurzer Cache-Fenster sichtbar.
3. Medien koennen aggressiv gecacht werden.

## 9. Fehlerverhalten

Seiten ohne Soft-Fail-Muster ersetzen ihren Hauptcontainer bei Fehlern durch:

```html
<p style="color:#ff4444;padding:40px;">DATA LINK LOST — ...</p>
```

Das ist das einheitliche Runtime-Fehlermuster fuer fehlende oder defekte JSON-Dateien.
