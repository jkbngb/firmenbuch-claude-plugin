---
name: firmenkarte
description: Eine Firma als standardisierte Firmenkarte darstellen, eine kompakte Tabelle mit allen wichtigen Stammdaten (Rechtsform, Sitz, Register-ID, Branche, Geschäftsführung, Größe, letzter Abschluss). Verwenden, wenn jemand eine Firma nachschlagen, einen Steckbrief, ein Kurzprofil, eine Visitenkarte oder "die wichtigsten Daten auf einen Blick" möchte, aus Österreich oder Deutschland.
---

# Firmenkarte

Beispiel: „Zeig mir die Firmenkarte der Red Bull GmbH." · „Kurzprofil der Siemens AG, München."

Eingabe: Firmenname, Firmenbuchnummer oder Register-ID (`AT:<fnr>` / `DE:<Gerichtscode>_<Art><Nummer>`,
z. B. `DE:D2601_HRB6684`). Der deutsche Gerichtscode (`D2601`) ist nicht der Gerichtsname: aus
„Amtsgericht München HRB 6684" nie eine ID bauen, sondern mit `filters.name` plus
`filters.registergericht` „München" suchen und die `company_id` aus der Trefferkarte übernehmen.

## Vorgehen

1. `search_companies` mit `filters.name`, `page_size` 3 (jede ausgelieferte Firma zählt auf das
   Monatsbudget). Bei mehreren plausiblen Treffern den wahrscheinlichsten nehmen und die anderen
   in einer Zeile nennen; eine Register-ID direkt verwenden.
2. `get_company_details` für genau diese ID, bei einer deutschen Firma mit `max_signatories` 3 (die
   Karte zeigt nur die erste Person; ohne Kappung kommen bis zu 15 Organe und 50 KB zurück). Nichts
   weiter laden; Historie und Dokumente nur anbieten.

## Ausgabe: immer dieselbe Tabelle, immer dieselbe Zeilenfolge

Zwei Spalten (Feld, Wert), Markdown. Felder, die der Datensatz nicht trägt, bleiben in der Tabelle
und bekommen „–" plus den Grund in zwei Worten (siehe Regeln). Nie eine Zeile weglassen, nie eine
Zeile erfinden.

| Feld | Woher (AT) | Woher (DE) |
|---|---|---|
| **Firma** | `identity.name` | `identity.name` |
| **Rechtsform** | `identity.legal_form` als Klarname (GES → GmbH, AG, KG, eGen, Privatstiftung …) | `identity.legal_form` |
| **Status** | `identity.status` + `status_detail` („aktiv", „aktiv, in Liquidation", „gelöscht") | `identity.status` |
| **Register-ID** | `AT:<fnr>` (Firmenbuchnummer) | `DE:<id>` mit Gericht und Nummer (z. B. „Amtsgericht München HRB 6684") |
| **Sitz** | `location.street`, `postal_code city`, Bundesland (Code → Klarname: W Wien, O Oberösterreich, N Niederösterreich, St Steiermark, T Tirol, S Salzburg, K Kärnten, V Vorarlberg, B Burgenland) | `location.*`, Bundesland |
| **Gründung** | `company.founded_year` (nur ab ~1995 erfasst) | `company.founded_year` (`founded_source` nennt die Quelle) |
| **Stammkapital** | `company.stammkapital` | `company.stammkapital` |
| **Branche** | `industry.oenace.section` + `section_label_de`, dann `division` + `division_label_de` (ÖNACE 2025 = NACE Rev. 2.1) | `industry.section` + `section_label` (WZ = NACE); `division` oft leer |
| **Tätigkeit** | `industry.geschaeftszweig` (Freitext des Registers) | `gegenstand` (Unternehmensgegenstand), auf zwei Zeilen gekürzt |
| **Geschäftsführung** | `management.primary_manager` (Name, Funktion, Vertretung) und `n_signatories_latest` als „+ n weitere" (keine weiteren Namen) | `management.primary_manager_name` und `n_signatories_latest` als „+ n weitere" (keine weiteren Namen) |
| **Größenklasse** | `size.gkl` (W Kleinst, K Klein, M Mittel, G Groß nach UGB) | `size.gkl` (kleinst/klein/mittel/groß nach HGB), nur mit Abschluss |
| **Letzter Abschluss** | `financials.latest_year`, `filings[0].stichtag` | `financials.latest_year` oder `financials.reporting_status` |
| **Bilanzsumme** | `financials.latest.bilanzsumme` | `financials.latest.bilanzsumme` |
| **Eigenkapitalquote** | `ratios.equity_ratio.latest` in % | `ratios.equity_ratio.latest` in % |
| **Umsatz** | `financials.latest.umsatzerloese` oder `rohergebnis` (dann „Rohergebnis" schreiben) mit `revenue_year` | nur wenn in `financials.guv` vorhanden, sonst „– nicht veröffentlicht" |
| **Beschäftigte** | `employees.latest` **mit** `employees.latest_year` („120 (2022)") | meist nicht erfasst |
| **Letzte Änderung** | neuestes `events[]`-Element (Datum, Typ) | `company.letzte_eintragung` |
| **Datenstand** | `provenance.built_at` als Datum, Quelle „Firmenbuch (BMJ)" | `provenance.built_at`, „Handelsregister" |

Darunter genau drei Sätze, **jeder zu einer Zeile der Karte**: was die Firma tut (Zeile Tätigkeit),
wie groß sie ist (Größenklasse, Bilanzsumme, Umsatz), was auffällt (Liquidation, alte
Mitarbeiterzahl, nur Bilanz ohne GuV, Konzernabschluss, niedrige Branchen-Konfidenz). Was nicht in
der Karte steht, steht auch nicht im Text: keine Konzernzugehörigkeit, keine Marken, keine
Rangaussagen wie „eines der größten", keine Geschichte der Firma. Dann anbieten: Bilanz-Tabelle
(`bilanz-tabelle`), Kennzahlen-Grafik (`kennzahlen-grafik`), Branchenvergleich (`branchen-vergleich`).

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

## Weitere Regeln

- Beträge in **Tausend Euro mit Jahr**: „12.345 T€ (2024)". Über 100 Mio. € in Mio. € mit einer
  Nachkommastelle. Quoten in Prozent mit einer Nachkommastelle.
- Eine Zahl nie ohne ihr Jahr, wenn das Jahr vom letzten Abschluss abweicht (Umsatz, Beschäftigte).
- Fehlende deutsche Finanzzahlen: `reporting_status` → `available` (Zahlen da), `filed_unparsed`
  („– nur als PDF veröffentlicht"), `none_published` („– kein Abschluss veröffentlicht"), leer
  („– noch nicht erfasst"). **Nie „0", nie „keine Umsätze".**
- Fehlende österreichische GuV (`has_guv_latest` false): Umsatz „– verkürzte GuV / nur Bilanz".
- Bei `financials.konzern` true: Zeile „Bilanzsumme" mit Zusatz „(Konzern)".
- Branche mit niedriger Konfidenz (`industry.confidence` low, aus dem Unternehmensgegenstand
  klassifiziert): in der Zeile „(aus dem Tätigkeitstext abgeleitet, unsicher)" anfügen.
- Personen: nur Name, Funktion, Geburtsjahr-genaues Alter, wie geliefert. Keine Suche nach weiteren
  Personendaten, keine Zusammenführung über andere Quellen.
