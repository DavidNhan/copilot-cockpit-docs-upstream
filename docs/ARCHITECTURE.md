# Architektur

## 1. Architektur in einem Satz

Copilot Cockpit ist eine **statische, clientseitig gerenderte Multi-Page-App**, in der jede Perspektive als eigene HTML-Seite mit eigener Renderlogik lebt und JSON-Dateien die Rolle einer kleinen in-repo Content-API uebernehmen.

## 2. Systemmodell

```text
Browser
  -> HTML-Seite oeffnen
  -> styles.css + optionale CDN-Skripte + search.js laden
  -> page-lokales Script oder app.js initialisieren
  -> JSON-Dateien aus /data/ per fetch() laden
  -> DOM rendern
  -> URL-Hash und localStorage fuer Zustand nutzen
```

### Was es bewusst nicht gibt

| Nicht vorhanden | Konsequenz |
|---|---|
| Backend-Service | Keine serverseitige API, kein Auth-, Session- oder Persistenz-Layer |
| Build/Bundle-Schritt | Seiten koennen direkt statisch ausgeliefert werden |
| SPA-Router | Navigation passiert ueber echte HTML-Dateien und page-lokale Hash-Logik |
| Gemeinsame Komponentenbibliothek | Wiederverwendete Patterns sind kopiert oder per JS-Helper nachgebildet |

## 3. Warum die Seiten so organisiert sind

Die Seitenstruktur folgt nicht primar technischen Schichten, sondern **fachlichen Blickwinkeln**:

| Perspektive | Warum als eigene Seite? |
|---|---|
| Terminal / Jet Bridge | Lern- und Onboarding-Inhalte sind eher sequentielle Guides als Cockpit-Karten |
| Cockpit | Zentraler Feature-Hub mit Grid, Blade, Filtern und Deep Links |
| Ramp / Security / Runway / Tower | Jede Sicht braucht ein eigenes Interaktionsmodell und eine eigene Datenselektion |
| Flight Log / Pre-Flight / Wiring | Das sind Utility-Perspektiven mit stark abweichender Darstellung |

Der Vorteil: jede Seite bleibt isoliert, leicht statisch deploybar und fachlich lesbar. Der Preis: Navigation, Theme und Fehlerbilder sind mehrfach implementiert.

## 4. Seiten- und Komponentenmodell

| Seite | Hauptdatei | Rendering-Modell | Wichtige Komponenten |
|---|---|---|---|
| Cockpit | `index.html` + `app.js` | externes Skript | Zonen-Grid, Filterbar, Detail-Blade, Diagramm-/Code-/Media-Tabs |
| Terminal | `terminal.html` | inline | Plan-Karten, IDE-Setup, Exercises, Next Steps |
| Jet Bridge | `jet-bridge.html` | inline | Prompt-Techniken, Kontextkarten, Edit-Workflows, Agent-Patterns |
| Ramp | `ramp.html` | inline | Ramp-Grid, Blade, Perspektivfilter auf Instrumente |
| Runway | `runway.html` | inline | Verification Banner, Filterbar, Departure Board, Model Blade, Topology, NOTAMs |
| Security | `security.html` | inline | Luggage Lane, Scanner, Mermaid-Threat-Modell, Posture Checklist |
| Tower | `tower.html` | inline | Framework-Legende, Control-Liste, Sovereign Cloud, Flight Plans |
| Flight Log | `flight-log.html` | inline | Timeline, Statistiken, Filter |
| Pre-Flight | `preflight.html` | inline | Kategorien, Checkboxen, Fortschritt, Reset |
| Wiring | `wiring.html` | inline | Mermaid-Graph, Filter, Legende, Zonen- und Statistikansicht |

## 5. Laufzeitfluss pro Seite

### 5.1 Gemeinsames Muster

1. Theme aus `localStorage['cockpit-theme']` laden.
2. Seite oder zentrales Skript startet.
3. JSON-Daten per `fetch()` laden.
4. DOM rendern.
5. Hash-State anwenden.
6. Interaktionen binden.

### 5.2 Cockpit-Flow

`app.js` ist die einzige zentrale Laufzeitdatei fuer eine Seite mit eigenem Modulcharakter.

| Phase | Verhalten |
|---|---|
| Bootstrap | laedt `copilot-instruments.json` zwingend und drei Zusatzkataloge tolerant |
| Enrichment | baut `scannerIndex`, `governanceIndex` und ein `_engineModels`-Fallback fuer die EICAS-Zone |
| Rendering | rendert Zonen in fester Reihenfolge, inklusive Sonderlogik fuer EICAS und FMS |
| Interaktion | initialisiert Filter, lokale Suche, Blade, Mermaid und Browser-Back/Forward |
| Deep Link | `#instrument-<id>` oeffnet die passende Detailansicht |

### 5.3 Perspektiv-spezifische Flows

| Seite | Datenfluss | Besondere Logik |
|---|---|---|
| Security | 3 parallele Fetches | initiale Auswahl aus Hash, sonst `cockpit-last-scan`, sonst erster Threat |
| Runway | 1 Pflichtkatalog | Filter dimmen statt zu entfernen; Blade via `#model-<id>` |
| Tower | 3 Pflichtkataloge | zwei Deep-Link-Typen: `#control=` und `#sovereign=` |
| Wiring | 2 Pflichtkataloge | JSON wird zu Mermaid-Source kompiliert; Knoten verlinken zur Cockpit-Blade |
| Pre-Flight | 1 Pflichtkatalog | Statuspersistenz in `copilot-preflight` |

## 6. Navigationsmodell

### 6.1 Zwischen Seiten

Das Repo nutzt **echte Dateinavigation**:

- `index.html`
- `terminal.html`
- `jet-bridge.html`
- `ramp.html`
- `runway.html`
- `security.html`
- `tower.html`
- `flight-log.html`
- `preflight.html`
- `wiring.html`

Die Hauptnavigation ist in den Seiten wiederholt eingebettet, nicht zentral komponiert.

### 6.2 Innerhalb einer Seite

| Mechanismus | Verwendet in | Zweck |
|---|---|---|
| URL-Hash | Cockpit, Ramp, Runway, Security, Tower | Deep Links in Blade-, Scanner- oder Highlight-Ziele |
| `history.pushState` / `replaceState` | Cockpit, Runway, Security | URL aktualisieren ohne Seitenwechsel |
| `hashchange` | Ramp, Runway, Security, Tower | Deep Links auch bei Browser-Navigation anwenden |
| `localStorage` | nahezu alle Seiten | Theme oder page-lokalen Zustand halten |

## 7. Zustand und Persistenz

| Key | Seite | Bedeutung |
|---|---|---|
| `cockpit-theme` | fast alle Seiten | Dark/Light Theme |
| `copilot-preflight` | `preflight.html` | erledigte Checklisteneintraege |
| `cockpit-last-scan` | `security.html` | zuletzt betrachteter Threat |
| `cockpit-security-posture` | `security.html` | 9 Checkbox-Zustaende fuer Security Posture |

## 8. Zentrale Komponenten

### 8.1 `app.js`

Die Cockpit-Runtime stellt den dichtesten Logikkern des Repos:

| Bereich | Funktion |
|---|---|
| Datenbootstrap | laedt Instrumente und weiche Zusatzkataloge |
| Zonenrendering | verarbeitet Zone Order, EICAS-Cluster und FMS-Chain |
| Detail-Blade | generiert Tabs nur, wenn Daten vorhanden sind |
| Ressourcen-Tab | verknuepft externe Links, Security-Relevanz und Data-Quality-Hinweise |
| Filter/Suche | dimmt Karten, statt sie aus dem DOM zu entfernen |

### 8.2 `search.js`

`search.js` ist das globale Suchsystem ueber mehrere Perspektiven. Es baut einen kleinen In-Memory-Index aus Instrumenten, Controls, Modellen und Changelog-Eintraegen und oeffnet eine Command Palette ueber `Ctrl+K` bzw. `Cmd+K`.

### 8.3 Mermaid als Renderbaustein

Mermaid ist kein dekoratives Extra, sondern Teil mehrerer Kernseiten:

| Seite | Rolle |
|---|---|
| Cockpit | Diagrams-Tab in der Blade |
| Runway | Topology |
| Security | Threat Models |
| Tower | Sovereign Data Flow |
| Wiring | kompletter Verbindungsgraph |

## 9. Architekturfolgen fuer Wartung und Refactoring

| Beobachtung | Praktische Folge |
|---|---|
| Page-lokale Renderpipelines | Aenderungen muessen seitenweise gedacht und getestet werden |
| `copilot-instruments.json` als Hub | ID-Aenderungen haben hohe Seiteneffekte |
| Tolerantes Enrichment im Cockpit | fehlende Zusatzdaten sollen Cockpit nicht komplett lahmlegen |
| Kopierte Navigation und Theme-Logik | Inkonsistenzen koennen leicht auf einzelnen Seiten entstehen |
| Statisches Deployment | Laufzeitprobleme sind haeufig Daten-, Cache- oder CDN-Themen statt Build-Fehler |

## 10. Grenzen und gesicherte Unsicherheiten

| Thema | Was belegt ist | Was nur abgeleitet ist |
|---|---|---|
| Seitliche Organisation | HTML-Dateien, DOM-Struktur und Fetch-Logik | die inhaltliche Metapher als Produktstrategie |
| Datenkatalog | JSON-Schemas und Konsumenten im Code | semantische Vollstaendigkeit der Inhalte |
| Modellkatalog | Struktur und Rendering im Code | fachliche Aktualitaet einzelner Modellangaben |
| Framework-Mappings | Links und IDs im Katalog | externe Richtigkeit ohne manuelle Verifikation |

Weiterfuehrend: [`API-REFERENCE.md`](API-REFERENCE.md), [`DATA-CATALOG.md`](DATA-CATALOG.md), [`OPERATIONS.md`](OPERATIONS.md)
