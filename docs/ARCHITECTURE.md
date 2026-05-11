# Architektur

## 1. Laufzeitmodell

Das Repository unter `C:\temp\copilot-cockpit` ist eine rein statische Webanwendung.

- Keine Server-API
- Kein Bundler
- Kein Transpiler
- Kein Build-Artefaktverzeichnis
- Client-seitiges Rendering aus JSON-Dateien

Die produktive Laufzeit ist:

1. Browser laedt eine HTML-Seite.
2. Die Seite oder ein gemeinsames Skript laedt JSON unter `data\`.
3. JavaScript rendert DOM-Strukturen dynamisch.
4. Deep Links und `localStorage` halten UI-Zustand stabil.

## 2. Struktur der Einstiegspunkte

| Ebene | Dateipfade | Technische Rolle |
| --- | --- | --- |
| Cockpit-Hauptseite | `C:\temp\copilot-cockpit\index.html` + `app.js` | Hauptgrid, Detailpanel, Cross-Perspective-Callouts |
| Perspektivseiten | `terminal.html`, `jet-bridge.html`, `ramp.html`, `runway.html`, `tower.html`, `security.html`, `flight-log.html`, `preflight.html`, `wiring.html` | Je Seite ein isolierter Daten- und Renderpfad |
| Gemeinsame Suche | `C:\temp\copilot-cockpit\search.js` | Globale Palette ueber Instrumente, Controls, Modelle, Changelog |
| Styling | `C:\temp\copilot-cockpit\styles.css` | Gemeinsame Layout-, Theme- und Komponentenregeln |

## 3. Komponentenmodell

### 3.1 Cockpit

`C:\temp\copilot-cockpit\app.js` laedt beim `DOMContentLoaded` parallel:

- `data\copilot-instruments.json`
- optional `data\security-threats.json`
- optional `data\governance-controls.json`
- optional `data\copilot-models.json`

Interne Hauptaufgaben:

- Zonen-Rendering in festem Ordnungsvektor (`ZONE_ORDER`)
- Sonderlogik fuer `eicas` (Modelle statt Instrumentkarten)
- Sonderlogik fuer `fms` (Kettenlayout)
- Detailpanel mit Tabs `overview`, `diagrams`, `code`, `media`, `resources`
- Cross-Page-Callouts nach Security, Tower und Runway
- Filter, Suche, Deep Links und Theme-Persistenz

### 3.2 Perspektivseiten

Jede Perspektivseite besitzt ihr eigenes Inline-Bootstrapping:

- `terminal.html` -> `terminal-guide.json`
- `jet-bridge.html` -> `jet-bridge-guide.json`
- `ramp.html` -> Filter auf `copilot-instruments.json` via `perspectives.includes('ramp')`
- `runway.html` -> `copilot-models.json`
- `tower.html` -> `governance-controls.json` + `copilot-models.json` + `sovereign-cloud.json`
- `security.html` -> `copilot-instruments.json` + `security-threats.json` + `security-frameworks.json`
- `flight-log.html` -> `known-changelog-entries.json`
- `preflight.html` -> `preflight-checklist.json`
- `wiring.html` -> `wiring-diagram.json` + `copilot-instruments.json`

Es gibt keine zentrale Router- oder SPA-Schicht. Navigation erfolgt ueber klassische HTML-Links.

### 3.3 Globale Suche

`C:\temp\copilot-cockpit\search.js` baut einen Suchindex aus vier Quellen:

- `data\copilot-instruments.json`
- `data\governance-controls.json`
- `data\copilot-models.json`
- `data\known-changelog-entries.json`

Die Suche erzeugt Ziel-URLs direkt aus den Hash-Kontrakten:

- `index.html#instrument-<id>`
- `tower.html#control=<id>`
- `runway.html#model-<id>`
- `flight-log.html`

## 4. Datenfluss

## 4.1 Primarfluss

```text
HTML -> fetch(data/*.json) -> in-memory state -> DOM rendering
```

## 4.2 Querverlinkung

```text
Cockpit detail panel
  -> security.html#scan=<id>
  -> tower.html#control=<id>
  -> runway.html

Wiring / Flight Log / Ramp / Search
  -> index.html#instrument-<id>
```

## 4.3 Persistenz

Nur Browser-seitig:

- `cockpit-theme`
- `cockpit-last-scan`
- `cockpit-security-posture`
- `copilot-preflight`

Es gibt keine serverseitige Session, kein Cookie-basiertes State-Management und keine externe Persistenz fuer Nutzerinteraktionen.

## 5. Deployment- und Caching-Regeln

`C:\temp\copilot-cockpit\vercel.json` konfiguriert statisches Hosting mit Headern:

| Pfadklasse | Cache-Control |
| --- | --- |
| `/media/*` | `public, max-age=31536000, immutable` |
| `/*.css` | `public, max-age=3600, must-revalidate` |
| `/*.js` | `public, max-age=3600, must-revalidate` |
| `/data/*` | `public, max-age=3600, must-revalidate` |

Folgen:

- Medien-GIFs sind langfristig gecacht.
- JSON- und JS-Kataloge sind kurzlebiger, aber weiterhin cachebar.
- Es gibt kein API-Versioning im URL-Schema; Aktualitaet haengt an Dateiinhalten und Revalidierung.

## 6. Offline-Datenanreicherung

`C:\temp\copilot-cockpit\tools\enrich\` ist kein Teil der Produktionslaufzeit. Die Pipeline dient dazu, Modelldaten aus Upstream-Quellen zu sammeln.

Technische Eigenschaften:

- `harvest.py` und Adapter lesen Upstream-Quellen.
- `normalize.py` ist als Merge-Stufe vorgesehen.
- `tools\cache\` ist der lokale Cache.
- Laut `tools\enrich\README.md` darf die Pipeline nicht direkt nach `data\` schreiben.

Das ist relevant fuer `data\copilot-models.json`: Der Katalog ist ein statisches Artefakt, nicht das Live-Ergebnis einer Laufzeit-Synchronisation.

## 7. Hash-Kontrakte im Detail

| Seite | Eingehender Hash | Effekt |
| --- | --- | --- |
| `index.html` | `#instrument-<id>` | Oeffnet das Cockpit-Detailpanel |
| `ramp.html` | `#instrument-<id>` | Oeffnet das Ramp-Blade |
| `runway.html` | `#model-<id>` | Oeffnet das Model-Blade |
| `security.html` | `#scan=<id>` | Aktiviert einen Threat-Scan |
| `tower.html` | `#control=<id>` | Markiert eine Governance-Control |
| `tower.html` | `#sovereign=<id>` | Markiert eine Sovereign-Cloud-Option |

Die Hashes sind nicht kosmetisch, sondern Teil der Integrationsvertraege zwischen Seiten, globaler Suche, Wiring-Graf und Tests.

## 8. Testrelevante Architektur-Invarianten

Die Architektur wird nicht nur ueber Rendering, sondern ueber konkrete Vertraege abgesichert:

- `tests\integrity.spec.js` prueft Cross-References zwischen JSON-Dateien.
- `tests\cockpit.spec.js` prueft `#instrument-<id>`, Blade-Verhalten und Theme.
- `tests\runway.spec.js` prueft `#model-<id>` und Modell-Matrix.
- `tests\security.spec.js` prueft `#scan=<id>` und die Callout-Integration vom Cockpit.
- `tests\tower.spec.js` prueft `#control=<id>` und `#sovereign=<id>`.
- `tests\wiring.spec.js` prueft die Diagramm- und Legendenstruktur.

## 9. Annahmen

1. **Mermaid bleibt produktiv aktiv, obwohl in diesem Lauf keine Mermaid-Diagramme dokumentiert werden sollten.** Die Laufzeitdokumentation beschreibt daher Mermaid als bestehende technische Abhaengigkeit, nicht als Dokumentationsmittel.
2. **`python3` wird fuer lokale Test-Serving-Pfade vorausgesetzt.** Das ist aus `playwright.config.js` abgeleitet; ein erfolgreicher frischer Lauf konnte hier wegen fehlendem `pwsh.exe` nicht bestaetigt werden.
