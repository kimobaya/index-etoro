<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Simple Trend Watchlist</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body { font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; margin: 0; background:#0f141a; color:#e6edf3; }
    header { padding: 16px; border-bottom: 1px solid #223140; position: sticky; top: 0; background:#0f141a; }
    h1 { margin: 0; font-size: 18px; }
    .sub { color:#9fb0c3; font-size: 12px; margin-top: 4px; }
    main { max-width: 900px; margin: 0 auto; padding: 16px; display: grid; gap: 16px; }
    .card { background:#121820; border:1px solid #223140; border-radius:10px; padding:14px; }
    .row { display: flex; gap: 8px; flex-wrap: wrap; }
    input, button { padding: 10px 12px; border-radius: 8px; border:1px solid #223140; background:#0e141b; color:#e6edf3; }
    button { cursor: pointer; }
    .list { display: grid; gap: 10px; }
    .item { background:#0e141b; border:1px dashed #223140; border-radius:8px; padding:10px; }
    .muted { color:#9fb0c3; font-size: 12px; }
    .badge { font-size: 11px; padding:2px 6px; border-radius: 6px; border:1px solid #223140; }
    .green { color:#2ea043; background:#0d1a12; border-color:#1f4b2b; }
    .red { color:#ff6b6b; background:#190d0d; border-color:#4b1f1f; }
    a { color:#58a6ff; text-decoration: none; }
    a:hover { text-decoration: underline; }
  </style>
</head>
<body>
  <header>
    <h1>Simple Trend Watchlist</h1>
    <div class="sub"><span id="today"></span> · Auto-refresh on load · Timezone: Europe/Berlin</div>
  </header>

  <main>
    <section class="card">
      <div class="row">
        <input id="tickerInput" placeholder="Add ticker, e.g. NVDA, AAPL" />
        <button id="addBtn">Add</button>
        <button id="scanBtn">Refresh</button>
      </div>
      <p class="muted">
        Tip: Add a few tickers you care about. Headlines are pulled from Google News. 
        Tiny sentiment: +1 (positive word), −1 (negative word), 0 (neutral).
      </p>
      <div id="watchlist" class="list"></div>
    </section>

    <section class="card">
      <h3 style="margin:0 0 8px 0;">Latest global headlines</h3>
      <div id="feed" class="list"></div>
    </section>
  </main>

  <footer class="muted" style="max-width:900px;margin:24px auto 40px;padding:0 16px;">
    🚨 Day trading is high risk. This is not financial advice.
  </footer>

  <script>
    // ===== Minimal config =====
    const TZ = "Europe/Berlin";
    const LOOKBACK_HOURS = 30;     // only show headlines from the last N hours
    const MAX_PER_TICKER = 5;      // show top N headlines per ticker
    const GLOBAL_QUERIES = ["earnings","guidance","upgrade","downgrade","acquisition","merger"];

    // Very small keyword lists for naive sentiment scoring
    const POS = ["beats","beat","tops","raises guidance","surge","soars","approval","approved","upgrade","overweight","buy rating","inflows"];
    const NEG = ["misses","cuts guidance","downgrade","underweight","sell rating","investigation","probe","lawsuit","slumps","plunge","falls","warning","outflows"];

    // ===== Helpers =====
    const $ = sel => document.querySelector(sel);
    const fmtDate = d => new Intl.DateTimeFormat("de-DE", {
      weekday: "short", year: "numeric", month: "2-digit", day: "2-digit",
      hour: "2-digit", minute: "2-digit", timeZone: TZ
    }).format(d);

    function withinLookback(pubDate) {
      const t = new Date(pubDate).getTime();
      return (Date.now() - t) <= LOOKBACK_HOURS * 3600 * 1000;
    }

    function score(text) {
      const t = (text || "").toLowerCase();
      let s = 0;
      POS.forEach(k => { if (t.includes(k)) s += 1; });
      NEG.forEach(k => { if (t.includes(k)) s -= 1; });
      return s;
    }

    function rssUrl(query) {
      // Use Google News RSS via a free json wrapper (rate-limited)
      const q = encodeURIComponent(query + " stock OR shares");
      return "https://api.rss2json.com/v1/api.json?rss_url=" +
             encodeURIComponent(`https://news.google.com/rss/search?q=${q}&hl=en-US&gl=US&ceid=US:en`);
    }

    async function fetchHeadlines(query) {
      try {
        const res = await fetch(rssUrl(query));
        if (!res.ok) throw new Error(res.status);
        const data = await res.json();
        return (data.items || []).filter(it => it?.title && withinLookback(it.pubDate));
      } catch (e) { return []; }
    }

    // ===== State =====
    let tickers = ["NVDA","AAPL","MSFT"]; // start simple; you can remove/replace

    // ===== Render =====
    function renderToday() { $("#today").textContent = "Today: " + fmtDate(new Date()); }

    async function renderWatchlist() {
      const box = $("#watchlist");
      box.innerHTML = "<div class='muted'>Loading…</div>";
      const items = [];

      for (const t of tickers) {
        const set = await fetchHeadlines(t);
        set.sort((a,b)=> new Date(b.pubDate) - new Date(a.pubDate));
        const top = set.slice(0, MAX_PER_TICKER).map(h => ({
          title: h.title, link: h.link, pubDate: h.pubDate, s: score(h.title)
        }));
        const sum = top.reduce((acc, h) => acc + h.s, 0);
        items.push({ t, top, sum });
      }

      // sort by absolute sentiment, then recency
      items.sort((a,b)=> Math.abs(b.sum) - Math.abs(a.sum) ||
                         (new Date(b.top?.[0]?.pubDate||0) - new Date(a.top?.[0]?.pubDate||0)));

      // draw
      box.innerHTML = "";
      items.forEach(({ t, top, sum }) => {
        const badge = sum >= 2 ? `<span class="badge green">Bias: Long · +${sum}</span>`
                    : sum <= -2 ? `<span class="badge red">Bias: Short · ${sum}</span>`
                    : `<span class="badge">Bias: Neutral · ${sum >= 0 ? "+" : ""}${sum}</span>`;

        const div = document.createElement("div");
        div.className = "item";
        div.innerHTML = `<div><strong>${t}</strong> ${badge}</div>`;
        if (top.length === 0) {
          div.innerHTML += `<div class="muted">No fresh headlines in the last ${LOOKBACK_HOURS}h.</div>`;
        } else {
          top.forEach(h => {
            const tag = h.s > 0 ? '<span class="badge green">+1</span>' :
                        h.s < 0 ? '<span class="badge red">−1</span>' :
                        '<span class="badge">0</span>';
            div.innerHTML += `<div>${tag} <a href="${h.link}" target="_blank" rel="noopener">${h.title}</a>
              <div class="muted">${fmtDate(new Date(h.pubDate))}</div></div>`;
          });
        }
        box.appendChild(div);
      });
    }

    async function renderFeed() {
      const box = $("#feed");
      box.innerHTML = "<div class='muted'>Loading…</div>";
      let pool = [];
      for (const q of GLOBAL_QUERIES) {
        const rows = await fetchHeadlines(q);
        pool = pool.concat(rows);
      }
      pool.sort((a,b)=> new Date(b.pubDate) - new Date(a.pubDate));
      box.innerHTML = "";
      pool.slice(0, 20).forEach(h => {
        const s = score(h.title);
        const tag = s > 0 ? '<span class="badge green">+1</span>' :
                    s < 0 ? '<span class="badge red">−1</span>' :
                    '<span class="badge">0</span>';
        const div = document.createElement("div");
        div.className = "item";
        div.innerHTML = `${tag} <a href="${h.link}" target="_blank" rel="noopener">${h.title}</a>
                         <div class="muted">${fmtDate(new Date(h.pubDate))}</div>`;
        box.appendChild(div);
      });
    }

    // ===== Events =====
    $("#addBtn").addEventListener("click", () => {
      const v = $("#tickerInput").value.toUpperCase().trim();
      if (v && !tickers.includes(v)) tickers.push(v);
      $("#tickerInput").value = "";
      renderWatchlist();
    });

    $("#scanBtn").addEventListener("click", () => {
      renderWatchlist();
      renderFeed();
    });

    // ===== Boot =====
    (async function boot() {
      renderToday();
      await renderWatchlist();
      await renderFeed();
    })();
  </script>
</body>
</html>
