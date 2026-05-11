# Contributing

## 1. Beitragspfade kurz erklaert

Dieses Doku-Repo beschreibt das Quell-Repo `C:\temp\copilot-cockpit`. Deshalb gibt es zwei unterschiedliche Beitragssituationen:

| Du willst ... | Arbeite primaer in ... |
|---|---|
| nur die Dokumentation verbessern | diesem Repo |
| Verhalten, Daten oder UI der Site aendern | `C:\temp\copilot-cockpit` |
| beides synchron halten | zuerst Quell-Repo, dann Doku-Repo |

## 2. Grundregeln

1. **Keine Behauptungen ohne Beleg im Quell-Repo.**
2. **Unsicherheiten explizit benennen**, statt sie weich zu formulieren.
3. **Dateinamen, IDs und Zaehlerstaende** gegen das Quell-Repo pruefen.
4. **Hub-Daten respektieren**: `copilot-instruments.json` und `copilot-models.json` haben Seiteneffekte.

## 3. Stil fuer Doku-Beitraege

| Regel | Erwartung |
|---|---|
| Schreibe fuer Maintainer, nicht fuer Marketing | konkret, knapp, handlungsorientiert |
| Nutze Tabellen, wenn Beziehungen wichtig sind | Routen, Konsumenten, Risiken, Validierung |
| Vermeide Wiederholungen | einmal sauber erklaeren, dann querverweisen |
| Markiere Grenzen | z. B. "im Repo nicht explizit belegt" |
| Bleibe datei- und verhaltensnah | nicht nur "was", sondern auch "wo" und "wodurch" |

## 4. Stil fuer Code- und Datenbeitraege im Quell-Repo

| Bereich | Erwartung |
|---|---|
| JSON | IDs stabil halten, Strukturen konsistent erweitern, Referenzen pruefen |
| HTML/JS | page-lokale Muster respektieren; keine implizite SPA annehmen |
| Navigation/Theme | Seitenkonsistenz mitdenken, weil Logik dupliziert ist |
| Diagramme | Mermaid-Syntax und Ziel-IDs pruefen |
| Copy | keine alten Zaehlerstaende oder unbewiesenen Aussagen fortschreiben |

## 5. Review-Checklisten

### Doku-Review

1. Sind Aussagen gegen `C:\temp\copilot-cockpit` belegbar?
2. Stimmen Pfade, Dateinamen, Counts und Linkziele?
3. Gibt es klare Leserpfade und Querverweise?
4. Werden Unsicherheiten korrekt markiert?

### Code-/Daten-Review

1. Welche Seiten konsumieren die geaenderte Datei?
2. Sind IDs, Hash-Ziele und Search-Eintraege weiter konsistent?
3. Welche Specs sind mindestens noetig?
4. Muss die Doku nachgezogen werden?

## 6. Empfohlener Arbeitsablauf

### Bei Doku-Only

1. Quell-Repo lesen.
2. Betroffene Doku-Seiten aktualisieren.
3. Querverweise pruefen.
4. Zahlen und Unsicherheiten noch einmal gegenchecken.

### Bei Code-/Daten-Aenderungen

1. Aenderung im Quell-Repo umsetzen.
2. Betroffene Tests laut [`TESTING-GUIDE.md`](TESTING-GUIDE.md) waehlen.
3. Danach Doku aktualisieren, falls Architektur, API-Flaeche, Datenmodell oder Betriebspfad betroffen sind.

## 7. Was in Reviews oft vergessen wird

| Thema | Warum es gerne uebersehen wird |
|---|---|
| Deep Links | sie liegen nicht zentral, sondern pro Seite |
| Search-Index | `search.js` nutzt mehrere Kataloge quer ueber das Repo |
| Data-Quality-Hinweise | einzelne Kataloge markieren sich selbst als unvollstaendig verifiziert |
| Cache-Auswirkungen | statische Aenderungen sind nicht immer sofort sichtbar |
| veraltete Zaehler in Kommentaren | historische Copy kann sich mit aktuellem Datenstand beissen |

## 8. Release-Hinweise

Dieses Repo dokumentiert keinen vollstaendig formalisierten Release-Prozess. Fuer saubere Auslieferungen sind dennoch diese Punkte sinnvoll:

| Vor einem inhaltlichen Release | Vor einem Doku-Release |
|---|---|
| betroffene Specs ausfuehren | Pfade, Querverweise, Counts und Schluessel validieren |
| `verificationRequired`-Hinweise ernst nehmen | keine staerkeren Zusagen als die Quelle machen |
| Cache-relevante Aenderungen bedenken | Screenshots/Beispiele nicht vom alten Stand uebernehmen |
| Demo-Artefakte bei Bedarf aktualisieren | Leserpfade aktuell halten |

## 9. Gute Commit-/PR-Beschreibungen

Beschreibe nicht nur die Datei, sondern den Vertrag:

- welche Seite oder welcher Katalog betroffen ist
- welche Deep Links, Counts oder Konsumenten sich mitveraendern
- welche Validierung angewendet wurde
- ob eine Aussage hart belegt oder bewusst vorsichtig formuliert ist

## 10. Wann diese Doku angepasst werden sollte

| Aenderung im Quell-Repo | Doku nachziehen? |
|---|---|
| neue Seite / neue Route | ja, `API-REFERENCE.md`, oft auch `ARCHITECTURE.md` |
| neuer JSON-Katalog | ja, `DATA-CATALOG.md`, oft `OPERATIONS.md` |
| neue Deep-Link-Form | ja, `API-REFERENCE.md` und ggf. `TESTING-GUIDE.md` |
| neue Teststrategie oder Workflow | ja, `TESTING-GUIDE.md` oder `OPERATIONS.md` |
| nur redaktionelle JSON-Copy | meist nein, ausser Counts, Risiken oder Leserpfade aendern sich |
