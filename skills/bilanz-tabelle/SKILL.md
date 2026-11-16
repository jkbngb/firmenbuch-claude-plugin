---
name: bilanz-tabelle
description: Bilanz, Gewinn- und Verlustrechnung und Kennzahlen einer Firma als standardisierte Mehrjahrestabelle darstellen (bis zu fünf Jahre, feste Zeilen, T€). Verwenden, wenn jemand die Bilanz, den Jahresabschluss, die GuV, die Zahlen, die Finanzkennzahlen oder die Entwicklung einer Firma in Tabellenform sehen will.
---

# Bilanz-Tabelle

Beispiel: „Bilanz und GuV der Rauch Fruchtsäfte GmbH der letzten fünf Jahre als Tabelle."

Eingabe: Firmenname oder Register-ID.

## Vorgehen

1. Register-ID klären: `search_companies` mit `filters.name`, `page_size` 3 (oder die ID direkt).
2. `get_company_details` für die ID. Darin liegen bereits alle Zeitreihen:
   `financials.bilanz.<position>.history` (Jahr → Wert), `financials.guv.<position>.history`,
   `ratios.<kennzahl>.history`. Ein zweiter Aufruf ist nur nötig, wenn jemand **jede** UGB-Position
   will: dann `get_full_record` (Österreich, schwer, 300+ Positionen) und nur auf Nachfrage.
   Vorher prüfen: `company.filing_years_available` 0 oder `filings` leer heißt, das Register hält
   keinen maschinenlesbaren Abschluss. Dann **eine** Zeile mit dem Befund („kein Jahresabschluss im
   Firmenbuch erfasst" / `reporting_status`-Grund), höchstens eine weitere Suche nach einer nahe
   liegenden Schwester- oder Holdinggesellschaft, und keine Vermutung über Rechtsform oder
   Offenlegungspflicht. Trägt `financials.register_documents` nur PDFs (Konzernabschluss, Protokolle),
   so sagen und `get_document` anbieten.
3. Jahre = die Schlüssel der `history`-Objekte, maximal die fünf jüngsten, **neuestes Jahr links**.
   Ein Jahr ohne Abschluss bekommt keine Spalte und wird unter der Tabelle genannt („2021: kein
   Abschluss eingereicht"). Nie interpolieren.

## Ausgabe: drei Blöcke, feste Zeilen, feste Reihenfolge

Kopf: Firma, Register-ID, Rechtsform, Größenklasse, „Beträge in T€, Geschäftsjahr endet am
<Stichtag aus `filings[]`>", bei `financials.konzern` true „Konzernabschluss".

**Bilanz** (Positionen aus `financials.bilanz`):

| Position | 2024 | 2023 | 2022 | … |
|---|---:|---:|---:|---|
| Anlagevermögen (`anlagevermoegen`) | | | | |
| davon Sachanlagen (`sachanlagen`, nur AT) | | | | |
| davon Finanzanlagen (`finanzanlagen`, nur AT) | | | | |
| Umlaufvermögen (`umlaufvermoegen`) | | | | |
| davon Vorräte (`vorraete`, nur AT) | | | | |
| davon Forderungen (`forderungen`, nur AT) | | | | |
| davon liquide Mittel (`cash`, nur AT) | | | | |
| **Bilanzsumme** (`bilanzsumme`) | | | | |
| Eigenkapital (`eigenkapital`) | | | | |
| davon Stammkapital (`stammkapital`) | | | | |
| davon Bilanzgewinn/-verlust (`bilanzgewinn_verlust`, nur AT) | | | | |
| Rückstellungen (`rueckstellungen`) | | | | |
| Verbindlichkeiten (`verbindlichkeiten`) | | | | |

**GuV** (nur wenn `financials.has_guv` true; sonst eine Zeile: „GuV nicht veröffentlicht, verkürzte
Einreichung nach § 279 UGB / § 326 HGB"). Positionen aus `financials.guv`:

| Position | 2024 | 2023 | … |
|---|---:|---:|---|
| Umsatzerlöse (`umsatzerloese`) oder Rohergebnis (`rohergebnis`; Zeile dann so beschriften) | | | |
| Materialaufwand (`materialaufwand`) | | | |
| Personalaufwand (`personalaufwand`) | | | |
| Abschreibungen (`abschreibungen`) | | | |
| Betriebserfolg (`operating_result`) | | | |
| EBITDA (`ebitda`) | | | |
| EBIT inkl. Finanzergebnis (`ebit_strict`) | | | |
| **Jahresüberschuss / -fehlbetrag** (`jahresueberschuss`) | | | |

**Kennzahlen** (aus `ratios`, in %, eine Nachkommastelle):

| Kennzahl | 2024 | 2023 | … | Trend |
|---|---:|---:|---|---|
| Eigenkapitalquote (`equity_ratio`) | | | | `trend` |
| Anlagendeckung I (`anlagedeckungsgrad_1`) | | | | |
| EBIT-Marge (`ebit_margin`) | | | | |
| EBITDA-Marge (`ebitda_margin`) | | | | |
| Umsatzrendite (`net_margin`) | | | | |
| Gesamtkapitalrendite (`roa`) | | | | |
| Eigenkapitalrendite (`roe`) | | | | |
| Personalkostenquote (`personalkostenquote`) | | | | |

Dann zwei bis drei Sätze, **jeder zu einer Zeile der Tabelle**: Richtung der Bilanzsumme und des
Eigenkapitals, Ergebnislage, ein auffälliger Punkt. Keine Erklärung von Ursachen, die nicht in der
Tabelle stehen. Danach anbieten: Grafik (`kennzahlen-grafik`), Vergleich (`branchen-vergleich`),
Originaldokument (`list_documents` → `get_document`).

## Fünf Regeln, die in jeder Antwort gelten

1. **Nur Registerdaten.** Jede Zahl und jeder Satz stammt aus der Antwort des Servers. Nichts aus
   Vorwissen, Websites oder Geschäftsberichten ergänzen, auch nicht als „bekanntlich".
2. **Fehlt ein Abschluss, nur der Befund.** Ein Satz: „Kein Jahresabschluss im Firmenbuch erfasst
   (Stand <Datum>)" oder „Abschlüsse nur als PDF hinterlegt". Keine Vermutung über Rechtsform, Größe,
   Konzern oder Offenlegungspflicht als Grund.
3. **Nie 0 für einen fehlenden Wert.** „–" mit dem Grund in zwei Worten.
4. **Jede Zahl mit Jahr und Einheit.** T€ mit Tausenderpunkt („12.345 T€ (2024)"), über 100 Mio. €
   in Mio. € mit einer Nachkommastelle, Quoten in % mit einer Nachkommastelle.
5. **Keine Feldnamen des Servers in der Antwort.** Nicht `has_pdf_only`, `series_sign_flip`,
   `reporting_status`, `filing_years_available`, sondern Worte: „nur als PDF hinterlegt",
   „Vorzeichen vereinheitlicht", „kein Abschluss veröffentlicht".

## Weitere Regeln

- **Aufwände werden negativ geliefert** (Materialaufwand, Personalaufwand, Abschreibungen), seit
  2026-11-04 unabhängig vom Einreicher. So auch anzeigen, nicht umdrehen. `financials.anomalies`
  ist Herkunftsangabe für die Pipeline und wird **nie zitiert**, weder Name noch Wert; wenn ein
  Hinweis nötig ist, ein Satz in Worten („Aufwände sind mit Minus ausgewiesen").
- Eine Zeile, deren Position im Datensatz fehlt, bleibt mit „–" stehen; „davon"-Zeilen ohne jeden
  Wert dürfen entfallen. Deutsche Abschlüsse tragen meist nur die Hauptpositionen.
- Abgeleitete Größen (`ebitda`, `ebit_strict`) fehlen absichtlich, wenn `guv_konfidenz` `low` ist:
  dann „– nicht abstimmbar", nie selbst nachrechnen.
- `revenue_basis_change` gesetzt: Zeile „Umsatzerlöse" endet in dem Jahr, ab dann „Rohergebnis";
  einen Satz dazu schreiben. Umsatz und Rohergebnis nie in einer Zeile mischen.
- `filings[].fassung` `corrected` oder `preliminary`: in der Kopfzeile nennen („2023: berichtigte
  Fassung").
- `filings[].parsed_source` `pdf_text_llm_v1`: Fußnote „Werte aus dem amtlichen PDF extrahiert".
- Deutsche Firma ohne Zahlen: nur der Kopf und eine Zeile mit dem `reporting_status`-Grund; nie eine
  Tabelle voller Nullen.
- Tausenderpunkt, kein Komma bei T€-Beträgen („12.345"), negative Beträge mit Minus („-1.234").
- Die Spalte „Trend" nur, wenn mindestens zwei Jahre vorliegen; bei einem Jahr „–", auch wenn der
  Server einen `trend`-Wert liefert. Liegt nur ein Jahr maschinenlesbar vor, das im Kopf sagen und
  die PDF-Jahre aus `filings[]` / `register_documents` nennen.
