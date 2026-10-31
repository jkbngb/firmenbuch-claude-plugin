# Firmenbuch für Claude (Plugin)

> English: the official Austrian company register (Firmenbuch) and the German commercial register (Handelsregister) inside Claude, via the MCP server at `mcp.agentic-firmenbuch.at`, with six German skills. The product and its users are German-speaking, so this README is in German.

Dieses Plugin verbindet Claude mit dem **österreichischen Firmenbuch** und dem **deutschen
Handelsregister** über den MCP-Server `https://mcp.agentic-firmenbuch.at/mcp` und bringt sechs
deutsche Skills mit, die Claude zeigen, welche Werkzeuge in welcher Reihenfolge zu verwenden sind.

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

| Skill | Was er tut |
|---|---|
| `firmen-screenen` | Firmenliste nach Branche, Bundesland und Mindestgröße erstellen |
| `firma-durchleuchten` | Profil, Kennzahlen, Historie und Dokumente einer Firma zusammenfassen |
| `peer-vergleich` | Eine Firma mit gleich großen Firmen vergleichen und einordnen |
| `geschaeftsfuehrer-finden` | Die Firmen finden, in denen eine Person eine Organfunktion hat |
| `neue-firmen-beobachten` | Neugründungen und Registeränderungen einer Branche in einer Region verfolgen |
| `liste-als-csv` | Eine gefilterte Firmenliste prüfen und als CSV exportieren |

## Beispiele

- „Aktive GmbHs in Oberösterreich mit Bilanzsumme über 5 Mio. €, sortiert nach Umsatz."
- „Vergleiche die Red Bull GmbH mit ihren Peers derselben Größenklasse und zeige die Kennzahlenentwicklung."
- „Welche Unternehmen in Bayern hatten im letzten Monat einen Geschäftsführerwechsel?"

## Zugang und Preis

50 Gratis-Credits für 7 Tage ab dem ersten Verbinden, danach Pro für 79 € netto im Monat mit
unbegrenzten Abfragen in Claude (Fair Use), monatlich kündbar. Verbindungstest, Feldreferenz und
`get_my_usage` bleiben frei. Preise: https://www.agentic-firmenbuch.at/preise.html

## Daten und Datenschutz

Das Plugin enthält keinen Code, der auf dem Rechner läuft, und keine Daten. Es sendet die Anfragen
von Claude (Suchbegriffe, Filter, Firmen-IDs) ausschließlich an `mcp.agentic-firmenbuch.at`; die
Anmeldung läuft per OAuth über dieselbe Adresse. Datenschutzerklärung:
https://www.agentic-firmenbuch.at/datenschutz.html

Support: jakob@agentic-firmenbuch.at · Impressum: https://www.agentic-firmenbuch.at/impressum.html

## Lizenz

MIT für die Dateien dieses Repositories (Manifest und Markdown). Server, Daten und Marke sind nicht
Teil dieser Lizenz.
