---
name: kennzahlen-grafik
description: Die wichtigsten Finanzkennzahlen einer Firma über die Jahre als Grafik darstellen (Bilanzsumme, Eigenkapital, Umsatz, Ergebnis, Eigenkapitalquote, Margen), vom Server fertig gezeichnet, immer gleich gestaltet. Verwenden, wenn jemand ein Diagramm, einen Chart, eine Visualisierung, einen Verlauf oder eine Entwicklung „als Grafik" sehen will.
---

# Kennzahlen-Grafik

Beispiel: „Zeichne die Umsatz- und Ergebnisentwicklung der Red Bull GmbH." · „Bilanzsumme und
Eigenkapitalquote der letzten Jahre als Chart."

Eingabe: Firmenname oder Register-ID, optional die gewünschten Kennzahlen und die Zahl der Jahre.

Die Grafik zeichnet der Server: das Werkzeug `chart_company_history` liefert ein fertiges Bild
(PNG, Balken für Beträge, Linien für Quoten, Legende, Quelle und Datenstand in der Fußzeile) und
dazu die Zahlen als Tabelle. **Kein eigenes HTML, SVG, React oder Python schreiben, nichts am
Aussehen entscheiden.** Die Antwort steht in Sekunden und jede Grafik sieht gleich aus.

## Vorgehen

1. Register-ID klären (`search_companies`, `page_size` 3) oder die ID direkt nehmen.
2. `chart_company_history` für die ID aufrufen. Ohne `metrics` zeichnet es Bilanzsumme,
   Eigenkapital, Umsatzerlöse und Jahresüberschuss (Balken) sowie Eigenkapitalquote, EBIT-Marge
   und Umsatzrendite (Linien); `metrics` wählt andere Reihen (deutsche Namen wie
   `eigenkapitalquote` oder `umsatzrendite` sind erlaubt), `years` begrenzt auf die jüngsten
   Jahre (höchstens acht). Die Antwort hat zwei Teile: das Bild und einen Text mit derselben
   Tabelle Kennzahl × Jahr samt Quelle und Datenstand.
3. Das Bild zeigen und die Tabelle aus der Antwort übernehmen (Beträge in T€ oder Mio. €, Quoten
   in %, neuestes Jahr rechts, „–" für ein fehlendes Jahr). Nichts umrechnen, nichts ergänzen.
4. Darunter ein bis drei Sätze, jeder zu einer gezeichneten Reihe (Richtung, Größenordnung).
   Nichts, was nicht in den Zahlen steht.

Liefert das Werkzeug kein Bild, sondern nur einen Text, dann hat die Firma weniger als zwei
Geschäftsjahre mit Kennzahlen im Register: den Text wiedergeben (er nennt die vorhandenen Jahre)
und `list_documents` → `get_document` anbieten. Keine leere Achse, keine Grafik aus einem Wert.

## Wenn `chart_company_history` nicht in der Werkzeugliste steht

Nur dann (älterer Server, anderer Client) gilt der frühere Weg über die Vorlage:
`get_company_history` mit `metrics` `["bilanzsumme", "eigenkapital", "umsatzerloese",
"jahresueberschuss", "equity_ratio", "ebit_margin", "net_margin"]` aufrufen, zuerst die Zahlen als
Tabelle nennen, dann `vorlage.md` neben dieser Datei lesen (in Claude Code
`${CLAUDE_PLUGIN_ROOT}/skills/kennzahlen-grafik/vorlage.md`), das HTML aus dem Codeblock nehmen
und nur den Block zwischen `// === DATEN` und `// === Ende DATEN` füllen:

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

In claude.ai, Claude Desktop und Cowork wird die gefüllte Vorlage ein Artefakt, in Claude Code eine
Datei `kennzahlen-<register-id>.html` im Arbeitsverzeichnis (nichts veröffentlichen, nichts
installieren). Reihe `rohergebnis` statt `umsatzerloese`: Schlüssel „Rohergebnis" nennen.

## Wo die Grafik erscheint

- **claude.ai, Claude Desktop, Cowork, Claude Code:** das Bild aus der Antwort von
  `chart_company_history` erscheint direkt im Gespräch. Nichts veröffentlichen, nichts speichern,
  außer ausdrücklich gewünscht.
- **Reiner Text (kein Bild möglich):** die Tabelle aus der Antwort, je Reihe ein Balken aus
  `█`-Zeichen mit Skala.

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

- Eine Reihe, die das Werkzeug als „nicht gezeichnet" nennt (weniger als zwei Werte), als Zahl
  nennen, nicht nachzeichnen.
- Beschäftigte nie in dieselbe Grafik wie Umsatz (verschiedene Jahre).
- Zwei Firmen in einer Grafik nur auf Wunsch, nur bei gleicher Einheit, nie Konzern- gegen
  Einzelabschluss; dann je Firma ein eigener Aufruf.
