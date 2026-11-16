---
name: marktsegment-uebersicht
description: Einen Teilmarkt (Branche nach NACE/ÖNACE/WZ, Region, Größenband) aus dem Register vermessen: Zahl der Firmen, Verteilung nach Bundesland und Größenklasse, die größten Firmen und ihr Anteil, Wachstumsprofile. Verwenden, wenn jemand eine Marktübersicht, Marktgröße, Branchenstruktur, Fragmentierung, „wie viele Firmen gibt es in …", eine Marktlandkarte oder einen Vergleich mehrerer Teilsegmente möchte.
---

# Marktsegment-Übersicht

Beispiel: „Wie groß ist der Markt für Getränkehersteller in Österreich, nach Bundesland?" ·
„Vergleiche Maschinenbau, Elektrotechnik und Kunststoffverarbeitung in Bayern: Anzahl, Größte,
Wachstum."

Eingabe: ein oder mehrere Teilsegmente (Branche in Worten oder Code), Region, optional Größenband.
Ergebnis: je Segment ein fester Block aus Zählungen und einer Top-Liste; mehrere Segmente
zusätzlich als Vergleichstabelle.

## Was gezählt werden kann, und was nicht

- **Exakt zählbar** (`search_companies` mit `exact_count` true, `page_size` 1; jeder Zählaufruf
  liefert eine Firma und zählt eine Firma aufs Budget): Firmen je Branche × Region × Status ×
  Größenband (`bilanzsumme_min`/`_max`, Österreich auch `revenue_min`/`_max`,
  `employees_min`/`_max`), je Größenklasse (`size_gkl`), je Wachstumsprofil (`growth_profile`:
  `shrinking`, `stable`, `growing`, `fast_growing`), je Bundesland.
- **Nur über gelieferte Zeilen** (eine Seite, `page_size` 25, Solo 100): Summe, Median und
  Bandbreite von Umsatz, Bilanzsumme und Eigenkapitalquote, Anteil der zehn größten. Das wird so
  beschriftet: „über die 100 größten nach Umsatz", nie als Marktsumme.
- **Nicht im Register**: Marktvolumen jenseits der Registerfirmen, Konsolidierungsgrad,
  Transaktionen, Bewertungen, Eintrittsbarrieren, Eigentümerstruktur. Wird gesagt, nicht geschätzt.
- Deutschland: Finanzfilter und -spalten nur in Bundesländern mit erfassten Abschlüssen
  (`get_coverage` zuerst); Zählungen nach Branche und Region gelten überall. Umsatz und
  Beschäftigte gibt es als Filter nur in Österreich.

## Vorgehen je Segment

1. Branche auflösen: `list_sectors` mit dem Begriff, Code auf Abteilungs- oder Gruppenebene
   übernehmen (`nace_division` „11", `nace_group` „10.3"). Nie raten.
2. Zählungen, jede mit `exact_count` true und `page_size` 1, `status` `active`:
   gesamt; je Bundesland der Region (ein Aufruf je Bundesland); je Größenklasse `W`/`K`/`M`/`G`;
   im Größenband, wenn eines genannt ist; je `growth_profile` (vier Aufrufe, nur Österreich, nur
   Firmen mit mehrjähriger GuV tragen ein Profil).
3. Top-Liste: eine Suche mit `sort` `{field: "revenue", descending: true}` (Deutschland
   `bilanzsumme`), `has_guv_latest` true in Österreich, `page_size` 10 (oder 25 auf Wunsch).
4. Mehr als 25 Zählaufrufe je Segment vermeiden; bei vielen Segmenten die Bundesland-Aufteilung
   nur für das gewünschte Hauptsegment.

## Ausgabe je Segment

**Kopf**: Segment mit Code und Klartext, Region, Stichtag (`provenance.built_at`).

| Kennzahl | Wert |
|---|---:|
| Aktive Firmen gesamt | |
| davon im Größenband <Band> | |
| nach Bundesland | W 120 · NÖ 80 · … |
| nach Größenklasse | Kleinst / Klein / Mittel / Groß |
| mit Jahresabschluss (GuV) | |
| Wachstumsprofil (Firmen mit mehrjähriger GuV) | schrumpfend / stabil / wachsend / stark wachsend |

**Die zehn größten** (nach Umsatz, Österreich; nach Bilanzsumme, Deutschland):

| # | Firma | Sitz | Jahr | Umsatz T€ | Bilanzsumme T€ | EK-Quote % | Beschäftigte (Jahr) |
|---:|---|---|---:|---:|---:|---:|---|

Darunter: Summe, Median und Bandbreite der gezeigten Zeilen, „Anteil der zehn größten am Umsatz der
<n> gelieferten Firmen". Dann drei Sätze: Größe des Segments, Konzentration (gleichmäßig oder von
wenigen dominiert, nur aus den gezeigten Zeilen), regionale Schwerpunkte.

**Mehrere Segmente**: zusätzlich eine Vergleichstabelle Segment | Firmen | im Größenband | größte
Firma (Umsatz) | Median Umsatz Top 10 | Anteil wachsend. Keine Rangliste nach „Attraktivität";
die Reihenfolge ist die Zahl der Firmen im Größenband.

Anbieten: `nachfolge-screening` für ein Segment, `recherche-excel` für die vollständige Liste,
`branchen-vergleich` für eine einzelne Firma.

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

- Eine Zählung ist exakt, eine Summe gilt nur für die gelieferten Zeilen; beides wird so
  beschriftet. Nie von zehn Zeilen auf den Markt hochrechnen.
- Umsatz und Rohergebnis nie in einer Spalte mischen (`revenue_basis` prüfen; Rohergebnis als
  Fußnote markieren).
- Konzernabschlüsse in der Top-Liste markieren; sie verzerren Summe und Median.
- Firmen ohne Abschluss zählen bei „Aktive Firmen gesamt" mit und fehlen in der Top-Liste; der
  Unterschied wird genannt.
- Branchen immer mit `list_sectors` auflösen; ÖNACE 2025 = WZ = NACE Rev. 2.1, derselbe Code
  gilt in beiden Ländern.
