---
name: neue-firmen-beobachten
description: Neugründungen und Registeränderungen einer Branche in einer Region verfolgen. Verwenden, wenn neue Firmen, Gründungen, Geschäftsführerwechsel oder Kapitalerhöhungen in einem Zeitraum gefragt sind.
---

# Neue Firmen beobachten

Beispiel: „Welche Firmen in Salzburg hatten im letzten Monat einen Geschäftsführerwechsel?"

Eingaben: Branche (Pflicht), Region (Bundesland oder Ort, leer für alle).

Vorgehen:
1. Zuerst `list_sectors` für den Branchencode.
2. Dann `list_events` mit types `["founding", "new_registration"]`, dem Branchencode und der Region, Zeitraum die letzten 30 Tage.
3. Zusätzlich `get_event_stats` für denselben Zeitraum als Überblick.
4. Bei Bedarf dieselbe Abfrage mit `["management_change", "capital_change"]` wiederholen.

Ergebnis: Liste der neuen Firmen mit Name, Sitz, Datum, ID, danach die Statistik. Anbieten, die Abfrage mit festem Zeitfenster (since, until) regelmäßig zu wiederholen.
