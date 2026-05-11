# Contributing

## 1. Scope: welches Repo ist gemeint?

Diese Dokumentation lebt im Ziel-Repo `C:\temp\copilot-cockpit-docs-final`, beschreibt aber das Quell-Repo `C:\temp\copilot-cockpit`.

| Ziel | Primaerer Arbeitsort |
| --- | --- |
| Doku verbessern | dieses Doku-Repo |
| Verhalten, Daten oder UI der Site aendern | Quell-Repo `C:\temp\copilot-cockpit` |
| beides synchron halten | zuerst Quell-Repo, dann Doku-Repo |

## 2. Beitragsstandard

Jede technische Aussage sollte mindestens eine dieser Formen von Beleg haben:

1. direkt lesbarer Code oder Konfiguration
2. JSON-Struktur im Quell-Repo
3. vorhandene Playwright-Spezifikation
4. explizite Repo-Markierung wie `verificationRequired`

Wenn etwas **nicht** explizit belegt ist, benenne es als Annahme oder Luecke.

## 3. Regeln fuer Doku-Aenderungen

| Regel | Erwartung |
| --- | --- |
| technische Praezision vor Marketing | beschreibe Dateien, Vertraege, Datenfluesse und Wartungsauswirkungen |
| Cross-References pflegen | verlinke auf die passende Detaildoku statt Inhalte zu duplizieren |
| Unsicherheit sichtbar machen | `verificationRequired` und "nicht im Repo belegt" nicht wegformulieren |
| Pfade und IDs exakt halten | Dateinamen, Hashes, JSON-Schluessel und `localStorage`-Keys muessen stimmen |
| Wartung mitdenken | nicht nur "was ist da", sondern auch "was bricht, wenn es sich aendert" dokumentieren |

## 4. Regeln fuer Quell-Repo-Aenderungen

| Bereich | Was besonders zu beachten ist |
| --- | --- |
| `data\copilot-instruments.json` | globale IDs, Zonen, `relatedInstruments`, Search, Deep Links |
| `data\copilot-models.json` | `#model-<id>`, Runway/Tower/Cockpit-Kopplung, Verifikationshinweise |
| `data\governance-controls.json` | `#control=<id>`, Wiring-Endpunkte, Compliance-Chips |
| Deep-Link-Logik | Search, Cross-Page-Callouts, Playwright |
| Mermaid-Inhalte | Syntax plus inhaltliche Referenzziele |
| page-lokale Skripte | es gibt kein zentrales Framework, daher Driftgefahr zwischen Seiten |

## 5. Empfohlener Arbeitsablauf

### 5.1 Bei Doku-only

1. Quell-Repo lesen, nicht raten.
2. Aussagen gegen Dateien, JSON und Tests verifizieren.
3. Nur die betroffenen Doku-Dateien aktualisieren.
4. Querverweise und Annahmen am Ende erneut pruefen.

### 5.2 Bei Code-/Datenaenderungen

1. Aenderung im Quell-Repo umsetzen.
2. Relevante Tests gemaess [TESTING-GUIDE.md](TESTING-GUIDE.md) waehlen.
3. Auswirkungen auf Hashes, Search und JSON-Referenzen mitpruefen.
4. Danach die passende Doku-Datei in diesem Repo nachziehen.

## 6. Welche Doku bei welcher Aenderung aktualisiert werden sollte

| Aenderung im Quell-Repo | Doku-Dateien |
| --- | --- |
| neue Seite oder neue Route | `API-REFERENCE.md`, oft `ARCHITECTURE.md` |
| neues Shared-Skript oder veraenderte Runtime-Rolle | `ARCHITECTURE.md`, `API-REFERENCE.md`, ggf. `OPERATIONS.md` |
| neuer oder geaenderter JSON-Katalog | `DATA-CATALOG.md`, ggf. `API-REFERENCE.md`, `OPERATIONS.md` |
| neue Hash-Form | `API-REFERENCE.md`, `ARCHITECTURE.md`, ggf. `TESTING-GUIDE.md` |
| neue Teststrategie / neue Spezifikation | `TESTING-GUIDE.md` |
| neue Ops-/Cache-/Deploy-Regel | `OPERATIONS.md` |

## 7. Review-Checkliste

### 7.1 Fuer Doku-Reviews

1. Ist jede starke Behauptung im Quell-Repo belegbar?
2. Stimmen Dateinamen, Schluessel, IDs und Hashes exakt?
3. Werden Unsicherheiten sichtbar statt geglaettet?
4. Sagt die Doku auch etwas ueber Wartungsauswirkungen aus?

### 7.2 Fuer Code-/Daten-Reviews

1. Welche Seiten konsumieren die geaenderte Datei?
2. Welche Hashes oder Search-Ziele haengen daran?
3. Welche Integritaetsregeln aus `tests\integrity.spec.js` koennen betroffen sein?
4. Muss die Doku synchron aktualisiert werden?

## 8. Dinge, die in Reviews oft uebersehen werden

| Thema | Warum es leicht uebersehen wird |
| --- | --- |
| Search-Ziele | `search.js` kodiert Hash-Ziele selbst und lebt getrennt von Seitenskripten |
| Cockpit-Callouts | `app.js` verlinkt in Security, Tower und Runway hinein |
| `verificationRequired` | Daten wirken "strukturiert", sind aber teilweise bewusst noch nicht final bestaetigt |
| Cache-Folgen | statische Aenderungen sind nicht automatisch sofort sichtbar |
| page-lokale Theme-/Boot-Logik | kein zentraler Komponentenlayer erzwingt Konsistenz |

## 9. Gute Commit- oder PR-Beschreibungen

Beschreibe bevorzugt den **Vertrag**, nicht nur die Datei:

- welche Seite oder welcher Katalog betroffen ist
- welche IDs, Hashes oder Konsumenten mitbetroffen sind
- welche Tests oder Quervergleiche die Aussage absichern
- ob etwas hart belegt oder als Annahme markiert ist

## 10. Nicht als Fakt dokumentieren

Vermeide in dieser Doku:

- Annahmen ueber Deploy-Trigger ohne Repo-Beleg
- "offizielle" Aussagen zu Modellen/Frameworks, wenn die Quelle selbst `verificationRequired` setzt
- implizite Backend- oder API-Features, die im Code nicht existieren
- pauschale Aussagen wie "nur redaktionelle Aenderung", wenn IDs, Hashes oder Diagramme betroffen sind

Weiterfuehrend: [ARCHITECTURE.md](ARCHITECTURE.md), [DATA-CATALOG.md](DATA-CATALOG.md), [OPERATIONS.md](OPERATIONS.md)
