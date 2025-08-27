<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Trend-Scanner Daytrader – Open Watchlist</title>
<meta name="description" content="One-Pager: automatische Trend-Suche, dynamische Watchlist und Daytrading-Plan."/>
<style>
:root{
  --bg:#0b0f14;--card:#121820;--muted:#9fb0c3;--text:#e6edf3;
  --accent:#2ea043;--danger:#ff6b6b;--link:#58a6ff;--border:#223140;
}
*{box-sizing:border-box} html,body{margin:0;background:var(--bg);color:var(--text);font-family:Inter,system-ui,Segoe UI,Helvetica,Arial,sans-serif}
a{color:var(--link);text-decoration:none} a:hover{text-decoration:underline}
header{padding:22px 18px 8px;border-bottom:1px solid var(--border);position:sticky;top:0;background:rgba(11,15,20,.9);backdrop-filter:blur(6px);z-index:5}
h1{margin:0;font-size:22px} .sub{color:var(--muted);font-size:13px;margin-top:6px}
main{padding:16px 18px 40px;max-width:1200px;margin:0 auto}
.grid{display:grid;gap:16px} @media(min-width:980px){.grid{grid-template-columns:1.15fr .85fr}}
.card{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:16px}
.card h2{font-size:16px;margin:0 0 10px}
.pill{display:inline-block;padding:4px 10px;border-radius:999px;background:#0e2533;border:1px solid var(--border);font-size:12px;color:var(--muted);margin-right:6px;margin-bottom:6px}
.row{display:flex;gap:8px;flex-wrap:wrap}
.btn{background:#0f2532;border:1px solid var(--border);color:var(--text);padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600;font-size:14px}
.btn:hover{filter:brightness(1.12)}
.badge{font-size:12px;padding:2px 8px;border-radius:6px;border:1px solid var(--border);background:#0e1923;color:var(--muted)}
.badge.green{color:var(--accent);border-color:#1f4b2b;background:#0d1a12}
.badge.red{color:var(--danger);border-color:#4b1f1f;background:#190d0d}
.muted{color:var(--muted)} .small{font-size:12px;color:var(--muted)}
.list{display:grid;gap:10px}
.item{padding:12px;border:1px dashed var(--border);border-radius:10px;background:#0e141b}
.item h3{margin:0 0 6px;font-size:14px}
.sep{height:1px;background:var(--border);margin:16px 0}
.two{display:grid;gap:10px} @media(min-width:680px){.two{grid-template-columns:1fr 1fr}}
input,textarea{background:#0e141b;border:1px solid var(--border);border-radius:10px;color:var(--text);padding:10px 12px}
footer{max-width:1200px;margin:24px auto 50px;color:var(--muted);padding:0 18px}
</style>
</head>
<body>
<header>
  <h1>Trend-Scanner Daytrader (Open Watchlist)</h1>
  <div class="sub"><span id="today"></span> · Auto-Update bei jedem Laden · Zeitzone: Europe/Berlin</div>
</header>

<main class="grid">
  <!-- LEFT: Discovery + Watchlist -->
  <section class="card">
    <h2>🔥 Trend-Discovery (News → dynamische Watchlist)</h2>
    <div class="row small">
      <span class="pill">Quelle: Google News RSS (earnings/guidance/upgrade/downgrade/M&A)</span>
      <span class="pill" id="lookback-pill"></span>
      <span class="pill" id="universe-pill"></span>
    </div>
    <div class="two" style="margin-top:8px">
      <div class="row">
        <button class="btn" id="scan">Jetzt scannen</button>
        <button class="btn" id="reset">Liste leeren</button>
      </div>
      <div class="row">
        <input id="addTicker" placeholder="Ticker hinzufügen (z. B. NKE, BABA)"/>
        <button class="btn" id="addBtn">Hinzufügen</button>
      </div>
    </div>
    <div class="sep"></div>
    <div id="watchlist" class="list"></div>
    <div class="hint small">Bias basiert auf Headline-Keywords. Entry: 5-Min ORB &amp; VWAP mit Volumen-Spike. Kein Overnight.</div>
  </section>

  <!-- RIGHT: Playbook + Hotlists -->
  <aside class="card">
    <h2>Regel-Sheet (Intraday)</h2>
    <ul class="small">
      <li>Universum: liquide US-Large/Mid Caps, ggf. hinzugefügte Ticker.</li>
      <li>Katalysatoren: Earnings, Guidance, M&amp;A, FDA/Deals, Up/Down-Grades.</li>
      <li>Bias: Sentiment ≥ +2 → Long; ≤ −2 → Short.</li>
      <li>Entry Long: 5-Min ORB-High <em>und</em> über VWAP (Volumen-Spike).</li>
      <li>Entry Short: 5-Min ORB-Low <em>und</em> unter VWAP (Volumen-Spike).</li>
      <li>Risk: max. 1 % pro Trade. Stop: Signal-Kerze oder −ATR(5)×0,8.</li>
      <li>Exit: 2R Take-Profit, dann Trailing (unter letztem 5-Min-Swing/VWAP).</li>
      <li>Alles bis 21:55–22:00 Uhr MESZ schließen.</li>
    </ul>
    <div class="sep"></div>
    <div class="two">
      <a class="btn" href="https://www.etoro.com/" target="_blank" rel="noopener">eToro öffnen</a>
      <button class="btn" id="refresh">Neu laden</button>
    </div>
    <p class="small muted">Hinweis: Diese Seite liefert Signale/Plan. Auto-Trading auf eToro ist regional/Account-abhängig beschränkt.</p>
  </aside>

  <!-- HOTLISTS -->
  <section class="card" style="grid-column:1/-1">
    <h2>🔥 Hotlists / Most Active (TradingView)</h2>
    <div class="tradingview-widget-container">
      <div id="tv_hotlists"></div>
    </div>
  </section>

  <!-- HEADLINES FEED -->
  <section class="card" style="grid-column:1/-1">
    <h2>Frische Schlagzeilen (global)</h2>
    <div id="headlines" class="list"></div>
  </section>
</main>

<footer>
  <p>🚨 <strong>Risikohinweis:</strong> Daytrading ist hochriskant. Dies ist keine Anlageberatung. Teste das Regelwerk zunächst im Demokonto.</p>
</footer>

<script>
// ===== CONFIG =====
const CONFIG = {
  timezone:"Europe/Berlin",
  lookbackHours: 30,
  headlinesPerTicker: 6,
  // groß & liquide + bekannte Mover – dient als Erkennungs-Universum
  baseUniverse: [
    "AAPL","MSFT","AMZN","GOOGL","META","NVDA","TSLA","BRK.B","JPM","V","MA","UNH","HD","PG","XOM","CVX","AVGO","NFLX","PEP","KO",
    "CSCO","ADBE","CRM","ORCL","INTC","AMD","QCOM","TXN","AMAT","MU","SMCI","SMH","ASML","TSM","NIO","LI","BABA","PDD","TCEHY",
    "SHOP","SQ","PYPL","UBER","ABNB","BKNG","RIVN","LCID","PLTR","SNOW","DDOG","ZS","CRWD","OKTA","MDB","NET","PANW",
    "BA","LMT","NOC","GE","CAT","DE","HON","MMM","UPS","FDX","DAL","UAL",
    "WMT","COST","TGT","KSS","NKE","MCD","SBUX","CMG","TAP","KO","PEP",
    "JNJ","PFE","MRK","ABBV","LLY","BMY","AMGN","REGN","GILD","ISRG",
    "XLE","XLF","XLK","XLY","XLV","XLI","XLB","XLU","IWM","QQQ","SPY"
  ],
  posLex:["beats","beat","tops","smashes","raises guidance","raises outlook","hikes","surge","soars","approval","approved","acquires","record","strong","bullish","upgrade","overweight","buy rating","inflows","call buying","accumulates"],
  negLex:["misses","miss","cuts guidance","cuts outlook","downgrade","underweight","sell rating","investigation","probe","lawsuit","recall","slumps","plunge","falls","bearish","warning","halt","outflows","short seller"],
  discoveryQueries:["earnings","guidance","acquisition","merger","upgrade","downgrade","activist investor","stake","ETF inflows","most active"]
};

// ===== Utils =====
const fmtDate = d => new Intl.DateTimeFormat('de-DE',{weekday:'short',year:'numeric',month:'2-digit',day:'2-digit',hour:'2-digit',minute:'2-digit',timeZone:CONFIG.timezone}).format(d);
const $ = s => document.querySelector(s);
function el(tag, attrs={}, children=[]){const n=document.createElement(tag);Object.entries(attrs).forEach(([k,v])=>k==="class"?n.className=v:k==="html"?n.innerHTML=v:n.setAttribute(k,v));children.forEach(c=>n.appendChild(c));return n;}
function scoreHeadline(text){const t=text.toLowerCase();let s=0;CONFIG.posLex.forEach(k=>{if(t.includes(k))s+=1});CONFIG.negLex.forEach(k=>{if(t.includes(k))s-=1});return s;}
function withinLookback(pub){return (Date.now()-new Date(pub).getTime())<=CONFIG.lookbackHours*3600*1000}
function buildRss(query){const q=encodeURIComponent(query+" stock OR shares");return `https://news.google.com/rss/search?q=${q}&hl=en-US&gl=US&ceid=US:en`}
async function fetchRss(url){const api="https://api.rss2json.com/v1/api.json?rss_url="+encodeURIComponent(url);const r=await fetch(api);if(!r.ok)throw new Error("RSS "+r.status);return r.json()}

// ===== State =====
let openList = new Set(); // vom Nutzer hinzugefügt
let discovered = new Map(); // ticker -> {hits, headlines, sum}

// ===== Render Header pills =====
function renderMeta(){
  $("#today").textContent = "Heute: "+fmtDate(new Date());
  $("#lookback-pill").textContent = "Lookback: "+CONFIG.lookbackHours+"h";
  $("#universe-pill").textContent = "Universum: "+CONFIG.baseUniverse.length+"+ Symbole";
}

// ===== Add/remove user tickers =====
function addUserTicker(t){
  t = (t||"").toUpperCase().trim();
  if(!t) return;
  openList.add(t);
  buildWatchlist(); $("#addTicker").value="";
}
$("#addBtn")?.addEventListener("click",()=>addUserTicker($("#addTicker").value));
$("#addTicker")?.addEventListener("keydown",e=>{if(e.key==="Enter") addUserTicker($("#addTicker").value)});
$("#reset")?.addEventListener("click",()=>{openList.clear(); discovered.clear(); buildWatchlist()});

// ===== Discovery: scan global headlines, detect tickers, then fetch per-ticker headlines =====
async function discoverTrends(){
  discovered.clear();
  const all = [];
  for(const q of CONFIG.discoveryQueries){
    try{
      const rss = await fetchRss(buildRss(q));
      (rss.items||[]).forEach(it=>{
        if(it?.title && withinLookback(it.pubDate)){
          all.push({title:it.title, link:it.link, pubDate:it.pubDate});
        }
      });
    }catch(e){}
  }
  // Tokenize titles → match baseUniverse tickers
  const uni = new Set(CONFIG.baseUniverse);
  const counts = new Map();
  all.forEach(h=>{
    // einfache Tokenisierung auf KÜRZEL
    const tokens = h.title.toUpperCase().replace(/[^A-Z0-9\s\.\-]/g," ").split(/\s+/).filter(Boolean);
    const seen = new Set();
    tokens.forEach(tok=>{
      // handle BRK.B etc.
      const clean = tok.replace(/[^A-Z0-9\.]/g,"");
      if(uni.has(clean) && !seen.has(clean)){
        counts.set(clean,(counts.get(clean)||0)+1);
        seen.add(clean);
      }
    });
  });
  // Top candidates
  const top = [...counts.entries()].sort((a,b)=>b[1]-a[1]).slice(0,20).map(([t])=>t);
  // Fetch per-ticker headlines to get sentiment and links
  for(const tkr of top){
    const kw = tkr; // simple: Ticker als Query
    let heads=[];
    try{
      const rss = await fetchRss(buildRss(kw));
      (rss.items||[]).forEach(it=>{
        if(it?.title && withinLookback(it.pubDate)){
          heads.push({title:it.title,link:it.link,pubDate:it.pubDate,score:scoreHeadline(it.title)});
        }
      });
    }catch(e){}
    heads.sort((a,b)=>new Date(b.pubDate)-new Date(a.pubDate));
    const sum = heads.slice(0,CONFIG.headlinesPerTicker).reduce((acc,h)=>acc+h.score,0);
    discovered.set(tkr,{hits:counts.get(tkr)||0,headlines:heads,sum});
  }
}

// ===== Build watchlist (discovered + user) =====
async function buildWatchlist(){
  const box = $("#watchlist"); box.innerHTML = "<div class='small muted'>Baue Watchlist…</div>";
  const items=[];
  // discovered to array
  discovered.forEach((v,t)=>items.push({ticker:t, sum:v.sum, hits:v.hits, headlines:v.headlines}));
  // user-added without headlines -> fetch some quickly
  for(const t of openList){
    if(!items.find(i=>i.ticker===t)){
      let heads=[]; try{
        const rss = await fetchRss(buildRss(t));
        (rss.items||[]).forEach(it=>{ if(it?.title && withinLookback(it.pubDate)) heads.push({title:it.title,link:it.link,pubDate:it.pubDate,score:scoreHeadline(it.title)}) });
      }catch(e){}
      heads.sort((a,b)=>new Date(b.pubDate)-new Date(a.pubDate));
      const sum = heads.slice(0,CONFIG.headlinesPerTicker).reduce((a,h)=>a+h.score,0);
      items.push({ticker:t,sum, hits:heads.length, headlines:heads});
    }
  }
  // sort: by |sum|, then recency
  items.sort((a,b)=>Math.abs(b.sum)-Math.abs(a.sum) || new Date(b.headlines?.[0]?.pubDate||0)-new Date(a.headlines?.[0]?.pubDate||0));

  // render
  box.innerHTML="";
  if(items.length===0){ box.innerHTML = "<div class='small muted'>Noch keine Treffer. Klicke auf <b>Jetzt scannen</b> oder füge Ticker hinzu.</div>"; return; }

  items.forEach(it=>{
    const bias = it.sum>=2?"long":(it.sum<=-2?"short":"neutral");
    const badge = bias==="long"?"<span class='badge green'>Bias: Long</span>":(bias==="short"?"<span class='badge red'>Bias: Short</span>":"<span class='badge'>Bias: Neutral</span>");
    const card = el("div",{class:"item"});
    card.appendChild(el("h3",{html:`${it.ticker} ${badge} <span class="badge">Sentiment: ${it.sum>=0?"+":""}${it.sum}</span> <span class="badge">Mentions: ${it.hits||0}</span>`}));
    const row = el("div",{class:"row"});
    const tv = `https://www.tradingview.com/chart/?symbol=${encodeURIComponent(exchangeGuess(it.ticker)+":"+it.ticker)}`;
    const etoro = `https://www.etoro.com/markets/${it.ticker.toLowerCase()}`;
    row.appendChild(el("a",{class:"btn",href:tv,target:"_blank",rel:"noopener",html:"Chart öffnen (TradingView)"}));
    row.appendChild(el("a",{class:"btn",href:etoro,target:"_blank",rel:"noopener",html:"Bei eToro ansehen"}));
    card.appendChild(row);
    const list = el("div",{class:"list"});
    (it.headlines||[]).slice(0,CONFIG.headlinesPerTicker).forEach(h=>{
      const s = h.score>0?'<span class="badge green">+1</span>':(h.score<0?'<span class="badge red">−1</span>':'<span class="badge">0</span>');
      list.appendChild(el("div",{class:"small",html:`${s} <a href="${h.link}" target="_blank" rel="noopener">${h.title}</a><div class="small muted">${fmtDate(new Date(h.pubDate))}</div>`}));
    });
    if(!it.headlines?.length) list.appendChild(el("div",{class:"small muted",html:"Keine frischen Headlines."}));
    card.appendChild(el("div",{class:"sep"}));
    card.appendChild(el("div",{class:"small muted",html:"<strong>Plan:</strong> 5-Min ORB + VWAP + Volumen. Stop = Signal-Kerze / −ATR(5)×0,8. TP 2R, dann Trailing."}));
    box.appendChild(card);
  });
}

// naive exchange guess for TradingView link
function exchangeGuess(t){ return ["AAPL","MSFT","AMZN","GOOGL","META","NVDA","TSLA","AMD","MU","PLTR","OKTA","MDB","CRWD","SNOW","ZS","ABNB","UBER","RIVN","LCID","NKE","KSS"].includes(t) ? "NASDAQ" : "NYSE"; }

// ===== Headlines feed (generic) =====
async function buildHeadlines(){
  const container = $("#headlines"); container.innerHTML = "<div class='small muted'>Lade Headlines…</div>";
  let pool=[];
  for(const q of CONFIG.discoveryQueries){
    try{
      const rss = await fetchRss(buildRss(q));
      (rss.items||[]).forEach(it=>{ if(it?.title && withinLookback(it.pubDate)) pool.push({title:it.title,link:it.link,pubDate:it.pubDate,score:scoreHeadline(it.title)}) });
    }catch(e){}
  }
  pool.sort((a,b)=>new Date(b.pubDate)-new Date(a.pubDate));
  container.innerHTML="";
  pool.slice(0,24).forEach(h=>{
    const s = h.score>0?'<span class="badge green">+1</span>':(h.score<0?'<span class="badge red">−1</span>':'<span class="badge">0</span>');
    const item = el("div",{class:"item"});
    item.innerHTML = `<div>${s} <a href="${h.link}" target="_blank" rel="noopener">${h.title}</a></div><div class="small muted">${fmtDate(new Date(h.pubDate))}</div>`;
    container.appendChild(item);
  });
}

// ===== TradingView Widgets =====
function mountTV(){
  // Hotlists widget
  const s=document.createElement("script");
  s.type="text/javascript"; s.src="https://s3.tradingview.com/external-embedding/embed-widget-hotlists.js"; s.async=true;
  s.innerHTML = JSON.stringify({exchange:"US",colorTheme:"dark",locale:"en",width:"100%",height:420,largeChartUrl:""});
  document.querySelector("#tv_hotlists").innerHTML=""; document.querySelector("#tv_hotlists").appendChild(s);
}

// ===== Boot =====
async function boot(){
  renderMeta(); mountTV();
  await discoverTrends(); await buildWatchlist(); await buildHeadlines();
}
document.addEventListener("DOMContentLoaded", ()=>{
  boot();
  $("#refresh").addEventListener("click", boot);
  $("#scan").addEventListener("click", async()=>{ await discoverTrends(); await buildWatchlist(); });
});
</script>
</body>
</html>
