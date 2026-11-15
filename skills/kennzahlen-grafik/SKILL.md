---
name: kennzahlen-grafik
description: Die wichtigsten Finanzkennzahlen einer Firma über die Jahre als Grafik darstellen (Bilanzsumme, Eigenkapital, Umsatz, Ergebnis, Eigenkapitalquote, Margen). Verwenden, wenn jemand ein Diagramm, einen Chart, eine Visualisierung, einen Verlauf oder eine Entwicklung „als Grafik" sehen will.
---

# Kennzahlen-Grafik

Beispiel: „Zeichne die Umsatz- und Ergebnisentwicklung der Palfinger AG." · „Bilanzsumme und
Eigenkapitalquote der letzten fünf Jahre als Chart."

Eingabe: Firmenname oder Register-ID, optional die gewünschten Kennzahlen.

## Vorgehen

1. Register-ID klären (`search_companies`, `page_size` 3) oder die ID direkt nehmen.
2. `get_company_history` für die ID. Für Österreich `metrics` eng setzen, z. B.
   `["bilanzsumme", "eigenkapital", "umsatzerloese", "jahresueberschuss", "equity_ratio",
   "ebit_margin"]`; Deutschland liefert alle vorhandenen Reihen. Die Antwort ist je Kennzahl ein
   Objekt `{history: {Jahr: Wert}, latest, latest_year}`.
3. Nur Jahre zeichnen, die wirklich einen Wert tragen. Keine Lücke füllen, keine Trendlinie durch
   fehlende Jahre, keine Prognose.

## Zwei Standardgrafiken

**Grafik 1 · Größe und Ergebnis** (Balken, T€ oder Mio. €): Bilanzsumme, Eigenkapital, Umsatzerlöse
(oder Rohergebnis, dann so beschriften), Jahresüberschuss. Negative Ergebnisse nach unten.

**Grafik 2 · Quoten** (Linien, %): Eigenkapitalquote, EBIT-Marge, Umsatzrendite; nur die Reihen mit
mindestens zwei Werten. Ohne GuV nur die Eigenkapitalquote.

Immer: Jahre auf der x-Achse in Zeitfolge, Einheit in der Achsenbeschriftung, Titel = Firmenname +
Register-ID, Fußzeile „Quelle: Firmenbuch (BMJ) / Handelsregister · Datenstand <provenance.built_at>".
Konzernabschluss (`financials.konzern`) in den Titel. Maximal zwei Grafiken, maximal vier Reihen je
Grafik.

## Wie zeichnen, je nach Umgebung

- **claude.ai, Claude Desktop, Cowork:** ein Artefakt (HTML mit eingebettetem SVG oder React). Werte
  fest einbetten, keine externen Daten laden. Farben: eine ruhige Palette, Eigenkapital und
  Eigenkapitalquote in derselben Farbe.
- **Claude Code:** eine Datei `kennzahlen-<register-id>.html` mit eingebettetem SVG in das
  Arbeitsverzeichnis schreiben und den Pfad nennen. Ist Python mit matplotlib vorhanden, zusätzlich
  ein PNG. Keine Pakete installieren.
- **Reiner Text:** Tabelle mit Jahr, Wert und einem Balken aus `█`-Zeichen je Reihe, Skala
  angegeben.

Unter jede Grafik dieselbe Mehrjahrestabelle wie in `bilanz-tabelle` (Kurzform: nur die gezeichneten
Reihen), damit jede Zahl nachlesbar ist.

## Regeln

- Eine Kennzahl mit nur einem Wert wird nicht gezeichnet, sondern als Zahl genannt.
- Beschäftigte nie in dieselbe Grafik wie Umsatz (verschiedene Jahre, siehe `datenregeln`).
- Deutsche Firma ohne Zahlen (`reporting_status` nicht `available`): keine Grafik, ein Satz mit dem
  Grund. Nie eine leere Achse zeichnen.
- Zwei Firmen in einer Grafik nur bei gleicher Währung und gleicher Einheit; dann je Firma eine
  Farbe, nie Konzern- gegen Einzelabschluss.
