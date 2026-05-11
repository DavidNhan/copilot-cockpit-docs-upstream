# Operations

## 1. Betriebsmodell

Das Quell-Repo `C:\temp\copilot-cockpit` wird als **statische Website** betrieben. Es gibt keinen Laufzeitserver mit eigener Fachlogik; der operative Fokus liegt daher auf:

1. korrekten statischen Artefakten
2. unverletzten JSON-Katalogen
3. funktionierenden CDN-Abhaengigkeiten
4. sinnvollen Cache-Regeln

## 2. Deployment-Grundlagen

| Bereich | Stand |
|---|---|
| Plattform | Vercel |
| `framework` | `null` |
| `buildCommand` | leer |
| `outputDirectory` | `.` |
| Daten-Caching | `/data/*` fuer 1 Stunde, `must-revalidate` |
| JS/CSS-Caching | 1 Stunde, `must-revalidate` |
| Medien-Caching | 1 Jahr, `immutable` |

### Operative Konsequenzen

| Beobachtung | Folge |
|---|---|
| kein Build-Schritt | Releases koennen an Daten- oder Inhaltsfehlern scheitern, nicht an Bundling |
| `/data/*` wird gecacht | Content-Refresh ist nicht immer instant sichtbar |
| Medien sind stark gecacht | neue GIFs oder Assets brauchen konsequente Dateipflege und Versionsdisziplin |

## 3. Lokaler Betrieb

| Bedarf | Weg |
|---|---|
| Seite nur ansehen | einfacher statischer Server |
| Tests starten | `npm test` startet den Playwright-Webserver automatisch |
| Produktionsnahe Repro | lokaler Static-Server auf Port 3000 oder Playwright-Webserver |

Praktisch relevant: Playwright nutzt `python3 -m http.server 3000 --bind 127.0.0.1`. Wenn `python3` fehlt, scheitert der Standard-Testpfad bereits vor den eigentlichen Tests.

## 4. Content-Refresh

### Wann ein Refresh noetig ist

| Ausloeser | Typische Dateien |
|---|---|
| neue Copilot-Funktionen | `copilot-instruments.json`, `known-changelog-entries.json`, `wiring-diagram.json` |
| neue/veraenderte Modelle | `copilot-models.json`, ggf. Tower-Flight-Plans |
| neue Governance-/Compliance-Anforderungen | `governance-controls.json`, `sovereign-cloud.json` |
| neue Security-Erkenntnisse | `security-threats.json`, `security-frameworks.json` |
| Onboarding-/Guide-Anpassungen | `terminal-guide.json`, `jet-bridge-guide.json`, `preflight-checklist.json` |

### Empfohlener Refresh-Ablauf

1. Quelldaten im Quell-Repo aendern.
2. Cross-Referenzen auf IDs und Deep Links pruefen.
3. Zielseiten und zugehoerige Specs validieren.
4. Dokumentation in diesem Repo nachziehen, wenn Strukturen, Zaehler oder Arbeitsweisen betroffen sind.

## 5. Demo- und Medienbetrieb

Das Quell-Repo enthaelt einen GitHub-Workflow `record-demos.yml`.

| Schritt | Bedeutung |
|---|---|
| Trigger | Push auf `media/scripts/**` oder `workflow_dispatch` |
| Tools | `asciinema`, `agg` |
| Aktion | `./tools/record-demo.sh --all` |
| Ergebnis | neue/aktualisierte `media/recordings/*.gif` werden automatisch committed |

### Operativer Hinweis

Weil Medien stark gecacht werden, ist dieser Workflow nicht nur Komfort, sondern Teil der konsistenten Auslieferung von Demo-Artefakten.

## 6. Bekannte Risiken

| Risiko | Warum es relevant ist | Typischer Effekt |
|---|---|---|
| ID-Brueche in Hub-Katalogen | mehrere Seiten referenzieren dieselben IDs | leere Scanner-Items, kaputte Graphkanten, tote Deep Links |
| verifikationspflichtige Daten | Modelle und Security-Frameworks markieren sich selbst als noch nicht abschliessend verifiziert | Doku oder UI wirken sicherer als die Quelle ist |
| duplizierte Navigation/Theme-Logik | kein gemeinsamer Komponentenlayer | einzelne Seiten driften funktional auseinander |
| CDN-Abhaengigkeiten | Mermaid, Prism, Fonts und Vercel-Skripte liegen ausserhalb des Repos | Diagramme oder Syntax-Highlighting fehlen zur Laufzeit |
| statische Fehlerbilder | Pflichtdaten werden nur mit seitenlokalen Fallbacks behandelt | Nutzer sehen `DATA LINK LOST`, aber kein zentrales Incident-Handling |
| veraltete Copy in Kommentaren | einzelne Kommentare nennen aeltere Zaehlerstaende | Review-Konfusion trotz technisch korrekter Daten |

## 7. Beobachtbare Betriebsindikatoren

Da das Repo keinen eigentlichen Backend-Monitoring-Stack mitbringt, sind dies die praktischsten Signale:

| Signal | Woran man es sieht |
|---|---|
| Pflichtdaten fehlen | Seite zeigt `DATA LINK LOST` |
| Mermaid-Probleme | Diagramme fehlen oder fallen auf Text/Fallback zurueck |
| Deep-Link-Probleme | Blade/Highlight/Scanner oeffnet nicht |
| Content-Drift | UI-Zaehler, Listenlaengen oder README-Angaben passen nicht mehr zusammen |
| Testdrift | betroffene Seitenspec oder `integrity.spec.js` bricht |

## 8. Incident-Triage fuer typische Fehler

| Symptom | Erste Pruefung | Wahrscheinliche Ursache |
|---|---|---|
| Seite bootet, aber Inhalt fehlt | betroffene JSON-Datei vorhanden und gueltig? | kaputte Datei oder falscher Schluessel |
| nur eine Perspektive kaputt | page-lokales Skript und Zielkatalog ansehen | seitenlokale Renderlogik gebrochen |
| Wiring- oder Security-Graph defekt | Mermaid und referenzierte Daten pruefen | Diagrammsyntax oder Cross-Reference-Problem |
| Deep Links ohne Wirkung | Hash-Format und Ziel-ID pruefen | ID-Drift oder fehlender Listener |
| neue Inhalte erscheinen nicht sofort | Cache-Regeln und Asset-Pfade beachten | erwartbares Cache-Fenster |

## 9. Was im Repo fuer Betrieb nicht explizit belegt ist

| Thema | Status |
|---|---|
| formaler Release-Prozess fuer Produktionsdeploys | nicht explizit dokumentiert |
| zentrales Observability-/Alerting-Konzept | nicht im Repo belegt |
| automatisierte Content-Freshness-Pipeline fuer alle Datenkataloge | nicht belegt; nur einzelne Hinweise und Demo-Workflow vorhanden |

Weiterfuehrend: [`TESTING-GUIDE.md`](TESTING-GUIDE.md), [`DATA-CATALOG.md`](DATA-CATALOG.md), [`CONTRIBUTING.md`](CONTRIBUTING.md)
