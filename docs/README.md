# Copilot Cockpit - Technische Dokumentation

Diese Dokumentation beschreibt den technischen Stand des Quell-Repositories `C:\temp\copilot-cockpit`. Sie dokumentiert die **reale statische Laufzeit**, die JSON-Vertraege und die Wartungspfade; sie ist kein Produkt-Flyer.

## Schnellbild

| Thema | Verifizierter Stand |
| --- | --- |
| Laufzeitmodell | Statische Multi-Page-Webanwendung ohne Backend, Bundler oder Build-Schritt |
| HTML-Einstiegspunkte | 10 Seiten im Repo-Root (`index.html` plus 9 Perspektiv-/Hilfsseiten) |
| Gemeinsame Runtime | `app.js` fuer das Cockpit, `search.js` fuer globale Suche, `styles.css` fuer Layout/Theming |
| Inhaltsquellen | 11 JSON-Dateien unter `data\` |
| Navigation | Klassische HTML-Links plus Hash-basierte Deep Links |
| Persistenz | Nur `localStorage`; keine Cookies, keine serverseitige Session |
| Deployment | Statisches Hosting via `vercel.json` |
| Automatisierte Tests | Playwright-only: 11 Spezifikationen, 222 Testfaelle |

## Empfohlene Lesepfade

| Wenn du ... | Lies zuerst | Dann |
| --- | --- | --- |
| das System einordnen willst | [ARCHITECTURE.md](ARCHITECTURE.md) | [API-REFERENCE.md](API-REFERENCE.md) |
| wissen willst, welche Seite welche Daten liest | [API-REFERENCE.md](API-REFERENCE.md) | [DATA-CATALOG.md](DATA-CATALOG.md) |
| einen JSON-Katalog aendern willst | [DATA-CATALOG.md](DATA-CATALOG.md) | [TESTING-GUIDE.md](TESTING-GUIDE.md) |
| Deployment-, Cache- oder Refresh-Folgen abschaetzen willst | [OPERATIONS.md](OPERATIONS.md) | [ARCHITECTURE.md](ARCHITECTURE.md) |
| Doku- oder Repo-Aenderungen sauber reviewen willst | [CONTRIBUTING.md](CONTRIBUTING.md) | [TESTING-GUIDE.md](TESTING-GUIDE.md) |

## Systemgrenzen

1. **Keine Server-API.** Die "API-Oberflaeche" dieses Repos besteht aus HTML-Routen, JSON-Dateien, URL-Hashes, `localStorage`-Keys und wenigen globalen Browser-Funktionen.
2. **Kein Shared App-Framework.** `index.html` nutzt `app.js`; alle anderen Seiten booten ueber page-lokale Inline-Skripte und binden `search.js` zusaetzlich ein.
3. **Datengetriebene UI.** Ein grosser Teil der Funktionalitaet haengt an stabilen IDs in `data\*.json`, nicht an Klassen- oder Komponentenhierarchien.
4. **Cross-Page-Vertraege sind hart.** `#instrument-<id>`, `#model-<id>`, `#scan=<id>`, `#control=<id>` und `#sovereign=<id>` sind Integrationspunkte fuer Seiten, Suche und Tests.

## Was als belastbar gilt

Die folgenden Aussagen sind direkt im Quell-Repo belegt:

- `package.json` definiert nur **einen** Automationsskriptpfad: `npm test`.
- `playwright.config.js` startet dafuer einen lokalen Static Server auf `http://localhost:3000`.
- `vercel.json` konfiguriert statisches Hosting ohne Build-Schritt und setzt Cache-Header fuer `data\`, `*.js`, `*.css` und `media\`.
- `app.js` laedt den Cockpit-Katalog hart und weitere Kataloge soft-fail.
- `search.js` baut einen globalen Index aus Instrumenten, Controls, Modellen und Changelog-Eintraegen.

## Verifikationshinweise und Unsicherheiten

| Bereich | Im Quell-Repo explizit markiert | Bedeutung fuer diese Doku |
| --- | --- | --- |
| `data\copilot-models.json` | `verificationRequired: true` | Modellmatrix, Verfuegbarkeiten und Teile der Metadaten sind als Katalogstand zu lesen, nicht als final verifizierte Wahrheit. |
| `data\security-frameworks.json` | `verificationRequired: true`, viele Eintraege `verified: false` | Framework-Mappings und Ziel-URLs sind dokumentiert, aber bewusst als manuell nachzupruefen gekennzeichnet. |
| Release-/Ops-Prozess | nicht explizit dokumentiert | Betriebs- und Releasehinweise in dieser Doku bleiben deshalb beim technisch Nachweisbaren und markieren Luecken als Annahme. |

## Dokumentenkarte

- [ARCHITECTURE.md](ARCHITECTURE.md) - Laufzeitmodell, Seiten-/Skript-Topologie, Datenfluesse, Integrationsvertraege
- [API-REFERENCE.md](API-REFERENCE.md) - Routen, JSON-Endpunkte, Hash-Kontrakte, Browser-Persistenz
- [DATA-CATALOG.md](DATA-CATALOG.md) - Datenkataloge, Konsumenten, Kopplungen, Pflegefallen
- [TESTING-GUIDE.md](TESTING-GUIDE.md) - vorhandene Testbasis, Suite-Zuschnitt, Revalidierung pro Aenderung
- [OPERATIONS.md](OPERATIONS.md) - Deployment-, Cache-, Refresh- und Triage-Hinweise
- [CONTRIBUTING.md](CONTRIBUTING.md) - Beitragspfade, Belegstandard, Review-Checklisten, Doku-Synchronisation
