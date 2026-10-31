---
name: firma-durchleuchten
description: Profil, Kennzahlen, Historie und Dokumente einer Firma zusammenfassen. Verwenden, wenn eine einzelne Firma aus dem Firmenbuch oder Handelsregister genau angesehen werden soll (Name oder Firmenbuchnummer).
---

# Firma durchleuchten

Beispiel: „Wer führt die Red Bull GmbH und wie hoch ist die Eigenkapitalquote?"

Eingabe: Firmenname oder Firmenbuchnummer.

Vorgehen:
1. Zuerst `search_companies` nach dem Namen (eine Firmenbuchnummer oder Registerkennung direkt als ID verwenden); bei mehreren Treffern kurz rückfragen, welche gemeint ist.
2. Dann `get_company_details` für die ID.
3. Anschließend `get_company_history` für Umsatz, Bilanzsumme und Eigenkapitalquote der letzten fünf Jahre.
4. Dann `list_documents` für die vorliegenden Urkunden und Jahresabschlüsse (bei einer deutschen Firma `get_document` für den amtlichen Registerauszug).
5. Dokumenttexte (`get_document_text`) nur auf Nachfrage laden.

Ergebnis: Stammdaten, Organe, Kennzahlen mit Entwicklung, letzte Registeränderungen, auffällige Punkte. Zum Schluss anbieten, die Firma mit `find_peers` zu vergleichen oder ein Dokument zu lesen.
