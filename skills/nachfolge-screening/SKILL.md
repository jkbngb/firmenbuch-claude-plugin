---
name: nachfolge-screening
description: Firmen mit Nachfolgesignalen aus dem Register finden (alternde Geschäftsführung, keine jüngere namensgleiche Person im Organ, Führungswechsel, Umgründung) in einer Region und Größenklasse, als Liste mit Kennzeichen statt Ausschlüssen. Verwenden, wenn jemand Unternehmensnachfolge, Nachfolger-Suche, Übernahmekandidaten, Buy-and-Build-Ziele, Deal-Sourcing, „Firmen, deren Inhaber in Pension gehen" oder eine Zielfirmenliste für M&A sucht.
---

# Nachfolge-Screening

Beispiel: „Finde Firmen in Oberösterreich mit 15 bis 80 Mio. € Umsatz, deren Geschäftsführung über
60 ist und die keinen Nachfolger im Organ haben." · „Welche Maschinenbauer in Tirol hatten in den
letzten drei Jahren einen Führungswechsel?"

Eingabe: Region (Bundesland oder Land), Größenband (Umsatz, Bilanzsumme oder Beschäftigte), optional
Branche, Mindestalter der Führung, Zeitfenster für Ereignisse. Ergebnis: eine Liste mit festen
Kennzeichen je Firma, keine Vorauswahl, keine Bewertung von Personen.

## Was das Register hergibt, und was nicht

- **Österreich**: Organe mit Geburtsjahr, daraus der Nachfolge-Screen je Firma (`officer_screen`:
  Zahl der aktiven Organe, Geburtsjahr des ältesten führenden Organs, Altersabstand zur jüngsten
  namensgleichen Person) und das Kennzeichen `family_successor` auf jeder Trefferkarte: `true`
  (jüngere namensgleiche Person im Organ), `false` (keine), `null` (nicht prüfbar, weil nur ein
  Organ oder kein Geburtsjahr). Dazu Umsatz, EBITDA, Wachstum, Ereignisse (Führungswechsel,
  Kapital, Umgründung).
- **Deutschland**: Organe mit Geburtsjahr in der Einzelabfrage, Ereignisse (Geschäftsführerwechsel,
  Kapital, Verschmelzung), aber kein Nachfolge-Screen als Filter und Finanzzahlen nur in
  Bundesländern mit erfassten Abschlüssen (`get_coverage`, Bayern ja, Baden-Württemberg im Aufbau).
- **Nicht im Register**: Gesellschafter und Eigentumsanteile (nicht in den Registerdaten des
  Servers, weder AT noch DE), Insolvenzverfahren (kein Registermerkmal; nur Liquidation und
  Löschung sind erkennbar), Börsennotierung, Kontaktdaten. Das wird gesagt, nicht geschätzt.

## Vorgehen

1. Kriterien in einem Satz zurückspiegeln; Branche über `list_sectors` auflösen.
2. **Umfang messen**: `search_companies`, `country` wie gewünscht, `exact_count` true,
   `page_size` 1, Filter: `bundesland`, `status` `active`, `revenue_min`/`revenue_max` (oder
   `bilanzsumme_min`/`_max`, `employees_min`/`_max`), optional `nace_section`/`nace_division`,
   `has_guv_latest` true wenn Umsatz oder EBITDA gebraucht werden. Trefferzahl nennen; jede Firma
   zählt einmal im Monat auf das Budget.
3. **Nachfolgesignal eingrenzen (Österreich)**: dieselbe Suche mit `gf_age_min` (Alter des
   erstgereihten Geschäftsführers, z. B. 60). Nur eine Führungsperson wird dabei geprüft; das
   Kennzeichen `family_successor` steht auf jeder Trefferkarte und wird in der Liste gezeigt, es
   ist kein Suchfilter. Für Führungswechsel oder Umgründungen im Zeitfenster: `event_signal`
   `["leadership_change"]` bzw. `["restructuring", "capital_raise"]` mit `event_since` (Pro).
4. **Blättern**: `sort` `{field: "revenue", descending: true}`, `page_size` 25 (Solo 100), `page`
   hochzählen bis zur gewünschten Zahl. Über 100 Firmen vorher in einem Satz bestätigen lassen.
5. **Vertiefen** nur für die Firmen, die jemand ansehen will: `get_company_details` liefert
   `management.officer_screen` (Zähler, ohne Namen), `events[]` der letzten 36 Monate und die
   Kennzahlenreihen. Nie alle Treffer vertiefen.
6. Liste, dann Kennzeichen, dann die Sätze unten. Export über `recherche-excel` anbieten (die
   Kennzeichen-Spalten kommen mit).

## Ausgabe: eine Liste, feste Spalten, Kennzeichen statt Ausschlüsse

| Firma | Sitz | Branche | Umsatz T€ (Jahr) | EBITDA T€ | Beschäftigte (Jahr) | Ältestes führendes Organ | Jüngere Namensgleiche | Signale |
|---|---|---|---:|---:|---|---|---|---|

- **Ältestes führendes Organ**: Geburtsjahr aus `officer_screen.oldest_leader_birth_year`
  („Jg. 1958"), ohne Namen. In der Suchliste nur, wenn `gf_age_min` gesetzt war (dann erfüllt jede
  Zeile die Altersgrenze); sonst aus der Vertiefung.
- **Jüngere Namensgleiche**: `family_successor` → „nein" (Kandidat), „ja", „nicht prüfbar" (nur ein
  Organ oder kein Geburtsjahr). „Nicht prüfbar" bleibt in der Liste; es heißt nie „keine Nachfolge".
- **Signale**: aus den Trefferkarten und Ereignissen, als Kürzel, jedes mit Datum wo vorhanden:
  `Alter` (führendes Organ über der Grenze), `KeinNamensgleicher`, `Führungswechsel <Datum>`,
  `Umgründung <Datum>` (Einbringung, Verschmelzung, Spaltung, Rechtsformwechsel),
  `Kapital <Datum>`, `Liquidation`, `Holding/Immobilien` (ÖNACE 64.2 oder 68 als Hauptbranche),
  `Konzern` (Konzernabschluss), `Wachstum <Profil>`. Die Zahl der Signale steht vorn
  („3: Alter, KeinNamensgleicher, Führungswechsel 2025-03-12").

Kennzeichen entscheiden nichts; die Liste wird nicht nach ihnen gekürzt, nur sortiert (Signale
absteigend, dann Umsatz). Darunter drei Sätze: wie viele Firmen das Größenband trifft, wie viele
davon das Nachfolgesignal tragen, wie viele „nicht prüfbar" sind. Dann anbieten: Firmenkarte,
Bilanz-Tabelle, Branchenvergleich für einzelne Firmen; Excel über `recherche-excel`.

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

- **Keine Personenliste.** Die Liste zeigt Zähler und Geburtsjahre ohne Namen. Namen der Organe
  erscheinen nur in der Einzelabfrage einer Firma (Firmenkarte); keine Tabelle „Person × Firma",
  keine Kontaktdaten, keine Verknüpfung mit anderen Quellen, keine Ansprache vorschlagen.
- **Kein Score, keine Wahrscheinlichkeit.** Signale werden gezählt und benannt, nicht gewichtet.
  Eine Bewertung („Nachfolgewahrscheinlichkeit 73 %") ist keine Registerinformation.
- Das Namensgleichheits-Signal vergleicht nur eingetragene Organe: Gesellschafter ohne
  Organfunktion fehlen, eine Tochter mit Ehenamen wird nicht erkannt, ein zufällig namensgleicher
  Fremder wird mitgezählt. Das steht einmal unter jeder Liste.
- Deutsche Firmen: Spalten „Ältestes führendes Organ" und „Jüngere Namensgleiche" nur aus der
  Vertiefung (Organe mit Geburtsjahr in `get_company_details`, `max_signatories` 10), sonst „–
  nicht gescreent".
- Umsatzband nur in Österreich als Filter; in Deutschland Bilanzsumme oder Stammkapital
  (`capital_min`/`_max`) als Ersatz nennen, nie einen Umsatz schätzen.
