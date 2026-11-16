# Firmenbuch für Claude (Plugin)

> English: the official Austrian company register (Firmenbuch) and the German commercial register (Handelsregister) inside Claude, via the MCP server at `mcp.agentic-firmenbuch.at`, with fourteen German skills: nine workflows (screen, deep-dive, peers, person search, watch, CSV, research into Excel, succession screening, market-segment overview) and five output formats (standardised company card, multi-year balance-sheet table, key-figure chart from a fixed template, sector comparison, data rules), plus `BEST_PRACTICES.md`. The product and its users are German-speaking, so this README is in German.

Dieses Plugin verbindet Claude mit dem **österreichischen Firmenbuch** und dem **deutschen
Handelsregister** über den MCP-Server `https://mcp.agentic-firmenbuch.at/mcp` und bringt vierzehn
deutsche Skills mit: neun Arbeitsabläufe, die Claude zeigen, welche Werkzeuge in welcher
Reihenfolge zu verwenden sind, und fünf Ausgabeformate, die jede Antwort gleich aufbauen, damit
sie überfliegbar und zwischen Firmen und Ländern vergleichbar ist.

**Skills sind für Claude, nicht für Sie.** Sie müssen keinen Skill kennen oder aufrufen: Sie sagen
in Worten, was Sie wollen, Claude wählt den passenden Skill und hält sich an sein Format. Die
Tabellen unten zeigen nur, was Sie bekommen.

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
| `recherche-excel` | Viele Firmen nach Kriterien recherchieren und alle Stammdaten und Kennzahlen als Excel-Datei liefern (zwei Blätter: Firmen, Abfrage) |
| `nachfolge-screening` | Firmen mit Nachfolgesignalen aus dem Register (alternde Führung, keine jüngere namensgleiche Person im Organ, Führungswechsel, Umgründung) in Region und Größenband, als Liste mit Kennzeichen statt Ausschlüssen |
| `marktsegment-uebersicht` | Einen Teilmarkt vermessen: Zahl der Firmen nach Bundesland und Größenklasse, die zehn größten und ihr Anteil, Wachstumsprofile; mehrere Segmente im Vergleich |

**Ausgabeformate** (wie das Ergebnis aussieht, immer gleich aufgebaut):

| Skill | Was er tut |
|---|---|
| `firmenkarte` | Eine Firma als standardisierte Karte: 18 feste Zeilen von Rechtsform bis Datenstand, Österreich und Deutschland gleich |
| `bilanz-tabelle` | Bilanz, GuV und Kennzahlen als Mehrjahrestabelle (bis fünf Jahre, feste Positionen, T€, Aufwände negativ) |
| `kennzahlen-grafik` | Bilanzsumme, Eigenkapital, Umsatz, Ergebnis und Quoten über die Jahre als Grafik, vom Server fertig gezeichnet (`chart_company_history`) und direkt im Gespräch gezeigt; die Zahlen dazu als Tabelle |
| `branchen-vergleich` | Die Firma gegen Firmen derselben Branche (NACE-Abschnitt oder enger) und Größenklasse: Tabelle, Median, Rang, Perzentile |
| `datenregeln` | Die fünf Regeln für jede Zahl: nur Registerdaten, fehlende Werte nie als 0, Jahr und Einheit, keine Feldnamen |

Die Begründung hinter den Formaten steht in [`BEST_PRACTICES.md`](BEST_PRACTICES.md).

## So sagen Sie es, das bekommen Sie

| Sie sagen | Sie bekommen |
|---|---|
| „Firmenkarte der Red Bull GmbH." | Eine Tabelle mit 18 Zeilen: Rechtsform, Sitz, Register-ID, Branche, Geschäftsführung, Größe, letzter Abschluss, Bilanzsumme, Eigenkapitalquote, Umsatz, Datenstand |
| „Bilanz und GuV der Hermann Pfanner Getränke GmbH als Tabelle." | Drei Tabellen (Bilanz, GuV, Kennzahlen), bis fünf Jahre nebeneinander, Trend |
| „Zeichne die Entwicklung von Umsatz und Ergebnis der Red Bull GmbH." | Eine fertige Grafik im Gespräch (Beträge als Balken, Quoten als Linien, Quelle und Datenstand) und die Zahlen als Tabelle |
| „Wie steht die Adolf Darbo AG in ihrer Branche da?" | Tabelle mit zehn Vergleichsfirmen derselben Branche und Größe, Median, Rang; auf Wunsch Perzentile |
| „Alle aktiven Getränkehersteller in Österreich über 10 Mio. € Bilanzsumme als Excel." | Eine Excel-Datei mit allen Firmen und 24 Spalten, dazu ein Blatt mit Abfrage, Datum und Quelle |
| „Firmen in Oberösterreich mit 15 bis 80 Mio. € Umsatz, deren Geschäftsführung über 60 ist und die keinen Nachfolger im Organ haben." | Eine Liste mit Kennzeichen je Firma (ältestes führendes Organ als Jahrgang, jüngere Namensgleiche ja/nein/nicht prüfbar, Führungswechsel, Umgründung), ohne Personennamen und ohne Score |
| „Wie groß ist der Maschinenbau in Bayern, nach Regierungsbezirk und Größe?" | Zählungen nach Region und Größenklasse, die zehn größten Firmen mit Anteil, Wachstumsprofile |
| „Welche Firmen in Bayern hatten im letzten Monat einen Geschäftsführerwechsel?" | Eine Liste der Registeränderungen mit Datum und Firma |
| „Wo ist Max Mustermann Geschäftsführer?" | Die Firmen mit dieser Organfunktion, mit dem Hinweis, dass gleiche Namen nicht dieselbe Person sind |

Fehlt ein Wert, steht „–" mit dem Grund, nie eine 0. Jede Zahl trägt ihr Jahr. Jede Antwort endet
mit Quelle und Datenstand.

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
