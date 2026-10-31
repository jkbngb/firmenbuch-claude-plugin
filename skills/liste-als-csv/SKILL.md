---
name: liste-als-csv
description: Eine gefilterte Firmenliste prüfen und als CSV exportieren. Verwenden, wenn eine Firmenliste als Datei, Export, CSV oder Tabelle zum Download gewünscht ist.
---

# Liste als CSV

Beispiel: „Alle Steuerberatungskanzleien in Wien als CSV exportieren?"

Eingabe: Filter in Worten, z. B. aktive GmbHs in Tirol mit Bilanz.

Vorgehen:
1. Zuerst `search_companies` mit den Filtern und `exact_count=true`, nur eine Seite, um Trefferzahl und Filterwirkung zu prüfen (Branchencodes über `list_sectors` auflösen).
2. Stimmen Filter und Umfang, dann `export_companies_csv` mit denselben Filtern und einem passenden `max_rows` aufrufen.

Ergebnis: Trefferzahl, angewendete Filter, Download-Link mit Gültigkeit, Hinweis bei gekappter Liste.
