<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1"/>
<title>eToroPro • News & Charts</title>
<meta name="description" content="Daytrading-Dashboard: Trend-Discovery, dynamische Watchlist, Live-Charts und Sentiment-Grafiken."/>
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
<style>
:root{
  --bg:#0b0f14;--surface:#0f151e;--card:#121a24;--muted:#9fb0c3;--text:#e6edf3;--link:#58a6ff;
  --border:#223140;--accent:#2ea043;--danger:#ff6b6b;--warn:#f2c94c;
}
*{box-sizing:border-box} html,body{margin:0;background:var(--bg);color:var(--text);font-family:Inter,system-ui,Segoe UI,Helvetica,Arial,sans-serif}
a{color:var(--link);text-decoration:none} a:hover{text-decoration:underline}
header{padding:18px;border-bottom:1px solid var(--border);position:sticky;top:0;z-index:10;background:linear-gradient(180deg,rgba(11,15,20,.95),rgba(11,15,20,.7))}
h1{margin:0;font-size:22px;letter-spacing:.2px} .sub{color:var(--muted);font-size:12px;margin-top:4px}
main{max-width:1250px;margin:0 auto;padding:16px;display:grid;gap:16px}
.grid{display:grid;gap:16px} @media(min-width:1100px){.grid{grid-template-columns:1.25fr .75fr}}
.card{background:var(--card);border:1px solid var(--border);border-radius:14px;padding:16px}
.card h2{margin:0 0 10px;font-size:16px}
.row{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
.pill{display:inline-flex;align-items:center;gap:6px;padding:6px 10px;border:1px solid var(--border);border-radius:999px;background:#0f2232;color:var(--muted);font-size:12px}
.btn{background:#0f2532;border:1px solid var(--border);color:var(--text);padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600;font-size:14px}
.btn:hover{filter:brightness(1.12)}
input{background:#0e141b;border:1px solid var(--border);border-radius:10px;color:var(--text);padding:10px 12px}
.small{font-size:12px;color:var(--muted)}
.list{display:grid;gap:10px}
.item{padding:12px;border:1px dashed var(--border);border-radius:12px;background:#0e141b}
.item h3{margin:0 0 8px;font-size:14px}
.badge{font-size:11px;padding:2px 8px;border-radius:6px;border:1px solid var(--border);background:#0e1822;color:var(--muted)}
.badge.green{color:var(--accent);border-color:#1e4728;background:#0d1a12}
.badge.red{color:var(--danger);border-color:#4b1f1f;background:#190d0d}
.sep{height:1px;background:var(--border);margin:14px 0}
footer{max-width:1250px;margin:22px auto 40px;padding:0 16px;color:var(--muted)}
.kpis{display:grid;gap:10px;grid-template-columns:repeat(2,1fr)} @media(min-width:680px){.kpis{grid-template-columns:repeat(4,1fr)}}
.kpi{background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:12px}
.kpi .v{font-weight:700;font-size:18px}
.chartWrap{background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:12px}
</style>
</head>
<body>
<header>
  <h1>eToroPro • News & Charts</h1>
  <div class="sub"><span id="today"></span> · Auto-Update beim Laden · Zeitzone: Europe/Berlin</div>
</header>

<main>
  <!-- KPIs -->
  <section class="kpis">
    <div class="kpi"><div class="small">Gesamt-Mentions heute</div><div id="kpiMentions" class="v">–</div></div>
    <div class="kpi"><div class="small">Durchschn. Sentiment</div><div id="kpiSent" class="v">–</div></div>
    <div class="kpi"><div class="small">Watchlist-Anzahl</div><div id="kpiWL" class="v">–</div></div>
    <div class="kpi"><div class="small">Letztes Update</div><div id="kpiUpd" class="v">–</div></div>
  </section>

  <section class="grid">
    <!-- LEFT: Discovery + Watchlist -->
    <section class="card">
      <h2>🔥 Trend-Discovery & Watchlist</h2>
      <div class="row">
        <input id="addTicker" placeholder="Ticker hinzufügen (z. B. NVDA, TSLA, AAPL)"/>
        <button class="btn" id="addBtn">Hinzufügen</button>
        <button class="btn" id="scanBtn">Jetzt scannen</button>
        <button class="btn" id="clearBtn">Liste leeren</button>
        <span class="pill"><span>Lookback:</span><strong id="lookbackPill"></strong></span>
      </div>
      <div class="sep"></div>
      <div id="watchlist" class="list"></div>
      <p class="small">Bias basiert auf Schlagwort-Sentiment (+1/0/−1) aus frischen Google-News-Headlines. Entry: 5-Min-ORB & VWAP mit Volumen-Spike. Kein Overnight.</p>
    </section>

    <!-- RIGHT: Charts -->
    <aside class="card">
      <h2>📊 Grafiken</h2>
      <div class="chartWrap"><canvas id="barMentions" height="180"></canvas></div>
      <div class="chartWrap" style="margin-top:12px"><canvas id="barSentiment" height="180"></canvas></div>
      <div class="chartWrap" style="margin-top:12px"><canvas id="pieBias" height="180"></canvas></div>
    </aside>
  </section>

  <!-- LIVE MARKETS -->
  <section class="card">
    <h2>📈 Live-Märkte (TradingView)</h2>
    <div class="tradingview-widget-container">
      <div id="tv_overview"></div>
    </div>
    <p class="small">Klicke in der Watchlist auf „TradingView“, um direkt in den Einzelchart zu springen.</p>
  </section>

  <!-- HOTLISTS -->
  <section class="card">
    <h2>🔥 Hotlists / Most Active</h2>
    <div class="tradingview-widget-container"><div id="tv_hotlists"></div></div>
  </section>

  <!-- HEADLINES -->
  <section class="card">
    <h2>📰 Frische Schlagzeilen (global)</h2>
    <div id="headlines" class="list"></div>
  </section>
</main>

<footer>
  🚨 <strong>Risikohinweis:</strong> Daytrading ist hochriskant. Dies ist keine Anlageberatung. Teste das Regelwerk zunächst im Demokonto.
</footer>

<!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
<!-- Dein JavaScript kommt hier, analog zur Version die ich dir oben gegeben habe -->
<script>
// hier bleibt der komplette JavaScript-Code wie in der vorherigen Antwort,
// wegen Länge habe ich den JS-Teil weggelassen.
// -> Du kannst einfach den Script-Block aus meiner letzten Nachricht hineinkopieren!
</script>

<!-- TradingView Widgets -->
<script>
// TradingView Widget Code (wie oben)
</script>
</body>
</html>
