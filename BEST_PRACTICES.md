# Best Practices · Registerdaten mit Claude richtig lesen und darstellen

> English: the house rules for presenting Austrian Firmenbuch and German Handelsregister data that the
> MCP server at `mcp.agentic-firmenbuch.at` serves. The skill `datenregeln` is the short form Claude
> loads automatically; this file carries the reasoning and the exact field names.

Diese Regeln gelten für jede Antwort, Tabelle und Grafik, die Zahlen aus dem Firmenbuch-MCP-Server
enthält. Die Skills `firmenkarte`, `bilanz-tabelle`, `kennzahlen-grafik`, `branchen-vergleich` und `recherche-excel`
setzen sie um; `datenregeln` ist die Kurzfassung, die Claude vor jeder Zahl lädt.

## 1. Warum ein festes Format

Dieselbe Firma soll heute und in drei Monaten, für Österreich und für Deutschland, gleich aussehen.
Ein festes Zeilenschema (Firmenkarte) und feste Positionen (Bilanz-Tabelle) machen Antworten
überfliegbar und vergleichbar, und sie zeigen, **was fehlt**: eine Zeile mit „–" und Grund sagt mehr
als eine weggelassene Zeile.

## 2. Werkzeuge: Reihenfolge und Kosten

| Schritt | Werkzeug | Wann |
|---|---|---|
| 1 | `search_companies` | immer zuerst, wenn nur ein Name vorliegt; `page_size` 3 beim Nachschlagen |
| 2 | `get_company_details` | das Profil **einer** Firma; enthält Bilanz-, GuV- und Kennzahlenreihen |
| 3 | `get_company_history` | nur die Reihen (für Grafiken), `metrics` eng setzen |
| 4 | `find_peers`, `benchmark_company` | Vergleich; Perzentile nur Österreich und nur im Pro-Plan |
| 5 | `list_documents` → `get_document` / `get_document_text` | Originale, nur auf Nachfrage |
| immer frei | `ping`, `get_coverage`, `describe_fields`, `list_sectors`, `get_my_usage` | Abdeckung, Felder, Branchencodes, Budgetstand |

Das Budget zählt **jede ausgelieferte Firma** einmal im Kalendermonat, nicht nur geöffnete Profile.
Eine Suche mit `page_size` 25 kostet 25 Firmen, auch wenn nur eine gemeint war. Darum: eng filtern,
klein blättern, Branchen mit `list_sectors` auflösen statt mehrere Suchen zu probieren.

## 3. Fehlende Werte sind nie Null

**Deutschland.** Jahresabschlüsse werden Bundesland für Bundesland erfasst (`get_coverage` nennt den
Stand). `financials.reporting_status` im Profil sagt, warum Zahlen fehlen:

| Wert | Bedeutung | So schreiben |
|---|---|---|
| `available` | mindestens ein geparster Abschluss | Zahlen zeigen |
| `filed_unparsed` | Abschlüsse im Register, aber nur als PDF/ESEF | „– nur als PDF veröffentlicht" |
| `none_published` | Quelle geprüft, kein Abschluss ab FY2022 | „– kein Abschluss veröffentlicht" |
| leer | Bundesland noch nicht erfasst | „– noch nicht erfasst" |

Kleine deutsche Gesellschaften veröffentlichen nach § 326 HGB **keine GuV**; ein fehlender Umsatz ist
deshalb der Normalfall, kein Hinweis auf eine Firma ohne Geschäft.

**Österreich.** Kleine Gesellschaften reichen nach § 279 UGB eine **verkürzte GuV** oder nur eine
Bilanz ein. `financials.has_guv_latest` false heißt: keine GuV-Zahlen, `guv` ist `{}`, Margen bleiben
leer. Rund 40 % der GuVs tragen statt `umsatzerloese` nur das `rohergebnis`; `revenue_basis` nennt die
Basis des Umsatzwerts, `revenue_basis_change` den Wechsel.

**Beschäftigte** (`employees.latest`) sind nur für rund 20 % der österreichischen Firmen bekannt und
stammen meist aus einem älteren Jahr als die Bilanz (`employees.latest_year`). Deutschland liefert
fast nie eine Zahl. Immer das Jahr dazu, nie „Umsatz je Mitarbeiter" aus verschiedenen Jahren.

## 4. Jahr, Einheit, Vorzeichen

- **Jede Zahl trägt ihr Jahr.** `financials.latest_year` ist das Bilanzjahr; `revenue_year` und
  `employees_year` können abweichen und werden dann genannt: „Umsatz 12.345 T€ (2023)".
- **Euro in T€** mit Tausenderpunkt („12.345"), über 100 Mio. € in Mio. € mit einer Nachkommastelle
  („1.234,5 Mio. €"), Quoten in % mit einer Nachkommastelle. `equity_ratio` kommt als Bruchteil
  (0,35 = 35,0 %).
- **Aufwände sind negativ** (Materialaufwand, Personalaufwand, Abschreibungen), seit 2026-11-04
  unabhängig davon, wie der Einreicher sie übermittelt hat (`sign_convention`, `anomalies` mit
  `expense_sign_normalised`). Nicht umdrehen, nicht „bereinigen".
- **EBIT ist nicht gleich EBIT.** `operating_result` ist der Betriebserfolg (vor Finanzergebnis),
  `ebit_strict` das EBIT einschließlich Finanzergebnis, `ebitda` = Betriebserfolg + Abschreibungen.
  Bei `guv_konfidenz` `low` fehlen `ebitda` und `ebit_strict` absichtlich: nicht nachrechnen.
- **Konzern.** `financials.konzern` true heißt, die Reihen sind ein Konzernabschluss. Immer so
  beschriften und nie mit Einzelabschlüssen in einen Median mischen.
- **Fassung und Herkunft.** `filings[].fassung` `corrected` / `preliminary` nennen;
  `parsed_source` `pdf_text_llm_v1` als „aus dem amtlichen PDF extrahiert" kennzeichnen.

## 5. Vergleiche, die etwas aussagen

- `peer_percentiles` (Profil) und `benchmark_company` ordnen **innerhalb der UGB-Größenklasse**
  ein, nicht innerhalb der Branche. So beschriften.
- `find_peers` sucht in Österreich die größennächsten Firmen derselben Branche so eng wie möglich
  (Klasse → Gruppe → Abteilung → Abschnitt) und sagt in `same_sector_level` und `note`, wie eng es
  wurde und ob Firmen anderer Branchen auffüllen. In Deutschland sind Peers die nach Tätigkeitstext
  ähnlichsten Firmen, kein Zahlenvergleich. `cross_border` ist ein Bedeutungsvergleich.
- Eigene Vergleichsgruppen über `search_companies` mit `nace_section` (Abschnitt, NACE Level 1),
  `oenace_division` (Abteilung) oder `oenace_group` (Gruppe), plus `size_gkl` und `status` `active`.
  ÖNACE 2025 = WZ = NACE Rev. 2.1, derselbe Code gilt in beiden Ländern. Die Abschnittsebene ist
  breit („C Herstellung von Waren"); wenn genug Firmen da sind, ist die Abteilung aussagekräftiger.
- Gleiches gegen Gleiches: `umsatzerloese` gegen `umsatzerloese`, nie gegen `rohergebnis`; Jahre
  nennen; Median nur über vorhandene Werte und die Anzahl dazu („Median über 7 von 10").

## 6. Personen

Namen der Organe stammen aus dem öffentlichen Register und werden für die Einzelabfrage gezeigt;
Geburtsdaten nur jahresgenau. Keine personenbezogenen Listen über viele Firmen, keine Verknüpfung
mit anderen Quellen, keine Vorschläge zur Ansprache. Gleiche Namen sind nicht dieselbe Person.
Wer eine Sperre nach Art. 21/17 DSGVO beantragt hat, erscheint nicht mehr; Antworten dürfen das nicht
umgehen.

## 7. Quelle, Stand, Gewähr

Unter jede Tabelle und Grafik: Quelle („Firmenbuch, BMJ, CC BY 4.0" bzw. „Handelsregister und
Unternehmensregister der Länder") und `provenance.built_at` als Datenstand. Die Daten werden
automatisiert verarbeitet und ohne Gewähr bereitgestellt; maßgeblich ist der amtliche Auszug, den
`get_document` liefert.

## 8. Checkliste vor dem Absenden

1. Steht an jeder Zahl das Jahr, an jedem Betrag die Einheit?
2. Ist jede fehlende Zahl „–" mit Grund, nirgends 0?
3. Sind Aufwände negativ, Konzern und Fassung genannt?
4. Vergleicht die Tabelle gleiche Kennzahlen gleicher Jahre?
5. Steht die Quelle mit Datenstand darunter?
6. Wurden nur so viele Firmen geladen, wie die Frage braucht?

## 9. Grafik und Excel: Vorlage statt Gestaltung

Eine Grafik, die Claude jedes Mal neu entwirft, dauert Minuten, kostet ein Vielfaches und sieht
jedes Mal anders aus. Darum zeichnet `kennzahlen-grafik` aus der festen Vorlage
`skills/kennzahlen-grafik/vorlage.md` (ein HTML-Dokument in einem Codeblock): zwei Grafiken (Beträge als Balken, Quoten als Linien),
Tabellen und Fußzeile entstehen aus **einem Datenblock**, den Claude füllt. Nichts am Aussehen wird
entschieden, kein eigenes HTML, SVG, React oder Python geschrieben. In claude.ai, Claude Desktop und
Cowork erscheint die gefüllte Vorlage als Artefakt, in Claude Code als Datei. Vor der Grafik stehen
die Zahlen immer als Texttabelle, damit niemand die Grafik öffnen muss, um die Werte zu lesen.

Für Recherchen über viele Firmen schreibt `recherche-excel` eine Datei mit festen Spalten (Blatt
„Firmen") und einem Blatt „Abfrage" (Filter, Datum, Trefferzahl, Quelle, Budget). Die Trefferkarten
von `search_companies` tragen alle Spalten; `get_company_details` ist nur für ein Zusatzblatt
„Details" nötig. Keine Personenspalte: Listen von Organen über viele Firmen sind nicht zulässig
(Abschnitt 6).

## 10. Die fünf Regeln, wortgleich in jedem Format

1. Nur Registerdaten: nichts aus Vorwissen, Websites oder Geschäftsberichten.
2. Fehlt ein Abschluss, nur der Befund: keine Vermutung über Rechtsform, Größe, Konzern oder
   Offenlegungspflicht als Grund.
3. Nie 0 für einen fehlenden Wert: „–" mit Grund.
4. Jede Zahl mit Jahr und Einheit.
5. Keine Feldnamen des Servers in der Antwort: Worte statt `has_pdf_only` oder `series_sign_flip`.

Die Regeln stehen in `datenregeln` mit einer Tabelle „Verboten und richtig" und wortgleich in jedem
Format-Skill, damit sie auch dann gelten, wenn nur ein Skill geladen wird.
