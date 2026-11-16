---
name: kennzahlen-grafik
description: Die wichtigsten Finanzkennzahlen einer Firma über die Jahre als Grafik darstellen (Bilanzsumme, Eigenkapital, Umsatz, Ergebnis, Eigenkapitalquote, Margen), aus einer fertigen Vorlage, immer gleich gestaltet. Verwenden, wenn jemand ein Diagramm, einen Chart, eine Visualisierung, einen Verlauf oder eine Entwicklung „als Grafik" sehen will.
---

# Kennzahlen-Grafik

Beispiel: „Zeichne die Umsatz- und Ergebnisentwicklung der Red Bull GmbH." · „Bilanzsumme und
Eigenkapitalquote der letzten Jahre als Chart."

Eingabe: Firmenname oder Register-ID, optional die gewünschten Kennzahlen.

Die Gestaltung ist entschieden: die Vorlage in `vorlage.md` neben dieser Datei (ein HTML-Dokument in einem
Codeblock) zeichnet zwei Grafiken
(Balken für Beträge, Linien für Quoten) samt Tabellen und Fußzeile aus **einem Datenblock**. Die
Aufgabe ist, die Zahlen zu holen und den Datenblock zu füllen. **Kein eigenes HTML, SVG, React oder
Python schreiben, nichts am Aussehen entscheiden.** Das hält die Antwort unter einer Minute und
macht jede Grafik gleich.

## Vorgehen

1. Register-ID klären (`search_companies`, `page_size` 3) oder die ID direkt nehmen.
2. `get_company_history` für die ID mit `metrics`
   `["bilanzsumme", "eigenkapital", "umsatzerloese", "jahresueberschuss", "equity_ratio",
   "ebit_margin", "net_margin"]` (Deutschland liefert alle vorhandenen Reihen). Die Antwort ist je
   Kennzahl ein Objekt `{history: {Jahr: Wert}, latest, latest_year}`.
3. **Zuerst die Zahlen als Text**: eine Tabelle Kennzahl × Jahr (Beträge in T€ oder Mio. €, Quoten
   in %), neuestes Jahr rechts, Quelle und Datenstand darunter. Damit hat die Person die Werte, auch
   wenn sie die Grafik nicht öffnet.
4. **Dann die Grafik aus der Vorlage**: `vorlage.md` lesen (im Ordner dieses Skills, in Claude
   Code `${CLAUDE_PLUGIN_ROOT}/skills/kennzahlen-grafik/vorlage.md`), das HTML aus dem Codeblock
   nehmen, nur den Block zwischen
   `// === DATEN` und `// === Ende DATEN` ersetzen, sonst nichts ändern.

```js
const DATA = {
  firma: "<identity.name>", id: "<AT:fnr oder DE:id>", konzern: <financials.konzern, true/false>,
  datenstand: "<provenance.built_at, Datum>", quelle: "Firmenbuch (BMJ) · CC BY 4.0",   // DE: "Handelsregister der Länder"
  jahre: [2022, 2023, 2024, 2025],                     // alle Jahre, die in irgendeiner Reihe vorkommen, aufsteigend, höchstens die acht jüngsten
  betraege: {                                          // Euro, nicht T€; fehlendes Jahr = null
    "Bilanzsumme": [...], "Eigenkapital": [...], "Umsatzerlöse": [...], "Jahresüberschuss": [...]
  },
  quoten: {                                            // Bruch wie geliefert (0,35 = 35 %); fehlendes Jahr = null
    "Eigenkapitalquote": [...], "EBIT-Marge": [...], "Umsatzrendite": [...]
  }
};
```

   Reihe `rohergebnis` statt `umsatzerloese`: Schlüssel „Rohergebnis" nennen. Reihen mit weniger als
   zwei Werten lässt die Vorlage selbst weg und nennt sie unter der Grafik; sie müssen nicht
   entfernt werden. Einheit (T€ oder Mio. €), Achsen, Farben und Tabellen setzt die Vorlage.

## Wo die Grafik erscheint

- **claude.ai, Claude Desktop, Cowork:** ein Artefakt mit dem vollständigen Inhalt der Vorlage (nur der
  Datenblock gefüllt). Es rendert sofort im Gespräch.
- **Claude Code:** die gefüllte Vorlage als Datei `kennzahlen-<register-id>.html` ins
  Arbeitsverzeichnis schreiben und den Pfad nennen. Nichts veröffentlichen (kein Artefakt, kein
  Upload), nichts installieren, kein PNG, außer ausdrücklich gewünscht.
- **Reiner Text:** nur die Tabelle aus Schritt 3, je Reihe ein Balken aus `█`-Zeichen mit Skala.

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

- Leere `metrics` (Österreich) oder `reporting_status` nicht `available` (Deutschland): keine
  Grafik, keine leere Achse. Ein Satz mit dem Befund aus Regel 2, dann `list_documents` →
  `get_document` anbieten.
- Eine Kennzahl mit nur einem Wert wird als Zahl genannt, nicht gezeichnet.
- Beschäftigte nie in dieselbe Grafik wie Umsatz (verschiedene Jahre).
- Zwei Firmen in einer Grafik nur auf Wunsch, nur bei gleicher Einheit, nie Konzern- gegen
  Einzelabschluss; dann je Firma eine eigene Vorlage.
- Unter die Grafik ein bis drei Sätze, jeder zu einer gezeichneten Reihe (Richtung, Größenordnung).
  Nichts, was nicht in den Zahlen steht.
