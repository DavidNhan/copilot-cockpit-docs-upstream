# Copilot Cockpit — Technische Dokumentation

Diese Dokumentation beschreibt das Quell-Repo `C:\temp\copilot-cockpit` als **statische, datengetriebene Multi-Page-Webanwendung**. Es gibt keinen Backend-Service und keinen Build-Schritt; jede HTML-Seite lädt ihre JSON-Daten direkt per `fetch()` und rendert den Inhalt im Browser.

## System auf einen Blick

| Bereich | Dateien | Aufgabe |
|---|---|---|
| Seiten-Shells | `index.html`, `terminal.html`, `security.html`, `jet-bridge.html`, `ramp.html`, `runway.html`, `tower.html`, `flight-log.html`, `preflight.html`, `wiring.html` | Perspektiven, Navigation, DOM-Container, page-lokale Scripts |
| Gemeinsame Assets | `styles.css`, `search.js`, `favicon.svg`, `og-image.png` | Styling, globale Suche, Branding |
| Hauptlogik Cockpit | `app.js` | Rendern des Cockpit-Grids, Filter, Detail-Blade, Deep Links |
| Datenkatalog | `data\*.json` | Inhalte fuer Instrumente, Modelle, Controls, Threats, Guides |
| Tests | `tests\*.spec.js`, `playwright.config.js` | Browser-E2E-Tests plus Integritaetschecks gegen JSON |
| Deployment | `vercel.json` | Statisches Hosting und Cache-Header |

## Seiten und Zweck

| Seite | Datei | Datenquelle(n) | Zweck |
|---|---|---|---|
| Cockpit | `index.html` + `app.js` | `data/copilot-instruments.json` plus optionale Enrichment-Dateien | Hauptansicht mit Instrumenten-Grid, Filtern und Detail-Blade |
| Terminal | `terminal.html` | `data/terminal-guide.json` | Einstieg: Plaene, IDE-Setup, erste Uebungen |
| Jet Bridge | `jet-bridge.html` | `data/jet-bridge-guide.json` | Prompting, Kontext, Edit Mode, Agent-Patterns |
| Ramp | `ramp.html` | `data/copilot-instruments.json` | Agenten, MCP, Ground-Handling-Perspektive |
| Runway | `runway.html` | `data/copilot-models.json` | Modellkatalog, Handover-Topologie, NOTAMs |
| Security | `security.html` | `data/copilot-instruments.json`, `data/security-threats.json`, `data/security-frameworks.json` | X-Ray-Scanner, Threat Models, Hardening |
| Tower | `tower.html` | `data/governance-controls.json`, `data/copilot-models.json`, `data/sovereign-cloud.json` | Governance, Compliance, Souveraenitaet |
| Flight Log | `flight-log.html` | `data/known-changelog-entries.json` | Changelog-Timeline |
| Pre-Flight | `preflight.html` | `data/preflight-checklist.json` | Interaktive Rollout-Checkliste |
| Wiring | `wiring.html` | `data/wiring-diagram.json`, `data/copilot-instruments.json` | Mermaid-Graph fuer Feature-Beziehungen |

## Wichtige technische Merkmale

1. **Statische MPA statt SPA**: Jede HTML-Datei ist ein eigener Einstiegspunkt.
2. **JSON als API-Ersatz**: Die komplette Fachlogik wird aus `data\*.json` gespeist.
3. **Gemeinsame UI-Bausteine**: `styles.css` und `search.js` werden auf fast allen Seiten wiederverwendet.
4. **Hash-basierte Navigation**: Details werden ueber `#instrument-...`, `#model-...`, `#scan=...`, `#control=...` und `#sovereign=...` adressiert.

## Beispiel: Cockpit-Bootstrap

Aus `app.js`:

```js
const [resp, threatsResp, governanceResp, modelsResp] = await Promise.all([
    fetch('data/copilot-instruments.json'),
    fetch('data/security-threats.json').catch(() => null),
    fetch('data/governance-controls.json').catch(() => null),
    fetch('data/copilot-models.json').catch(() => null)
]);
```

Das Muster ist typisch fuer das Repo: **eine Pflichtquelle plus optionale Zusatzdaten**.

## Doku-Navigation

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — Datenfluesse, Seitenstruktur, Komponenten
- [`API-REFERENCE.md`](./API-REFERENCE.md) — HTML-Endpunkte, Deep Links, `fetch()`-Calls
- [`DATA-CATALOG.md`](./DATA-CATALOG.md) — alle JSON-Dateien unter `data\`
- [`TESTING-GUIDE.md`](./TESTING-GUIDE.md) — Playwright-Setup, Spec-Dateien, Abdeckung
