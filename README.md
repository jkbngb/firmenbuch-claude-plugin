# Firmenbuch für Claude (Plugin)

> English: the official Austrian company register (Firmenbuch) and the German commercial register (Handelsregister) inside Claude, via the MCP server at `mcp.agentic-firmenbuch.at`, with eleven German skills: six workflows (screen, deep-dive, peers, person search, watch, CSV) and five output formats (standardised company card, multi-year balance-sheet table, key-figure chart, sector comparison, data rules), plus `BEST_PRACTICES.md`. The product and its users are German-speaking, so this README is in German.

Dieses Plugin verbindet Claude mit dem **österreichischen Firmenbuch** und dem **deutschen
Handelsregister** über den MCP-Server `https://mcp.agentic-firmenbuch.at/mcp` und bringt elf
deutsche Skills mit: sechs Arbeitsabläufe, die Claude zeigen, welche Werkzeuge in welcher
Reihenfolge zu verwenden sind, und fünf Ausgabeformate, die jede Antwort gleich aufbauen, damit
sie überfliegbar und zwischen Firmen und Ländern vergleichbar ist.

Österreich: alle eingetragenen Firmen mit Jahresabschlüssen, Kennzahlen, Organen und Urkunden.
Deutschland: Stammdaten aller Handelsregisterfirmen; Jahresabschlüsse Bundesland für Bundesland im
Aufbau, `get_coverage` nennt den aktuellen Stand. Alle 19 Werkzeuge lesen nur; jede Zahl stammt aus
dem amtlichen Register.

## Installation

Nach der Freigabe im Claude-Verzeichnis: das Plugin in claude.ai hinzufügen (Verzeichnis → Plugins),
oder in Claude Code:

```
claude plugin install firmenbuch@claude-plugins-official
```

Aus diesem Repository direkt (eine Sitzung, zum Testen):

```
claude --plugin-dir ./firmenbuch-claude-plugin
```

Beim ersten Datenaufruf öffnet Claude die Anmeldung per E-Mail-Code (OAuth, kein API-Key). Anleitung
mit Bildern: https://www.agentic-firmenbuch.at/cowork.html

## Skills

**Arbeitsabläufe** (welche Werkzeuge, in welcher Reihenfolge):

| Skill | Was er tut |
|---|---|
| `firmen-screenen` | Firmenliste nach Branche, Bundesland und Mindestgröße erstellen |
| `firma-durchleuchten` | Profil, Kennzahlen, Historie und Dokumente einer Firma zusammenfassen |
| `peer-vergleich` | Eine Firma mit gleich großen Firmen vergleichen und einordnen |
| `geschaeftsfuehrer-finden` | Die Firmen finden, in denen eine Person eine Organfunktion hat |
| `neue-firmen-beobachten` | Neugründungen und Registeränderungen einer Branche in einer Region verfolgen |
| `liste-als-csv` | Eine gefilterte Firmenliste prüfen und als CSV exportieren |

**Ausgabeformate** (wie das Ergebnis aussieht, immer gleich aufgebaut):

| Skill | Was er tut |
|---|---|
| `firmenkarte` | Eine Firma als standardisierte Karte: 18 feste Zeilen von Rechtsform bis Datenstand, Österreich und Deutschland gleich |
| `bilanz-tabelle` | Bilanz, GuV und Kennzahlen als Mehrjahrestabelle (bis fünf Jahre, feste Positionen, T€, Aufwände negativ) |
| `kennzahlen-grafik` | Bilanzsumme, Eigenkapital, Umsatz, Ergebnis und Quoten über die Jahre als Grafik (Artefakt, Datei oder Text) |
| `branchen-vergleich` | Die Firma gegen Firmen derselben Branche (NACE-Abschnitt oder enger) und Größenklasse: Tabelle, Median, Rang |
| `datenregeln` | Die Regeln für jede Zahl: fehlende Werte nie als 0, Jahr und Einheit, Vorzeichen, Budget, Personen |

Die Begründung hinter den Formaten steht in [`BEST_PRACTICES.md`](BEST_PRACTICES.md).

## Beispiele

- „Aktive GmbHs in Oberösterreich mit Bilanzsumme über 5 Mio. €, sortiert nach Umsatz."
- „Vergleiche die Red Bull GmbH mit ihren Peers derselben Größenklasse und zeige die Kennzahlenentwicklung."
- „Welche Unternehmen in Bayern hatten im letzten Monat einen Geschäftsführerwechsel?"
- „Firmenkarte der Palfinger AG." · „Bilanz und GuV der letzten fünf Jahre als Tabelle."
- „Zeichne die Umsatz- und Ergebnisentwicklung." · „Wie steht die Firma in ihrer Branche da?"

## Zugang und Preis

Dauerhaft gratis, ohne Karte und ohne Ablaufdatum: **10 Firmen pro Kalendermonat**, Suchtreffer bis
10 Zeilen pro Abfrage. Eine Firma zählt pro Monat einmal, egal wie oft sie angesehen wird. Darüber
hinaus zwei Pläne, monatlich kündbar: **Solo** für 29 € netto im Monat mit 500 Firmen pro Monat und
100 Treffern pro Abfrage (Stammdaten, Organe, Kennzahlen, Historie, Personensuche), und **Pro** für
79 € netto im Monat mit beliebig vielen Firmen im Rahmen der Fair Use, dazu Originaldokumente
(Jahresabschluss im Volltext, Registerauszug als PDF), Peer-Suche und registerweite Statistik.
Verbindungstest, Feldreferenz und `get_my_usage` bleiben immer frei. Aktuelle Preise:
https://www.agentic-firmenbuch.at/preise.html

## Daten und Datenschutz

Das Plugin enthält keinen Code, der auf dem Rechner läuft, und keine Daten. Es sendet die Anfragen
von Claude (Suchbegriffe, Filter, Firmen-IDs) ausschließlich an `mcp.agentic-firmenbuch.at`; die
Anmeldung läuft per OAuth über dieselbe Adresse. Datenschutzerklärung:
https://www.agentic-firmenbuch.at/datenschutz.html

Support: jakob@agentic-firmenbuch.at · Impressum: https://www.agentic-firmenbuch.at/impressum.html

## Lizenz

MIT für die Dateien dieses Repositories (Manifest und Markdown). Server, Daten und Marke sind nicht
Teil dieser Lizenz.
