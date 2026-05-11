# Architektur

## 1. Architekturmodell

Copilot Cockpit ist eine **statische, clientseitig gerenderte Multi-Page-Anwendung**:

```text
Browser
  -> HTML-Seite laden
  -> styles.css + search.js + page-spezifisches Script laden
  -> JSON unter /data/ per fetch() laden
  -> DOM rendern
  -> Hash / localStorage / Theme aktualisieren
```

Es existieren:

- **kein API-Server**
- **kein Build/Bundle-Schritt**
- **kein Router im SPA-Sinn**
- **kein serverseitiges Rendering**

`vercel.json` konfiguriert statisches Hosting und Cache-Header fuer `.css`, `.js`, `media/*` und `data/*`.

## 2. Zentrale Bausteine

### 2.1 Gemeinsame UI-Schicht

| Datei | Rolle |
|---|---|
| `styles.css` | globales HUD-Styling, responsive Navigation, Light/Dark Theme |
| `search.js` | globale Command-Palette mit Daten aus Instrumenten, Controls, Modellen und Changelog |
| Header/Footer in jeder HTML-Datei | wiederkehrende Navigation ueber alle Perspektiven |

Fast jede Seite implementiert ausserdem dasselbe Theme-Muster:

```js
const saved = localStorage.getItem('cockpit-theme');
if (saved === 'light') {
    document.body.classList.add('light-theme');
}
```

### 2.2 Cockpit-spezifische Runtime

`index.html` ist die einzige Seite, die `app.js` als externes Laufzeitmodul benutzt. Andere Seiten haben ihre Renderlogik inline im `<script>`-Block.

Wichtige Zustandsobjekte in `app.js`:

- `cockpitData`
- `allInstruments`
- `activeFilters`
- `searchQuery`
- `scannerIndex`
- `governanceIndex`

## 3. Seitenarchitektur

| Seite | Rendering-Modell | Besondere Komponenten |
|---|---|---|
| `index.html` | externes Script `app.js` | Grid, Zonen, Detail-Blade, lokale Suche, Prism, Mermaid |
| `terminal.html` | inline | Check-In, Boarding Pass, First Flight, Departure Board |
| `jet-bridge.html` | inline | Prompt-Cards, Kontextkarten, Edit-Workflows, Agent-Patterns |
| `ramp.html` | inline | Instrument-Grid fuer Ramp-Perspektive, Detail-Blade, Hash-Handling |
| `runway.html` | inline | Filterbar, Departure Board, Modell-Blade, Topologie, NOTAMs |
| `security.html` | inline | X-Ray-Scanner, Threat-Diagramme, Posture-Score, Compliance-Tabelle |
| `tower.html` | inline | Framework-Legende, Governance Controls, Sovereign Cloud, Flight Plans |
| `flight-log.html` | inline | Timeline, Filterbar fuer Entry Type und Zone |
| `preflight.html` | inline | persistente Checklist-UI mit Fortschrittsbalken |
| `wiring.html` | inline | Mermaid-Graph, Connection-Type-Filter, Zonen- und Statistikansichten |

## 4. Datenfluesse

### 4.1 Cockpit-Flow

`index.html` stellt nur Container bereit:

- `#cockpit-grid`
- `#cockpit-legend`
- `#detail-panel`

`app.js` erledigt dann:

1. Daten laden
2. Zonen in fester Reihenfolge rendern
3. Instrument-Karten rendern
4. Detail-Blade on demand aufbauen
5. Filter, Suche und Deep Links aktivieren

Wichtige Sonderpfade:

- **EICAS-Zone** rendert Modelle statt Instrument-Karten
- **FMS-Zone** rendert eine feste Chain-Reihenfolge
- Security- und Governance-Daten werden nur als Zusatzindex geladen

### 4.2 Security-Flow

`security.html` laedt drei Quellen parallel:

```js
const [inst, threats, frameworks] = await Promise.all([
    fetch('data/copilot-instruments.json').then(r => r.json()),
    fetch('data/security-threats.json').then(r => r.json()),
    fetch('data/security-frameworks.json').then(r => r.json())
]);
```

Danach baut die Seite:

- die Luggage Lane (`.luggage-item`)
- das Scan-Detail
- Mermaid-Threat-Modelle
- Before/After-Demos
- Framework-Chips

Die aktive Auswahl kommt aus:

1. `#scan=<instrumentId>`
2. sonst `localStorage['cockpit-last-scan']`
3. sonst erster Threat-Eintrag

### 4.3 Runway-Flow

`runway.html` ist die Modellperspektive. Das Datenmodell aus `copilot-models.json` steuert:

- Provider-Chips
- Plan-Select
- Statusfilter
- Departure Board
- Model Blade
- Topology Matrix

Die Blade-Navigation ist hash-basiert:

```text
runway.html#model-gpt-4-1
```

### 4.4 Tower-Flow

`tower.html` verknuepft drei fachlich getrennte Sichten:

- Governance Controls (`governance-controls.json`)
- Modellrouting / Flight Plans (`copilot-models.json`)
- Sovereignty / Residency (`sovereign-cloud.json`)

Deep Links highlighten Zielknoten im bestehenden Layout, statt eine Blade zu oeffnen:

```text
tower.html#control=feature-policies
tower.html#sovereign=byok-enterprise
```

### 4.5 Wiring-Flow

`wiring.html` kombiniert:

- `wiring-diagram.json` fuer Kanten- und Typdefinitionen
- `copilot-instruments.json` fuer Knotennamen, Symbole und Zonen

Aus den JSON-Daten wird Mermaid-Source generiert. Click-Aktionen springen direkt ins Cockpit:

```js
src += `    click ${id} "index.html#instrument-${id}" "..."\n`;
```

## 5. Komponenten und Beziehungen

### 5.1 Header-Navigation

Alle Seiten verwenden dieselbe Perspektivenleiste. Die Navigation ist also **kopiert, nicht zentral komponiert**. Das reduziert Abhaengigkeiten, fuehrt aber zu Redundanz.

### 5.2 Suche

Es gibt zwei Suchsysteme:

| Suche | Datei | Scope |
|---|---|---|
| lokale Cockpit-Suche | `app.js` | Filtert Instrument-Karten in `index.html` |
| globale Suche | `search.js` | palette-basierte Suche ueber mehrere Datenkataloge |

Wichtig: Auf `index.html` teilen sich beide Systeme Shortcut-Logik um `Ctrl+K`.

### 5.3 Theme

Das Theme wird in `localStorage['cockpit-theme']` gespeichert und auf allen Seiten wiederverwendet. Mermaid-Seiten rendern Diagramme nach Theme-Wechsel neu.

### 5.4 Detail-Komponenten

| Blade / Detailansicht | Seite | Trigger |
|---|---|---|
| Cockpit Detail Blade | `index.html` | Klick auf `.instrument`, `#instrument-...` |
| Ramp Blade | `ramp.html` | Klick auf `.ramp-card`, `#instrument-...` |
| Runway Model Blade | `runway.html` | Klick auf `.departure-row`, `#model-...` |
| Security Scanner Detail | `security.html` | Klick auf `.luggage-item`, `#scan=...` |

## 6. Persistenz und URL-State

| Mechanismus | Verwendet in | Zweck |
|---|---|---|
| `localStorage['cockpit-theme']` | fast alle Seiten | Theme |
| `localStorage['copilot-preflight']` | `preflight.html` | Checklist-Status |
| `localStorage['cockpit-last-scan']` | `security.html` | zuletzt gescannter Threat |
| `localStorage['cockpit-security-posture']` | `security.html` | Posture-Checkboxen |
| URL-Hash | mehrere Seiten | Deep Link auf Instrument, Modell, Control oder Sovereign Option |

## 7. Externe Laufzeitabhaengigkeiten

| Technologie | Verwendung |
|---|---|
| Mermaid | `index.html`, `runway.html`, `security.html`, `tower.html`, `wiring.html` |
| Prism.js | nur `index.html`, fuer Code-Tabs in der Detail-Blade |
| Google Fonts (`JetBrains Mono`) | alle Seiten |
| Vercel Insights / Speed Insights | Footer-Scripts auf mehreren Seiten |

## 8. Architekturentscheidungen und Konsequenzen

### Vorteile

- sehr einfacher Deploy
- gute Lesbarkeit der HTML-Einstiegspunkte
- Daten und Darstellung klar getrennt
- JSON-Dateien lassen sich unabhaengig pflegen

### Kosten

- Header/Footer/Theme-Code ist ueber viele Seiten dupliziert
- keine zentrale Router- oder State-Schicht
- Deep-Link-Logik ist seitenlokal implementiert
- Integrationsfehler zwischen JSON-Dateien muessen durch Tests abgefangen werden

## 9. Wichtigste technische Beobachtungen

1. **`copilot-instruments.json` ist der Hub** fuer mehrere andere Datenmodelle.
2. **`app.js` behandelt Zusatzdaten tolerant**; fehlende Threat-, Governance- oder Modelldaten blockieren das Cockpit nicht.
3. **Die Seite ist absichtlich page-local aufgebaut**: fast jede Perspektive besitzt ihre eigene Renderpipeline.
4. **Die Dokumentation sollte das Repo als MPA beschreiben, nicht als SPA.**
