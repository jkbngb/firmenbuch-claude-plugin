---
name: peer-vergleich
description: Eine Firma mit gleich großen Firmen vergleichen und einordnen. Verwenden, wenn Vergleichsfirmen, Peers, Benchmarks oder Perzentile zu einer Firma gefragt sind.
---

# Peer-Vergleich

Beispiel: „Welche Firmen sind ähnlich groß wie die Rauch Fruchtsäfte GmbH?"

Eingabe: Firmenname.

Vorgehen:
1. Zuerst `search_companies` nach dem Namen und die ID übernehmen.
2. Dann `find_peers` mit dieser ID (10 Peers). Mit `cross_border=true`, wenn Vergleichsfirmen aus beiden Ländern gewünscht sind.
3. Danach `benchmark_company` für die Perzentile im Größenvergleich (für österreichische Firmen).

Ergebnis als Tabelle: Firma und Peers mit Bilanzsumme, Umsatz, Eigenkapitalquote, Beschäftigten; darunter die Perzentilränge der Firma und zwei Sätze Einordnung. Anbieten, die Peers mit `export_companies_csv` zu exportieren.
