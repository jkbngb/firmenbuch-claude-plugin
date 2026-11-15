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

Dann zwei bis vier Sätze: Richtung der Bilanzsumme und des Eigenkapitals, Ergebnislage, ein
auffälliger Punkt. Danach anbieten: Grafik (`kennzahlen-grafik`), Vergleich (`branchen-vergleich`),
Originaldokument (`list_documents` → `get_document`).

## Regeln

- **Aufwände werden negativ geliefert** (Materialaufwand, Personalaufwand, Abschreibungen), seit
  2026-11-04 unabhängig vom Einreicher. So auch anzeigen, nicht umdrehen. `financials.anomalies`
  mit `expense_sign_normalised` ist Herkunftsangabe, kein Mangel.
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
