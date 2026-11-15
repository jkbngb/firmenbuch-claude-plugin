---
name: branchen-vergleich
description: Eine Firma mit Firmen derselben Branche (NACE/ÖNACE/WZ-Abschnitt oder enger) und Größenklasse vergleichen, als Tabelle mit Rang und Median. Verwenden, wenn jemand einen Branchenvergleich, Benchmark, Wettbewerber-Vergleich, „wie steht die Firma in ihrer Branche da" oder Perzentile sehen will.
---

# Branchen-Vergleich

Beispiel: „Wie steht die Rauch Fruchtsäfte GmbH in ihrer Branche da?" · „Vergleiche die Firma mit
Herstellern im selben NACE-Abschnitt in Vorarlberg."

Eingabe: Firmenname oder Register-ID; optional Vergleichsebene (Abschnitt, Abteilung, Gruppe) und
Region.

## Vorgehen

1. Register-ID klären (`search_companies`, `page_size` 3).
2. `get_company_details` für die Firma: daraus Branche (`industry.oenace.section` / `.division` /
   `.group`, Deutschland `industry.section`), Größenklasse (`size.gkl`), Bundesland, Land, und die
   eigenen Werte (`financials.latest`, `ratios.*.latest`).
3. Vergleichsgruppe laden, **eine** der beiden Arten, nie beide ungefragt:
   - **Server-Peers** (Standard): `find_peers` mit der ID, `n` 10. Österreich liefert die
     größennächsten Firmen derselben Branche so eng wie möglich (Kaskade Klasse → Gruppe →
     Abteilung → Abschnitt), `same_sector_level` nennt die erreichte Ebene, `note` sagt, ob
     Firmen anderer Branchen auffüllen. Deutschland liefert die nach Tätigkeitstext ähnlichsten
     Firmen. `cross_border=true` nur auf Wunsch (Bedeutungs-, kein Zahlenvergleich). Die
     Peer-Zeilen stehen in `peers_home`; ist die Liste leer, vor „keine Peers" erst
     `home_envelope.result.peers` und `note` lesen (dort steht die Liste des Registers samt
     Erweiterungen des Aktualitätsfensters).
   - **Eigene Gruppe** (wenn Ebene oder Region vorgegeben sind): `search_companies` mit
     `nace_section` (oder `oenace_division` / `oenace_group` für enger), `size_gkl` der Firma,
     `status` `active`, `has_guv_latest` true wenn Margen verglichen werden, optional
     `bundesland`, `sort` `{field: "bilanzsumme", descending: true}`, `page_size` 10,
     `country` wie die Firma. Für Deutschland zuerst `get_coverage`: Bilanzfilter greifen nur in
     Bundesländern mit erfassten Abschlüssen.
4. Nur Österreich, nur wenn Perzentile gefragt sind: `benchmark_company` (Pro). `percentiles` sind
   Ränge **innerhalb der eigenen Größenklasse**, nicht innerhalb der Branche; `percentiles_note`
   lesen und so beschriften.

## Ausgabe

Kopf: Firma, Branche mit Code und Klartext (z. B. „C 10.3 · Obst- und Gemüseverarbeitung"),
Vergleichsebene und Größenklasse, Zahl der Vergleichsfirmen, Region, Land.

Eine Tabelle, **die Firma selbst in der ersten Zeile fett**, dann die Vergleichsfirmen nach
Bilanzsumme absteigend:

| Firma | Sitz | Jahr | Bilanzsumme T€ | EK-Quote % | Umsatz T€ | Beschäftigte (Jahr) |
|---|---|---:|---:|---:|---:|---|

Die Spalten kommen aus den Trefferkarten: `bilanzsumme_latest`, `equity_ratio_latest`,
`revenue_latest` mit `revenue_year`, `employees_latest` mit `employees_year`, Jahr =
`last_filing_year` bzw. `latest_year` der Peer-Zeile. Eine Spalte **EBIT-Marge %** nur, wenn die
Zeilen sie tragen (`benchmark_company`-Peers) oder ausdrücklich gewünscht; dann je Firma
`get_company_details` (`ratios.ebit_margin.latest`), was weitere Aufrufe kostet, und vorher sagen.

Darunter eine Zeile **Median der Vergleichsfirmen** je Spalte (ohne die Firma selbst) und eine Zeile
**Rang der Firma** („3 von 11 nach Bilanzsumme, 7 von 11 nach EK-Quote"). Dann drei Sätze
Einordnung: Größe, Kapitalausstattung, Ertragskraft relativ zur Gruppe. Bei Perzentilen aus
`benchmark_company` ein eigener Absatz „Innerhalb der Größenklasse <gkl> (alle Branchen):
Bilanzsumme P<xx>, …". Die Perzentile gelten über alle Branchen derselben Größenklasse; die
Branche der Firma (`section`) beschreibt sie nur, sie grenzt die Perzentile nicht ein.

Anbieten: engere Ebene (`oenace_group`), andere Region, Export (`export_companies_csv`, Pro).

## Regeln

- Hat die Firma selbst keine Zahlen (keine Einreichung, nur PDF, `reporting_status`), bleibt sie
  mit „–" in der ersten Zeile, die Gruppe wird trotzdem gezeigt, Rang „nicht ermittelbar", und ein
  Satz nennt nur den Registerbefund, keine Vermutung über Rechtsform oder Offenlegungspflicht.
- Jede Vergleichsfirma kostet eine Firma aus dem Monatsbudget. Vor einer Gruppe über 10 Firmen
  kurz sagen, wie viele es werden.
- Die Spalte „Jahr" ist Pflicht: Peers tragen `latest_year`, und Abschlussjahre weichen ab.
  Firmen, deren jüngster Abschluss mehr als drei Jahre älter ist als der der Firma, kennzeichnen.
- Umsatz nur vergleichen, wenn beide Werte `umsatzerloese` sind; ein `rohergebnis` ist kein
  Umsatz (Fußnote). Fehlende GuV → „–", nicht aus dem Median werfen, sondern Median nur über
  vorhandene Werte und die Anzahl nennen („Median über 7 von 10").
- Konzernabschlüsse (`financials.konzern`) in der Tabelle markieren; ein Konzern neben
  Einzelabschlüssen verzerrt den Median.
- Deutsche Vergleichsfirmen ohne Zahlen (`reporting_status` ≠ `available`): in der Tabelle lassen,
  Zahlen „–", Grund in der Fußnote; nie als 0 werten.
- Abschnittsebene (NACE Level 1) ist breit: dazu immer einen Satz, dass eine engere Ebene
  (Abteilung, Gruppe) aussagekräftiger ist, wenn genug Firmen vorhanden sind.
- Branchen immer mit `list_sectors` auflösen, Codes nie raten.
