---
name: firmen-screenen
description: Firmenliste nach Branche, Bundesland und Mindestgröße erstellen. Verwenden, wenn eine Screening-Liste von Firmen aus Österreich oder Deutschland gefragt ist (Branche, Region, Bilanzsumme, Umsatz, Beschäftigte).
---

# Firmen screenen

Beispiel: „Welche aktiven GmbHs in Oberösterreich haben eine Bilanzsumme über 5 Mio. €?"

Eingaben: Branche (Pflicht), Bundesland (leer für alle), Mindestgröße (z. B. Bilanzsumme über 5 Mio. € oder mehr als 50 Beschäftigte, leer für keine).

Vorgehen:
1. Zuerst `list_sectors` mit der Branche aufrufen und den passenden Branchencode übernehmen.
2. Dann `search_companies` mit diesem Code, dem Bundesland und der Mindestgröße als Filter (Bilanzsumme, Umsatz oder Beschäftigte, je nach Angabe), nur aktive Firmen, sortiert nach Bilanzsumme absteigend, 25 Treffer pro Seite.

Umsatz und Beschäftigte gibt es nur für Österreich; in Deutschland greifen Bilanzsumme und Eigenkapitalquote nur dort, wo Jahresabschlüsse erfasst sind (`get_coverage` nennt den Stand). Eine fehlende deutsche Bilanz heißt nicht erfasst, nie null.

Ergebnis als Tabelle: Name, Sitz, Rechtsform, Bilanzsumme, Eigenkapitalquote, ID (AT:<Firmenbuchnummer> oder DE:<Registerkennung>). Danach die Gesamtzahl nennen und anbieten, die Liste mit `export_companies_csv` zu exportieren oder einzelne Firmen mit `get_company_details` zu vertiefen.
