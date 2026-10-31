---
name: geschaeftsfuehrer-finden
description: Die Firmen finden, in denen eine Person eine Organfunktion hat. Verwenden, wenn nach den Firmen einer Person (Geschäftsführung, Vorstand, Aufsichtsrat, Gesellschafter) gefragt wird.
---

# Geschäftsführer finden

Beispiel: „Welche Firmen führt eine Person namens Maria Huber?"

Eingabe: Name der Person.

Vorgehen:
1. `search_person` mit dem Namen aufrufen.

Ergebnis gruppiert nach gefundenem Namen: Firma, Sitz, Funktion, Status. Hinweis, dass gleiche Namen nicht dieselbe Person bedeuten müssen. Anbieten, einzelne Firmen mit `get_company_details` zu vertiefen.
