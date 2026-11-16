---
name: recherche-excel
description: Viele Firmen nach Kriterien recherchieren (Branche, Region, Größe, Status, Land) und alle Stammdaten und Kennzahlen in eine Excel-Datei schreiben (oder CSV, wo kein Excel möglich ist). Verwenden, wenn jemand eine Recherche, eine Marktübersicht, eine Zielfirmenliste, „alle Firmen, die …" oder eine Tabelle zum Weiterarbeiten in Excel möchte.
---

# Recherche als Excel

Beispiel: „Alle aktiven Getränkehersteller in Österreich mit Bilanzsumme über 10 Mio. € als Excel
mit allen Kennzahlen." · „Marktübersicht Steuerberatung in Wien, als Tabelle zum Weiterarbeiten."

Eingabe: Kriterien in Worten. Ergebnis: eine Datei `firmen-recherche-<JJJJ-MM-TT>.xlsx` mit zwei
Blättern, dazu drei Sätze im Gespräch.

## Vorgehen

1. **Kriterien in einem Satz zurückspiegeln**: Land, Branche (Code über `list_sectors` auflösen,
   nie raten), Region, Status (Standard: nur aktive), Mindestgröße, „nur mit Abschluss". Dann
   weitermachen, nicht auf Bestätigung warten, außer ein Kriterium ist unklar.
2. **Umfang messen**: `search_companies` mit den Filtern, `exact_count` true, `page_size` 1. Die
   Trefferzahl nennen und sagen, dass jede Firma einmal im Monat auf das Budget zählt (Gratis 10,
   Solo 500, Pro ohne Limit; `get_my_usage` zeigt den Stand). Über 50 Firmen: in einem Satz fragen,
   ob alle oder die größten N.
3. **Daten holen**, eine der beiden Arten:
   - Pro-Plan: `export_companies_csv` mit denselben Filtern und `max_rows` (ein Aufruf, Download-Link),
     die CSV laden und daraus die Excel-Datei bauen.
   - Sonst: `search_companies` blättern, `page_size` = Maximum des Plans (10 Gratis, 100 Solo),
     `page` 1, 2, 3 … bis alle Treffer da sind, `sort` `{field: "bilanzsumme", descending: true}`,
     `country` wie gewünscht. Die Trefferkarten tragen alle Spalten unten; `get_company_details` ist
     dafür nicht nötig.
   - Deutschland: zuerst `get_coverage`. Bilanzfilter greifen nur in Bundesländern mit erfassten
     Abschlüssen; sonst sind die Finanzspalten „–".
4. **Excel schreiben**: in Claude Code mit `python3` und `openpyxl` (fehlt es: CSV mit Semikolon,
   UTF-8 mit BOM, gleiche Spalten); in claude.ai, Claude Desktop und Cowork die Datei mit den
   Dateiwerkzeugen der Umgebung erzeugen und zum Download geben. Nichts installieren.
5. **Antwort im Gespräch**: Trefferzahl und Filter, Pfad oder Download der Datei, drei Auffälligkeiten
   aus den Spalten (größte Firma, Spanne der Eigenkapitalquote, Anteil ohne Abschluss). Nichts, was
   nicht in der Datei steht.

## Blatt „Firmen", eine Zeile je Firma, feste Spaltenfolge

| Spalte | Woher (Trefferkarte) | Format |
|---|---|---|
| Register-ID | `company_id` | Text |
| Firma | `name` | Text |
| Land | `country` | AT / DE |
| Rechtsform | `legal_form` | Text |
| Status | `status`, `status_detail` | „aktiv", „aktiv, in Liquidation", „gelöscht" |
| PLZ | `postal_code` | Text |
| Ort | `city` | Text |
| Bundesland | `bundesland` | Klarname |
| Branche Code | `industry_section` + `oenace_division` (AT) / `industry_section` (DE) | „C 11" |
| Branche | `oenace_division_label` oder Abschnittsname | Text |
| Tätigkeit | `geschaeftszweig` (AT) | Text, DE leer |
| Größenklasse | `size_gkl` | W / K / M / G |
| Abschlussjahr | `latest_year` | Zahl |
| Bilanzsumme T€ | `bilanzsumme_latest` / 1000 | Zahl, Tausenderpunkt |
| Anlagevermögen T€ | `anlagevermoegen_latest` / 1000 | Zahl |
| Eigenkapitalquote % | `equity_ratio_latest` × 100 | Zahl, eine Nachkommastelle |
| Umsatz T€ | `revenue_latest` / 1000 | Zahl |
| Umsatzbasis | `revenue_basis` | „Umsatzerlöse" / „Rohergebnis" |
| Umsatzjahr | `revenue_year` | Zahl |
| Umsatzwachstum 1 J. % | `revenue_growth_1y` × 100 | Zahl |
| Umsatzwachstum 3 J. % | `revenue_growth_3y` × 100 | Zahl |
| GuV vorhanden | `has_guv_latest` | ja / nein |
| Nur PDF | `has_pdf_only` | ja / nein |
| Gründung | `founded_year` | Zahl |

Fehlende Werte: Zelle leer, nie 0. Kopfzeile fett und eingefroren, Autofilter, Spaltenbreite nach
Inhalt, Zahlen als Zahlen (kein Text). **Keine Personenspalte** (Geschäftsführung, Organe), auch wenn
die Karte `manager_name` trägt: Personenlisten über viele Firmen sind nicht zulässig.

## Blatt „Abfrage"

Zeilen: Datum, Land, Filter (jeder einzeln), Sortierung, Trefferzahl gesamt, Zeilen in der Datei,
Plan und Budgetstand (`get_my_usage`), Quelle („Firmenbuch (BMJ), CC BY 4.0" / „Handelsregister
der Länder"), Datenstand (`provenance.built_at`), Hinweis „Maßgeblich bleibt der amtliche Auszug".

Auf Wunsch ein drittes Blatt „Details" mit den Mehrjahreswerten aus `get_company_details` für
höchstens 25 Firmen (dieselben Positionen wie `bilanz-tabelle`, ein Block je Firma): kostet keine
neuen Firmen, aber einen Aufruf je Firma; vorher sagen.

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

- Die Datei und das Gespräch zeigen dieselben Zahlen aus derselben Abfrage; nichts nachträglich
  aus anderen Quellen ergänzen.
- Umsatz nur in eine Spalte; die Spalte „Umsatzbasis" sagt, ob es Umsatzerlöse oder Rohergebnis
  sind. Nie beides mischen.
- Deutsche Firmen ohne erfasste Abschlüsse bleiben in der Liste; ihre Finanzspalten sind leer, das
  Blatt „Abfrage" nennt den Abdeckungsstand.
- Mehr als 1.000 Zeilen nur über `export_companies_csv` (Pro); sonst die größten 1.000 nach
  Bilanzsumme und das im Blatt „Abfrage" vermerken.
