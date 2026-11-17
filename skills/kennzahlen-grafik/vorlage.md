# Vorlage der Kennzahlen-Grafik

Die Datei zwischen den Zäunen ist die vollständige Vorlage. Sie wird unverändert übernommen; nur der Block zwischen `// === DATEN` und `// === Ende DATEN` wird mit den Werten der Firma gefüllt. (Als Markdown abgelegt, weil das Claude-Verzeichnis in Plugins nur Markdown, Text, JSON, YAML und Bilder zulässt.)

```html
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kennzahlen</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: "Helvetica Neue", Helvetica, Arial, sans-serif; background: #F5F6F7; color: #17211C; padding: 24px 16px; }
  .card { background: #fff; border-radius: 12px; padding: 24px 28px; box-shadow: 0 2px 10px rgba(0,0,0,.08); max-width: 820px; margin: 0 auto 20px; }
  h1 { font-size: 1.05rem; font-weight: 700; }
  h2 { font-size: .9rem; font-weight: 600; margin: 4px 0 10px; color: #333; }
  .meta { font-size: .75rem; color: #777; margin: 3px 0 16px; }
  svg { display: block; width: 100%; height: auto; }
  .legend { display: flex; gap: 16px; margin-top: 10px; flex-wrap: wrap; }
  .li { display: flex; align-items: center; gap: 6px; font-size: .72rem; color: #555; }
  .sw { width: 12px; height: 12px; border-radius: 3px; }
  table { width: 100%; border-collapse: collapse; margin-top: 16px; font-size: .74rem; }
  th { background: #f7f8fa; padding: 6px 10px; font-weight: 600; color: #555; text-align: right; border-bottom: 2px solid #e8e9ec; }
  th:first-child, td:first-child { text-align: left; }
  td { padding: 5px 10px; text-align: right; border-bottom: 1px solid #f0f1f3; }
  .note { font-size: .72rem; color: #777; margin-top: 10px; }
  .footer { font-size: .65rem; color: #999; margin-top: 12px; }
</style>
</head>
<body>
<script>
// === DATEN: nur diesen Block ersetzen. Beträge in Euro, Quoten als Bruch (0,35 = 35 %),
// fehlende Jahre als null. Reihen mit weniger als zwei Werten werden nicht gezeichnet.
const DATA = {
  firma: "Musterfirma GmbH",
  id: "AT:000000a",
  konzern: false,
  datenstand: "2026-01-01",
  quelle: "Firmenbuch (BMJ) · CC BY 4.0",
  jahre: [2021, 2022, 2023, 2024],
  betraege: {
    "Bilanzsumme":      [1000000, 1200000, 1300000, 1500000],
    "Eigenkapital":     [300000, 350000, 420000, 500000],
    "Umsatzerlöse":     [2000000, 2100000, null, 2500000],
    "Jahresüberschuss": [50000, -20000, 70000, 90000]
  },
  quoten: {
    "Eigenkapitalquote": [0.30, 0.29, 0.32, 0.33],
    "EBIT-Marge":        [0.04, -0.01, 0.05, 0.06],
    "Umsatzrendite":     [0.025, -0.01, 0.03, 0.036]
  }
};
// === Ende DATEN
</script>

<div class="card" id="k1">
  <h1 id="titel"></h1>
  <p class="meta" id="meta"></p>
  <h2 id="h1"></h2>
  <svg id="c1" viewBox="0 0 760 340"></svg>
  <div class="legend" id="l1"></div>
  <table id="t1"></table>
  <p class="note" id="n1"></p>
</div>

<div class="card" id="k2">
  <h2>Quoten in %</h2>
  <svg id="c2" viewBox="0 0 760 300"></svg>
  <div class="legend" id="l2"></div>
  <table id="t2"></table>
  <p class="note" id="n2"></p>
  <p class="footer" id="fuss"></p>
</div>

<script>
(function () {
  const NS = "http://www.w3.org/2000/svg";
  const COLORS = ["#0A6B3D", "#3FBF7F", "#0E8F52", "#9AA79D", "#17211C", "#5F6F65"];  // Agentic Firmenbuch: Grün-Rampe, Grau-Grün, Tinte
  const COLOR_OF = { "Eigenkapital": "#3FBF7F", "Eigenkapitalquote": "#3FBF7F" };
  const fmt = (v, d) => v == null ? "–" : v.toLocaleString("de-DE", { minimumFractionDigits: d, maximumFractionDigits: d });
  const el = (t, a, txt) => { const e = document.createElementNS(NS, t); for (const k in a) e.setAttribute(k, a[k]); if (txt != null) e.textContent = txt; return e; };
  const years = DATA.jahre.map(String);

  function usable(obj) {
    const out = [], skipped = [];
    Object.keys(obj || {}).forEach((k, i) => {
      const n = (obj[k] || []).filter(v => v != null).length;
      (n >= 2 ? out : skipped).push({ name: k, values: obj[k], color: COLOR_OF[k] || COLORS[i % COLORS.length] });
    });
    return { out: out.slice(0, 4), skipped };
  }
  function ticks(min, max) {
    const span = max - min || 1, raw = span / 4, p = Math.pow(10, Math.floor(Math.log10(raw)));
    const step = [1, 2, 2.5, 5, 10].map(m => m * p).find(s => span / s <= 5) || p * 10;
    const lo = Math.floor(min / step) * step, hi = Math.ceil(max / step) * step, t = [];
    for (let v = lo; v <= hi + 1e-9; v += step) t.push(+v.toFixed(10));
    return { lo, hi, t };
  }
  function legend(id, series) {
    const L = document.getElementById(id);
    series.forEach(s => { const d = document.createElement("div"); d.className = "li";
      d.innerHTML = '<span class="sw" style="background:' + s.color + '"></span>' + s.name; L.appendChild(d); });
  }
  function table(id, head, series, f) {
    const T = document.getElementById(id);
    T.innerHTML = "<thead><tr><th>" + head + "</th>" + years.map(y => "<th>" + y + "</th>").join("") + "</tr></thead><tbody>" +
      series.map(s => "<tr><td>" + s.name + "</td>" + s.values.map(v => "<td>" + f(v) + "</td>").join("") + "</tr>").join("") + "</tbody>";
  }
  function axes(svg, W, H, ml, mr, mt, mb, tk, unit) {
    const ph = H - mt - mb, pw = W - ml - mr, y = v => mt + ph - (v - tk.lo) / (tk.hi - tk.lo) * ph;
    tk.t.forEach(v => {
      svg.appendChild(el("line", { x1: ml, y1: y(v), x2: ml + pw, y2: y(v), stroke: v === 0 ? "#9AA79D" : "#E3EFE8", "stroke-width": v === 0 ? 1.2 : 1 }));
      svg.appendChild(el("text", { x: ml - 8, y: y(v) + 4, "text-anchor": "end", "font-size": 11, fill: "#888" }, fmt(v, 0)));
    });
    svg.appendChild(el("text", { x: 12, y: mt + ph / 2, "text-anchor": "middle", "font-size": 10, fill: "#999", transform: "rotate(-90,12," + (mt + ph / 2) + ")" }, unit));
    years.forEach((yr, i) => svg.appendChild(el("text", { x: ml + (i + .5) * pw / years.length, y: H - mb + 22, "text-anchor": "middle", "font-size": 12, "font-weight": 600, fill: "#444" }, yr)));
    return { y, pw, ph };
  }

  // Kopf
  document.getElementById("titel").textContent = DATA.firma + " · " + DATA.id + (DATA.konzern ? " · Konzernabschluss" : "");
  const b = usable(DATA.betraege);
  const maxAbs = Math.max(1, ...b.out.flatMap(s => s.values.filter(v => v != null).map(Math.abs)));
  const div = maxAbs >= 1e8 ? 1e6 : 1e3, unit = div === 1e6 ? "Mio. €" : "T€", dec = div === 1e6 ? 1 : 0;
  document.getElementById("meta").textContent = "Beträge in " + unit + " · Jahre mit Abschluss: " + years.join(", ");
  document.getElementById("h1").textContent = "Größe und Ergebnis in " + unit;

  // Grafik 1: Balken
  if (b.out.length) {
    const svg = document.getElementById("c1"), W = 760, H = 340, ml = 70, mr = 16, mt = 16, mb = 40;
    const vals = b.out.flatMap(s => s.values.filter(v => v != null).map(v => v / div));
    const tk = ticks(Math.min(0, ...vals), Math.max(0, ...vals));
    const A = axes(svg, W, H, ml, mr, mt, mb, tk, unit);
    const gw = A.pw / years.length, bw = Math.min(28, (gw - 16) / b.out.length), pad = (gw - bw * b.out.length) / 2;
    years.forEach((yr, gi) => b.out.forEach((s, si) => {
      const v = s.values[gi]; if (v == null) return;
      const x = ml + gi * gw + pad + si * bw, y0 = A.y(0), y1 = A.y(v / div);
      svg.appendChild(el("rect", { x, y: Math.min(y0, y1), width: bw - 2, height: Math.abs(y1 - y0), fill: s.color, rx: 2 }));
      svg.appendChild(el("text", { x: x + (bw - 2) / 2, y: v >= 0 ? y1 - 4 : y1 + 11, "text-anchor": "middle", "font-size": 8.5, fill: s.color, "font-weight": 700 }, fmt(v / div, dec)));
    }));
    legend("l1", b.out);
    table("t1", "Kennzahl (" + unit + ")", b.out, v => fmt(v == null ? null : v / div, dec));
  } else document.getElementById("c1").remove();
  document.getElementById("n1").textContent = b.skipped.length ? "Nicht gezeichnet (weniger als zwei Werte): " + b.skipped.map(s => s.name).join(", ") : "";

  // Grafik 2: Linien
  const q = usable(DATA.quoten);
  if (q.out.length) {
    const svg = document.getElementById("c2"), W = 760, H = 300, ml = 70, mr = 16, mt = 16, mb = 40;
    const vals = q.out.flatMap(s => s.values.filter(v => v != null).map(v => v * 100));
    const tk = ticks(Math.min(0, ...vals), Math.max(...vals));
    const A = axes(svg, W, H, ml, mr, mt, mb, tk, "%");
    const X = i => ml + (i + .5) * A.pw / years.length;
    q.out.forEach(s => {
      let d = "", pen = false;
      s.values.forEach((v, i) => { if (v == null) { pen = false; return; } d += (pen ? " L" : " M") + X(i) + " " + A.y(v * 100); pen = true; });
      svg.appendChild(el("path", { d, fill: "none", stroke: s.color, "stroke-width": 2.2 }));
      s.values.forEach((v, i) => { if (v == null) return;
        svg.appendChild(el("circle", { cx: X(i), cy: A.y(v * 100), r: 3.5, fill: s.color }));
        svg.appendChild(el("text", { x: X(i), y: A.y(v * 100) - 8, "text-anchor": "middle", "font-size": 8.5, fill: s.color, "font-weight": 700 }, fmt(v * 100, 1) + " %")); });
    });
    legend("l2", q.out);
    table("t2", "Kennzahl (%)", q.out, v => v == null ? "–" : fmt(v * 100, 1) + " %");
  } else { document.getElementById("c2").remove(); document.getElementById("l2").remove(); document.getElementById("t2").remove(); }
  document.getElementById("n2").textContent = q.skipped.length ? "Nicht gezeichnet (weniger als zwei Werte): " + q.skipped.map(s => s.name).join(", ") : "";
  document.getElementById("fuss").textContent = "Quelle: " + DATA.quelle + " · Datenstand " + DATA.datenstand + " · Maßgeblich bleibt der amtliche Auszug.";
})();
</script>
</body>
</html>
```
