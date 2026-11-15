---
name: datenregeln
description: Grundregeln für das Lesen und Darstellen von Firmenbuch- und Handelsregisterdaten aus dem Firmenbuch-MCP-Server (fehlende Werte, Jahresangaben, Vorzeichen, Einheiten, Budget, Personendaten). Verwenden, bevor Zahlen aus search_companies, get_company_details oder get_company_history in eine Antwort, Tabelle oder Grafik übernommen werden, und bei jeder Frage nach Verlässlichkeit, Aktualität oder Herkunft der Daten.
---

# Datenregeln

Die kurze Fassung der Regeln, die jede Antwort mit Registerzahlen einhalten muss. Ausführlich mit
Begründung: `BEST_PRACTICES.md` im Plugin.

## Werkzeuge in der richtigen Reihenfolge

1. `search_companies` zuerst, mit engem Filter und kleinem `page_size`; eine Register-ID direkt.
2. `get_company_details` für **eine** Firma. Es enthält schon Bilanz-, GuV- und Kennzahlenreihen.
3. `get_company_history` für Reihen ohne das übrige Profil (Grafiken), `get_full_record` nur für
   alle UGB-Positionen, `list_documents` / `get_document` / `get_document_text` nur auf Nachfrage.
4. Branchen über `list_sectors`, Abdeckung über `get_coverage`, Felder über `describe_fields`.

## Fehlende Werte

- **Nie 0 für einen fehlenden Wert.** Deutschland: `financials.reporting_status` → `available`
  (Zahlen da), `filed_unparsed` (nur PDF/ESEF veröffentlicht), `none_published` (kein Abschluss
  im Register), leer (noch nicht erfasst, Aufbau Bundesland für Bundesland, `get_coverage`).
  Österreich: `has_guv_latest` false heißt verkürzte Einreichung (nur Bilanz), `employees.latest`
  null heißt keine Angabe (rund 80 % der Firmen).
- Ein Feld, das im Profil fehlt, bleibt in der Tabelle als „–" mit Grund; es wird nicht gestrichen
  und nicht geschätzt.

## Jahr und Einheit

- Jede Zahl mit ihrem Jahr. Umsatz (`revenue_year`) und Beschäftigte (`employees_year`) stammen
  oft aus einem älteren Jahr als die Bilanz; nie „Umsatz je Mitarbeiter" aus Werten verschiedener
  Jahre bilden.
- Beträge sind Euro. Darstellen in T€ (Tausend) mit Tausenderpunkt, über 100 Mio. € in Mio. € mit
  einer Nachkommastelle, Quoten in % mit einer Nachkommastelle (`equity_ratio` 0,35 = 35,0 %).
- Aufwände (Material, Personal, Abschreibungen) sind **negativ** geliefert und bleiben negativ.
- `financials.konzern` true: Konzernabschluss, immer so beschriften, nie neben Einzelabschlüssen
  ohne Hinweis.
- `filings[].fassung` `corrected` / `preliminary` nennen; `parsed_source` `pdf_text_llm_v1` als
  „aus dem PDF extrahiert" ausweisen.

## Vergleiche

- Vergleichbar sind nur gleiche Kennzahlen gleicher Definition: `umsatzerloese` gegen
  `umsatzerloese`, nie gegen `rohergebnis`; `operating_result` (Betriebserfolg) ist nicht
  `ebit_strict` (inkl. Finanzergebnis).
- `peer_percentiles` und `benchmark_company` ordnen **innerhalb der Größenklasse** ein, nicht
  innerhalb der Branche. Branchenvergleiche über `find_peers` oder `search_companies` mit
  `nace_section` / `oenace_division`.
- ÖNACE 2025 = WZ = NACE Rev. 2.1: derselbe Abschnittscode gilt in beiden Ländern.

## Budget

- Jede ausgelieferte Firma zählt einmal im Kalendermonat, erneutes Nachsehen derselben Firma ist
  frei. Deshalb `page_size` 3 beim Nachschlagen, 10 beim Vergleich, und vor großen Listen die
  Zahl nennen. `get_my_usage` zeigt den Stand und ist immer frei.
- Mit `search_companies` nicht „probieren": lieber einmal `list_sectors` und dann eine Suche.

## Personen

- Namen der Organe stammen aus dem öffentlichen Register und werden nur für die Einzelabfrage
  gezeigt; Geburtsdaten nur jahresgenau. Keine Listen von Personen über viele Firmen bauen, keine
  Verknüpfung mit anderen Quellen, keine Ansprache von Personen vorschlagen.
- Gleiche Namen sind nicht dieselbe Person (`search_person` sagt das selbst).

## Quelle und Gewähr

- Unter jede Tabelle und Grafik: Quelle (Firmenbuch, BMJ, CC BY 4.0 / Handelsregister der Länder)
  und `provenance.built_at` als Datenstand. Maßgeblich bleibt der amtliche Auszug
  (`get_document`).
