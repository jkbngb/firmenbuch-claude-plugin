---
name: datenregeln
description: Die Regeln für jede Antwort mit Zahlen aus dem Firmenbuch-MCP-Server (fehlende Werte, nur Registerdaten, Jahr und Einheit, Vorzeichen, Budget, Personen). Immer verwenden, wenn jemand nach Umsatz, Eigenkapital, Bilanzsumme, Beschäftigten, Kennzahlen oder Jahresabschlüssen einer Firma fragt, bevor Zahlen aus search_companies, get_company_details oder get_company_history in eine Antwort übernommen werden, und bei jeder Frage nach Verlässlichkeit, Aktualität oder Herkunft der Daten.
---

# Datenregeln

Die fünf Regeln unten gelten in jeder Antwort, Tabelle, Grafik und Datei. Sie stehen wortgleich in
`firmenkarte`, `bilanz-tabelle`, `kennzahlen-grafik`, `branchen-vergleich` und `recherche-excel`.
Begründung und Feldnamen: `BEST_PRACTICES.md` im Plugin.

## Sechs Regeln, die in jeder Antwort gelten

1. **Nur Registerdaten.** Jede Zahl und jeder Satz stammt aus der Antwort des Servers. Nichts aus
   Vorwissen, Websites oder Geschäftsberichten ergänzen, auch nicht als „bekanntlich".
2. **Fehlt ein Abschluss, nur der Befund.** Ein Satz: „Kein Jahresabschluss im Firmenbuch erfasst
   (Stand <Datum>)" oder „Abschlüsse nur als PDF hinterlegt". Keine Vermutung über Rechtsform, Größe,
   Konzern oder Offenlegungspflicht als Grund.
3. **Nie 0 für einen fehlenden Wert.** „–" mit dem Grund in zwei Worten.
4. **Jede Zahl mit Jahr und Einheit.** T€ mit Tausenderpunkt („12.345 T€ (2024)"), über 100 Mio. €
   in Mio. € mit einer Nachkommastelle, Quoten in % mit einer Nachkommastelle. **Filterwerte sind
   Euro, nicht T€** (15 Mio. € Umsatz = `revenue_min: 15000000`), Quoten als Bruch (35 % = `0.35`).
5. **Keine Feldnamen des Servers in der Antwort.** Nicht `has_pdf_only`, `series_sign_flip`,
   `reporting_status`, `filing_years_available`, sondern Worte: „nur als PDF hinterlegt",
   „Vorzeichen vereinheitlicht", „kein Abschluss veröffentlicht".
6. **Antwort im Gespräch.** Kein Artefakt, keine Veröffentlichung, keine Datei und kein Upload,
   außer der Skill sieht es ausdrücklich vor oder jemand bittet darum.

## Verboten und richtig

| Verboten | Richtig |
|---|---|
| „OGs mit natürlichen Personen als Gesellschafter sind nach UGB häufig von der Offenlegungspflicht befreit." | „Im Firmenbuch ist für diese Firma kein Jahresabschluss erfasst (Stand 06.11.2026)." |
| „Die Red Bull GmbH ist die Markenverwertungsgesellschaft des Red-Bull-Konzerns und eines der umsatzstärksten Unternehmen Österreichs." | „Umsatzerlöse 8.011,4 Mio. € (2025), Eigenkapitalquote 88,5 %." |
| „Umsatz: 0" · „keine Umsätze" | „Umsatz: – (verkürzte GuV, nur Bilanz)" |
| „Datensatz-Anomalie: `series_sign_flip`" | „Aufwände sind mit Minus ausgewiesen." |
| „Umsatz 12.345 T€" (ohne Jahr, obwohl 2023 und die Bilanz 2024) | „Umsatz 12.345 T€ (2023)" |

## Werkzeuge in der richtigen Reihenfolge

1. `search_companies` zuerst, mit engem Filter und `page_size` 3; eine Register-ID direkt.
2. `get_company_details` für **eine** Firma. Es enthält schon Bilanz-, GuV- und Kennzahlenreihen.
3. `get_company_history` nur für Reihen ohne das übrige Profil (Grafiken), `get_full_record` nur für
   alle UGB-Positionen, `list_documents` / `get_document` / `get_document_text` nur auf Nachfrage.
4. Branchen über `list_sectors`, Abdeckung über `get_coverage`, Felder über `describe_fields`.

## Was ein fehlender Wert bedeutet (und wie man es schreibt)

- Deutschland, `financials.reporting_status`: `available` → Zahlen zeigen; `filed_unparsed` → „nur
  als PDF veröffentlicht"; `none_published` → „kein Abschluss veröffentlicht"; leer → „noch nicht
  erfasst" (Aufbau Bundesland für Bundesland, `get_coverage`).
- Österreich: `has_guv_latest` false → „verkürzte GuV, nur Bilanz"; `employees.latest` null →
  „nicht erfasst" (rund 80 % der Firmen); `filing_years_available` 0 → „kein Jahresabschluss im
  Firmenbuch erfasst"; `has_pdf_only` true → „Abschlüsse nur als PDF hinterlegt, Originale über
  `get_document`".
- Ein Feld, das fehlt, bleibt in der Tabelle als „–" mit Grund; es wird nicht gestrichen und nicht
  geschätzt.

## Vorzeichen, Konzern, Fassung

- Aufwände (Material, Personal, Abschreibungen) sind negativ geliefert und bleiben negativ.
- `financials.konzern` true: „Konzernabschluss" in jeden Kopf, nie neben Einzelabschlüssen ohne
  Hinweis.
- `filings[].fassung` `corrected` / `preliminary`: „berichtigte Fassung" / „vorläufig" nennen;
  `parsed_source` `pdf_text_llm_v1`: „aus dem amtlichen PDF extrahiert".
- Umsatz (`revenue_year`) und Beschäftigte (`employees_year`) stammen oft aus einem älteren Jahr als
  die Bilanz; nie „Umsatz je Mitarbeiter" aus verschiedenen Jahren bilden.

## Vergleiche

- Vergleichbar sind nur gleiche Kennzahlen gleicher Definition: `umsatzerloese` gegen
  `umsatzerloese`, nie gegen `rohergebnis`; Betriebserfolg (`operating_result`) ist nicht EBIT inkl.
  Finanzergebnis (`ebit_strict`).
- `peer_percentiles` und `benchmark_company` ordnen **innerhalb der Größenklasse über alle
  Branchen** ein. Branchenvergleiche laufen über `find_peers` oder `search_companies` mit
  `nace_section` / `oenace_division`.
- Bewertungen des Servers (`growth_profile`, Hinweise wie „überkapitalisiert") sind abgeleitete
  Einstufungen aus den Kennzahlen, nicht Aussagen des Registers: „nach den Kennzahlen", nie „das
  Register weist aus".
- ÖNACE 2025 = WZ = NACE Rev. 2.1: derselbe Abschnittscode gilt in beiden Ländern.

## Budget

- Jede ausgelieferte Firma zählt einmal im Kalendermonat; dieselbe Firma erneut ansehen ist frei.
  Deshalb `page_size` 3 beim Nachschlagen, 10 beim Vergleich, und vor großen Listen die Zahl nennen.
  `get_my_usage` zeigt den Stand und ist immer frei.
- Nicht „probieren": lieber einmal `list_sectors` und dann eine Suche.

## Personen

- Namen der Organe stammen aus dem öffentlichen Register und erscheinen nur in der Einzelabfrage;
  Geburtsdaten nur jahresgenau. Keine Listen von Personen über viele Firmen, keine Verknüpfung mit
  anderen Quellen, keine Ansprache von Personen vorschlagen.
- Gleiche Namen sind nicht dieselbe Person.

## Quelle und Gewähr

Unter jede Tabelle, Grafik und Datei: „Quelle: Firmenbuch (BMJ), CC BY 4.0" bzw. „Handelsregister
der Länder", Datenstand aus `provenance.built_at`. Maßgeblich bleibt der amtliche Auszug
(`get_document`).
