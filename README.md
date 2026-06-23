/* =========================================================
   SIGNALFORGE — APP CORE
   ========================================================= */
'use strict';

/* ---------- Storage helpers ---------- */
const DB = {
  get(key, fallback){
    try{ const v = localStorage.getItem(key); return v ? JSON.parse(v) : fallback; }
    catch(e){ return fallback; }
  },
  set(key, val){ try{ localStorage.setItem(key, JSON.stringify(val)); }catch(e){} }
};

const STORE_KEYS = { USERS:'sf_users', SESSION:'sf_session', HISTORY:'sf_history' };

function getUsers(){ return DB.get(STORE_KEYS.USERS, {}); }
function saveUsers(u){ DB.set(STORE_KEYS.USERS, u); }
function getSession(){ return DB.get(STORE_KEYS.SESSION, null); }
function setSession(email){ DB.set(STORE_KEYS.SESSION, email); }
function clearSession(){ localStorage.removeItem(STORE_KEYS.SESSION); }
function currentUser(){
  const email = getSession();
  if(!email) return null;
  const users = getUsers();
  return users[email] || null;
}
function getHistory(email){
  const all = DB.get(STORE_KEYS.HISTORY, {});
  return all[email] || [];
}
function addHistory(email, entry){
  const all = DB.get(STORE_KEYS.HISTORY, {});
  if(!all[email]) all[email] = [];
  all[email].unshift(entry);
  all[email] = all[email].slice(0, 30);
  DB.set(STORE_KEYS.HISTORY, all);
}

/* ---------- Toast ---------- */
function toast(msg, type='success'){
  const root = document.getElementById('toastRoot');
  const el = document.createElement('div');
  el.className = `toast ${type}`;
  el.innerHTML = `<span>${type==='success' ? '✓' : type==='error' ? '✕' : 'ℹ'}</span><span>${msg}</span>`;
  root.appendChild(el);
  setTimeout(()=>{ el.style.opacity='0'; el.style.transform='translateX(20px)'; el.style.transition='.25s'; setTimeout(()=>el.remove(),250); }, 3200);
}

/* ---------- Small utils ---------- */
function esc(s){ return String(s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c])); }
function fmtDate(ts){
  const d = new Date(ts);
  return d.toLocaleDateString('en-US', { month:'short', day:'numeric', year:'numeric' }) + ' · ' + d.toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit'});
}
function initials(nameOrEmail){
  const s = (nameOrEmail||'U').trim();
  const parts = s.split(/[\s@._]+/).filter(Boolean);
  if(parts.length >= 2) return (parts[0][0] + parts[1][0]).toUpperCase();
  return s.slice(0,2).toUpperCase();
}
function rand(min,max){ return Math.random()*(max-min)+min; }
function pick(arr){ return arr[Math.floor(Math.random()*arr.length)]; }
function clamp(v,min,max){ return Math.max(min, Math.min(max, v)); }

/* ---------- Icon set (inline SVG, stroke style) ---------- */
const ICONS = {
  upload: `<svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>`,
  chart: `<svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 3v18h18"/><path d="M7 16l4-6 3 3 5-8"/></svg>`,
  shield: `<svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>`,
  bolt: `<svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>`,
  target: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><circle cx="12" cy="12" r="5"/><circle cx="12" cy="12" r="1"/></svg>`,
  trend: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>`,
  check: `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>`,
  x: `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>`,
  arrowUp: `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="19" x2="12" y2="5"/><polyline points="5 12 12 5 19 12"/></svg>`,
  arrowDown: `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="5" x2="12" y2="19"/><polyline points="19 12 12 19 5 12"/></svg>`,
  buyIcon: `<svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="19" x2="12" y2="5"/><polyline points="5 12 12 5 19 12"/></svg>`,
  sellIcon: `<svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="5" x2="12" y2="19"/><polyline points="19 12 12 19 5 12"/></svg>`,
  holdIcon: `<svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="12" x2="19" y2="12"/></svg>`,
  gauge: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 14l3-3"/><path d="M2 12a10 10 0 1 1 20 0"/><path d="M12 22a2 2 0 1 0 0-4 2 2 0 0 0 0 4z"/></svg>`,
  risk: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 9v4"/><path d="M12 17h.01"/><path d="M10.3 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.7 3.86a2 2 0 0 0-3.4 0z"/></svg>`,
  zap: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>`,
  brain: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9.5 2A2.5 2.5 0 0 1 12 4.5v15a2.5 2.5 0 0 1-4.96.44 2.5 2.5 0 0 1-2.96-3.08 3 3 0 0 1-.34-5.58 2.5 2.5 0 0 1 1.32-4.24 2.5 2.5 0 0 1 2.44-3A2.5 2.5 0 0 1 9.5 2z"/><path d="M14.5 2A2.5 2.5 0 0 0 12 4.5v15a2.5 2.5 0 0 0 4.96.44 2.5 2.5 0 0 0 2.96-3.08 3 3 0 0 0 .34-5.58 2.5 2.5 0 0 0-1.32-4.24 2.5 2.5 0 0 0-2.44-3A2.5 2.5 0 0 0 14.5 2z"/></svg>`,
  lock: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="11" width="18" height="11" rx="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>`,
  mail: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 4h16v16H4z" stroke="none"/><path d="M22 6l-10 7L2 6"/><rect x="2" y="4" width="20" height="16" rx="2"/></svg>`,
  phone: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72c.12.81.3 1.6.54 2.36a2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45c.76.24 1.55.42 2.36.54A2 2 0 0 1 22 16.92z"/></svg>`,
  pin: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 1 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg>`,
  history: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 3v5h5"/><path d="M3.05 13A9 9 0 1 0 6 5.3L3 8"/><path d="M12 7v5l4 2"/></svg>`,
  user: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="8" r="4"/><path d="M4 21v-1a8 8 0 0 1 16 0v1"/></svg>`,
  settings: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 1 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 1 1-2.83-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 1 1 2.83-2.83l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 1 1 2.83 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/></svg>`,
  logout: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>`,
  credit: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="1" y="4" width="22" height="16" rx="2"/><line x1="1" y1="10" x2="23" y2="10"/></svg>`,
};

/* =========================================================
   MARKET DATA (mocked, realistic)
   ========================================================= */
const MARKET_DATA = {
  crypto: [
    { sym:'BTC', name:'Bitcoin', price:97540.32, chg:2.41, color:'#F7931A', icon:'₿' },
    { sym:'ETH', name:'Ethereum', price:3812.18, chg:1.86, color:'#627EEA', icon:'Ξ' },
    { sym:'SOL', name:'Solana', price:228.55, chg:-1.32, color:'#14F195', icon:'◎' },
    { sym:'BNB', name:'BNB', price:712.40, chg:0.64, color:'#F0B90B', icon:'B' },
    { sym:'XRP', name:'XRP', price:2.84, chg:-0.91, color:'#23292F', icon:'X' },
  ],
  forex: [
    { sym:'EUR/USD', name:'Euro / US Dollar', price:1.0862, chg:0.18, color:'#4D8DFF', icon:'€' },
    { sym:'GBP/USD', name:'Pound / US Dollar', price:1.2734, chg:-0.24, color:'#FF4757', icon:'£' },
    { sym:'USD/JPY', name:'Dollar / Yen', price:154.62, chg:0.41, color:'#16D38F', icon:'¥' },
    { sym:'AUD/USD', name:'Aussie / Dollar', price:0.6512, chg:-0.12, color:'#FFB020', icon:'A' },
    { sym:'USD/CHF', name:'Dollar / Franc', price:0.9043, chg:0.08, color:'#A78BFA', icon:'₣' },
  ],
  stocks: [
    { sym:'SPX', name:'S&P 500 Index', price:6184.45, chg:0.62, color:'#4D8DFF', icon:'S' },
    { sym:'NDX', name:'Nasdaq 100', price:21940.10, chg:0.95, color:'#16D38F', icon:'N' },
    { sym:'DJI', name:'Dow Jones', price:42880.30, chg:-0.18, color:'#FFB020', icon:'D' },
    { sym:'AAPL', name:'Apple Inc.', price:232.18, chg:1.12, color:'#A6B1C2', icon:'A' },
    { sym:'NVDA', name:'NVIDIA Corp.', price:148.92, chg:3.04, color:'#76B900', icon:'N' },
  ],
  commodities: [
    { sym:'XAU/USD', name:'Gold Spot', price:2384.70, chg:0.74, color:'#FFD700', icon:'Au' },
    { sym:'XAG/USD', name:'Silver Spot', price:30.18, chg:1.05, color:'#C0C0C0', icon:'Ag' },
    { sym:'WTI', name:'Crude Oil WTI', price:78.42, chg:-1.18, color:'#3B2F2F', icon:'O' },
    { sym:'NATGAS', name:'Natural Gas', price:3.12, chg:2.20, color:'#4D8DFF', icon:'G' },
    { sym:'XPT/USD', name:'Platinum Spot', price:1024.60, chg:-0.36, color:'#E5E4E2', icon:'Pt' },
  ],
};

function allAssetsFlat(){
  return Object.entries(MARKET_DATA).flatMap(([cat,list]) => list.map(a => ({...a, cat})));
}

function generateSparkPath(up, w=80, h=28){
  let pts = []; let y = h/2;
  const n = 12;
  for(let i=0;i<n;i++){
    const bias = up ? -0.6 : 0.6;
    y = clamp(y + rand(-3,3) + bias, 3, h-3);
    pts.push([ (i/(n-1))*w, y ]);
  }
  return pts.map(p=>p.join(',')).join(' ');
}

function sparkSvg(up, w=80, h=28){
  const pts = generateSparkPath(up, w, h);
  const color = up ? 'var(--green)' : 'var(--red)';
  const firstY = pts.split(' ')[0].split(',')[1];
  const lastPt = pts.split(' ').slice(-1)[0];
  return `<svg class="mini-spark" width="${w}" height="${h}" viewBox="0 0 ${w} ${h}">
    <polyline points="${pts}" fill="none" stroke="${color}" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
  </svg>`;
}

/* =========================================================
   MOCK AI ANALYSIS ENGINE
   Produces a deterministic-feeling but randomized "analysis"
   so every upload yields a plausible, internally consistent result.
   ========================================================= */
function runAnalysis(){
  const signal = pick(['BUY','SELL','HOLD']);
  const confidence = Math.round(rand(signal==='HOLD'?55:68, signal==='HOLD'?78:96));
  const riskLevel = pick(signal==='HOLD' ? ['Low','Medium'] : ['Low','Medium','High']);
  const trendStrength = Math.round(rand(2,5)); // out of 5
  const basePrice = rand(80,400);
  const direction = signal==='BUY' ? 1 : signal==='SELL' ? -1 : (Math.random()>0.5?1:-1);
  const volatility = riskLevel==='High' ? 0.045 : riskLevel==='Medium' ? 0.025 : 0.012;

  const entry = basePrice;
  const tp = basePrice * (1 + direction * volatility * rand(1.4,2.6));
  const sl = basePrice * (1 - direction * volatility * rand(0.8,1.4));

  const trendWord = signal==='BUY' ? pick(['Uptrend','Bullish Reversal','Breakout Continuation'])
                   : signal==='SELL' ? pick(['Downtrend','Bearish Reversal','Breakdown Continuation'])
                   : pick(['Sideways / Consolidation','Range-Bound','Indecisive']);

  const patterns = {
    BUY: ['an ascending triangle nearing breakout', 'a higher-low structure with rising volume', 'a bullish flag forming after a strong impulse leg', 'price reclaiming a key moving average with momentum confirmation'],
    SELL: ['a descending triangle with weakening support', 'a lower-high structure with declining volume on rallies', 'a bearish flag following a sharp impulse leg down', 'price rejecting a key moving average with momentum rolling over'],
    HOLD: ['a tight consolidation range with no clear directional bias', 'overlapping candles near the midpoint of recent range', 'price coiling between two converging moving averages', 'low-volatility compression ahead of a potential expansion move']
  };
  const indicatorNotes = {
    BUY: ['RSI curling up from oversold territory', 'MACD histogram flipping positive', 'price holding above the 50-period moving average', 'volume expanding on up-candles'],
    SELL: ['RSI rolling over from overbought territory', 'MACD histogram flipping negative', 'price rejected at the 50-period moving average', 'volume expanding on down-candles'],
    HOLD: ['RSI oscillating near the midline', 'MACD lines flat with minimal separation', 'price pinned between key moving averages', 'volume contracting, signaling indecision']
  };

  const summary = `The chart shows ${pick(patterns[signal])}. ${pick(indicatorNotes[signal])}, which supports a ${signal.toLowerCase()} bias at current levels. Momentum and structure are ${trendStrength>=4?'strongly':'moderately'} aligned with the detected trend.`;

  const insightsPool = {
    BUY: [
      'Watch for a confirmed close above resistance before adding to size.',
      'Volume should expand on the breakout to validate the move.',
      'Consider scaling in rather than entering the full position at once.',
      'A pullback to the breakout level often offers a lower-risk re-entry.'
    ],
    SELL: [
      'Watch for a confirmed close below support before adding to size.',
      'A failed bounce back to resistance can offer a lower-risk entry.',
      'Tighten risk if broader market volatility is elevated.',
      'Avoid chasing the move after an extended decline without a pullback.'
    ],
    HOLD: [
      'Wait for a decisive break of the range before committing capital.',
      'Reduce position size until volatility expansion confirms direction.',
      'Mark both range boundaries — the eventual breakout side often defines the next trend.',
      'This is typically a lower-conviction setup; conserve capital for higher-probability trades.'
    ]
  };
  const insights = [...insightsPool[signal]].sort(()=>Math.random()-0.5).slice(0,3);

  const asset = pick(allAssetsFlat());

  return {
    id: 'an_' + Date.now() + '_' + Math.floor(Math.random()*1000),
    timestamp: Date.now(),
    asset: asset.sym,
    signal, confidence, riskLevel, trendStrength, trendWord,
    entry: entry.toFixed(2), tp: tp.toFixed(2), sl: sl.toFixed(2),
    summary, insights,
  };
}

/* =========================================================
   PAGE: HOME
   ========================================================= */
function pageHome(){
  return `
  <div class="page">
    <section class="hero">
      <div class="hero-bg-grid"></div>
      <div class="hero-glow"></div>
      <div class="hero-inner">
        <div class="hero-copy">
          <span class="eyebrow"><span class="dot"></span> AI Chart Analysis · Live</span>
          <h1>Upload a chart.<br>Get an <span class="accent">AI trading signal</span> in seconds.</h1>
          <p class="lead">Drop in a screenshot of any candlestick chart — crypto, forex, stocks, or commodities — and SignalForge reads the structure, momentum, and trend to return a clear Buy, Sell, or Hold read with entry, target, and stop levels.</p>
          <div class="hero-cta-row">
            <a href="#/upload" data-link class="btn btn-primary btn-lg">${ICONS.upload} Upload Chart</a>
            <a href="#/markets" data-link class="btn btn-ghost btn-lg">View Live Markets</a>
          </div>
          <div class="hero-stats">
            <div class="hero-stat"><div class="num green">2.4M+</div><div class="label">Charts analyzed</div></div>
            <div class="hero-stat"><div class="num">86<span class="mono">%</span></div><div class="label">Avg. model confidence</div></div>
            <div class="hero-stat"><div class="num">4.8/5</div><div class="label">Trader rating</div></div>
          </div>
        </div>

        <div class="hero-visual">
          <div class="hero-float-card fc1"><div class="fc-label">CONFIDENCE</div><div class="fc-val green">91%</div></div>
          <div class="hero-float-card fc2"><div class="fc-label">RISK</div><div class="fc-val">Medium</div></div>
          <div class="scan-card">
            <div class="scan-card-head">
              <div>
                <div class="pair mono">BTC/USD · 4H</div>
              </div>
              <div class="scan-card-dots"><span></span><span></span><span></span></div>
            </div>
            <div class="scan-chart-wrap" id="heroChartWrap">
              <div class="scan-beam"></div>
              <span class="scan-annot buy">SUPPORT</span>
              <span class="scan-annot res">RESISTANCE</span>
            </div>
            <div class="scan-card-foot">
              <div class="scan-verdict">
                <span class="badge-buy">▲ BUY SIGNAL</span>
                <span class="conf">91% confidence</span>
              </div>
              <span class="scanning-label"><span class="dot"></span> analyzing live</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <div class="ticker-strip">
      <div class="ticker-track" id="tickerTrack"></div>
    </div>

    <section class="section">
      <div class="container">
        <div class="section-head center">
          <h2>From screenshot to signal in three steps</h2>
          <p>No connected broker, no manual setup. Just upload what you're already looking at.</p>
        </div>
        <div class="grid-3">
          <div class="card step-card">
            <div class="step-num">01</div>
            <h3>Upload your chart</h3>
            <p>Drag in a PNG, JPG, or JPEG screenshot from TradingView, MT4/5, or any platform.</p>
            <span class="step-arrow">${ICONS.trend}</span>
          </div>
          <div class="card step-card">
            <div class="step-num">02</div>
            <h3>AI reads the structure</h3>
            <p>Pattern recognition detects trend, support/resistance, candlestick formations, and momentum.</p>
            <span class="step-arrow">${ICONS.trend}</span>
          </div>
          <div class="card step-card">
            <div class="step-num">03</div>
            <h3>Get your signal</h3>
            <p>A Buy, Sell, or Hold call with confidence score, risk level, and suggested trade levels.</p>
          </div>
        </div>
      </div>
    </section>

    <section class="section" style="padding-top:0;">
      <div class="container">
        <div class="section-head center">
          <h2>Built like a trading terminal, not a toy</h2>
          <p>Every analysis is backed by a structured read of trend, momentum, and risk — presented the way a desk analyst would brief it.</p>
        </div>
        <div class="grid-3">
          <div class="card feature-card card-hover">
            <div class="feature-icon">${ICONS.brain}</div>
            <h3>Pattern recognition</h3>
            <p>Detects trend direction, chart patterns, and candlestick formations directly from the image you upload.</p>
          </div>
          <div class="card feature-card card-hover">
            <div class="feature-icon">${ICONS.target}</div>
            <h3>Entry, TP &amp; SL levels</h3>
            <p>Every signal ships with a suggested entry point, take-profit target, and stop-loss — not just a direction.</p>
          </div>
          <div class="card feature-card card-hover">
            <div class="feature-icon">${ICONS.gauge}</div>
            <h3>Confidence scoring</h3>
            <p>Each signal is graded with a confidence percentage and trend strength, so you can size positions accordingly.</p>
          </div>
          <div class="card feature-card card-hover">
            <div class="feature-icon">${ICONS.risk}</div>
            <h3>Risk classification</h3>
            <p>Low, Medium, or High risk tagging based on volatility and structure clarity in the chart.</p>
          </div>
          <div class="card feature-card card-hover">
            <div class="feature-icon">${ICONS.history}</div>
            <h3>Analysis history</h3>
            <p>Registered users get a saved log of every chart they've analyzed, with signals and outcomes side by side.</p>
          </div>
          <div class="card feature-card card-hover">
            <div class="feature-icon">${ICONS.shield}</div>
            <h3>Privacy-first uploads</h3>
            <p>Charts are processed for analysis only. You control what's saved to your history.</p>
          </div>
        </div>
      </div>
    </section>

    <section class="section" style="padding-top:0;">
      <div class="container">
        <div class="cta-band">
          <h2>See what the AI sees in your next setup.</h2>
          <p>Upload a chart and get a full breakdown — signal, confidence, risk, and trade levels — in under ten seconds.</p>
          <a href="#/upload" data-link class="btn btn-primary btn-lg">${ICONS.upload} Upload Your First Chart</a>
        </div>
      </div>
    </section>
  </div>`;
}

function renderTicker(){
  const track = document.getElementById('tickerTrack');
  if(!track) return;
  const assets = allAssetsFlat();
  const doubled = [...assets, ...assets];
  track.innerHTML = doubled.map(a => `
    <span class="ticker-item">
      <span class="sym">${a.sym}</span>
      <span class="mono">${a.price.toLocaleString(undefined,{maximumFractionDigits:2})}</span>
      <span class="chg ${a.chg>=0?'up':'down'}">${a.chg>=0?'▲':'▼'} ${Math.abs(a.chg).toFixed(2)}%</span>
    </span>
  `).join('');
}

/* =========================================================
   PAGE: UPLOAD ANALYSIS
   ========================================================= */
function pageUpload(){
  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / Upload Analysis</div>
        <h1>Upload your chart</h1>
        <p>PNG, JPG, or JPEG — a clean screenshot of the candlestick chart works best. We'll detect trend, structure, and momentum automatically.</p>
      </div>
    </div>

    <div class="container" style="padding:48px 28px 96px;">
      <div class="upload-shell">

        <div id="uploadStage">
          <div class="dropzone" id="dropzone" tabindex="0" role="button" aria-label="Upload chart image">
            <div class="dropzone-icon">${ICONS.upload}</div>
            <h3>Drag &amp; drop your chart here</h3>
            <p>or click to browse your files</p>
            <div class="formats mono">PNG · JPG · JPEG &nbsp;·&nbsp; Max 10MB</div>
          </div>
          <input type="file" id="fileInput" accept=".png,.jpg,.jpeg,image/png,image/jpeg">
        </div>

        <div id="previewStage" style="display:none;">
          <div class="card preview-card">
            <div class="preview-img-wrap" id="previewImgWrap">
              <img id="previewImg" alt="Uploaded chart preview">
            </div>
            <div class="preview-meta">
              <span id="previewFileName" class="mono"></span>
              <span id="previewFileSize" class="mono"></span>
            </div>
            <div class="preview-actions">
              <button class="btn btn-primary btn-block" id="analyzeBtn">${ICONS.bolt} Run AI Analysis</button>
              <button class="btn btn-ghost" id="clearBtn">${ICONS.x}</button>
            </div>
          </div>
        </div>

        <div class="card upload-tips">
          <h3 style="font-size:15px; margin-bottom:8px;">For the most accurate read</h3>
          <div class="tip-row">${ICONS.check}<span>Use a clear, unobstructed candlestick or OHLC chart.</span></div>
          <div class="tip-row">${ICONS.check}<span>Include at least 30–50 candles of price history if possible.</span></div>
          <div class="tip-row">${ICONS.check}<span>Avoid cropping out the price axis — scale context helps accuracy.</span></div>
          <div class="tip-row">${ICONS.check}<span>Screenshots from TradingView, MT4/5, or broker platforms work best.</span></div>
        </div>
      </div>
    </div>
  </div>`;
}

let _uploadedImageData = null;
let _uploadedFileMeta = null;

function initUploadPage(){
  const dropzone = document.getElementById('dropzone');
  const fileInput = document.getElementById('fileInput');
  const uploadStage = document.getElementById('uploadStage');
  const previewStage = document.getElementById('previewStage');
  const previewImg = document.getElementById('previewImg');
  const previewFileName = document.getElementById('previewFileName');
  const previewFileSize = document.getElementById('previewFileSize');
  const analyzeBtn = document.getElementById('analyzeBtn');
  const clearBtn = document.getElementById('clearBtn');

  if(!dropzone) return;

  const openPicker = () => fileInput.click();
  dropzone.addEventListener('click', openPicker);
  dropzone.addEventListener('keydown', (e)=>{ if(e.key==='Enter' || e.key===' ') openPicker(); });

  ['dragenter','dragover'].forEach(evt => dropzone.addEventListener(evt, e=>{
    e.preventDefault(); dropzone.classList.add('drag');
  }));
  ['dragleave','drop'].forEach(evt => dropzone.addEventListener(evt, e=>{
    e.preventDefault(); dropzone.classList.remove('drag');
  }));
  dropzone.addEventListener('drop', e=>{
    const file = e.dataTransfer.files[0];
    if(file) handleFile(file);
  });
  fileInput.addEventListener('change', e=>{
    const file = e.target.files[0];
    if(file) handleFile(file);
  });

  function handleFile(file){
    const validTypes = ['image/png','image/jpeg','image/jpg'];
    if(!validTypes.includes(file.type)){
      toast('Please upload a PNG, JPG, or JPEG image.', 'error');
      return;
    }
    if(file.size > 10*1024*1024){
      toast('File too large. Max size is 10MB.', 'error');
      return;
    }
    const reader = new FileReader();
    reader.onload = (e)=>{
      _uploadedImageData = e.target.result;
      _uploadedFileMeta = { name:file.name, size:(file.size/1024).toFixed(0)+' KB' };
      previewImg.src = _uploadedImageData;
      previewFileName.textContent = file.name;
      previewFileSize.textContent = _uploadedFileMeta.size;
      uploadStage.style.display = 'none';
      previewStage.style.display = 'block';
    };
    reader.readAsDataURL(file);
  }

  clearBtn.addEventListener('click', ()=>{
    _uploadedImageData = null; _uploadedFileMeta = null;
    fileInput.value = '';
    uploadStage.style.display = 'block';
    previewStage.style.display = 'none';
  });

  analyzeBtn.addEventListener('click', ()=>{
    if(!_uploadedImageData) return;
    const wrap = document.getElementById('previewImgWrap');
    const overlay = document.createElement('div');
    overlay.className = 'analyzing-overlay';
    overlay.innerHTML = `
      <div class="analyzing-beam"></div>
      <div class="spinner-ring"></div>
      <div class="analyzing-text" id="analyzeStepText">Detecting candlestick structure…</div>
      <div class="analyzing-sub">This usually takes a few seconds</div>
    `;
    wrap.style.position = 'relative';
    wrap.appendChild(overlay);
    analyzeBtn.disabled = true;

    const steps = [
      'Detecting candlestick structure…',
      'Mapping support & resistance…',
      'Evaluating momentum & volume bias…',
      'Scoring confidence & risk…',
      'Finalizing signal…'
    ];
    let i = 0;
    const stepEl = document.getElementById('analyzeStepText');
    const interval = setInterval(()=>{
      i++;
      if(i < steps.length && stepEl) stepEl.textContent = steps[i];
    }, 480);

    setTimeout(()=>{
      clearInterval(interval);
      const result = runAnalysis();
      result.image = _uploadedImageData;
      result.fileName = _uploadedFileMeta ? _uploadedFileMeta.name : 'chart.png';
      _pendingAnalysis = result;

      const user = currentUser();
      if(user){
        addHistory(user.email, result);
      }
      navigate('#/dashboard');
    }, 2500);
  });
}

let _pendingAnalysis = null;

/* =========================================================
   PAGE: DASHBOARD
   ========================================================= */
function pageDashboard(){
  const result = _pendingAnalysis;
  if(!result){
    return `
    <div class="page">
      <div class="container" style="padding:90px 28px;">
        <div class="card" style="max-width:560px; margin:0 auto; text-align:center; padding:48px;">
          <div class="feature-icon" style="margin:0 auto 18px;">${ICONS.chart}</div>
          <h2 style="font-size:22px; margin-bottom:10px;">No analysis yet</h2>
          <p style="margin-bottom:24px;">Upload a chart to see your AI trading signal dashboard here.</p>
          <a href="#/upload" data-link class="btn btn-primary">${ICONS.upload} Upload a Chart</a>
        </div>
      </div>
    </div>`;
  }

  const sigClass = result.signal.toLowerCase();
  const sigIcon = result.signal==='BUY' ? ICONS.buyIcon : result.signal==='SELL' ? ICONS.sellIcon : ICONS.holdIcon;
  const riskColor = result.riskLevel==='Low' ? 'green' : result.riskLevel==='Medium' ? 'amber' : 'red';
  const riskPct = result.riskLevel==='Low' ? 30 : result.riskLevel==='Medium' ? 62 : 90;

  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / Dashboard</div>
        <h1>Analysis Dashboard</h1>
        <p>AI-generated read of your uploaded chart. Review the full breakdown below before making any decisions.</p>
      </div>
    </div>

    <div class="container" style="padding:40px 28px 96px;">
      <div class="dash-grid">

        <div>
          <div class="signal-banner ${sigClass}">
            <div class="signal-left">
              <div class="signal-icon">${sigIcon}</div>
              <div>
                <div class="signal-label">AI Signal · ${esc(result.asset)}</div>
                <div class="signal-value">${result.signal}</div>
              </div>
            </div>
            <div class="signal-conf">
              <div class="conf-num">${result.confidence}%</div>
              <div class="conf-label">Confidence score</div>
            </div>
          </div>

          <div class="stat-grid">
            <div class="card stat-card">
              <div class="stat-label">${ICONS.trend} Market Trend</div>
              <div class="stat-val">${esc(result.trendWord)}</div>
            </div>
            <div class="card stat-card">
              <div class="stat-label">${ICONS.risk} Risk Level</div>
              <div class="stat-val ${riskColor}">${result.riskLevel}</div>
              <div class="risk-bar"><div class="risk-bar-fill" style="width:${riskPct}%; background:var(--${riskColor==='amber'?'amber':riskColor});"></div></div>
            </div>
            <div class="card stat-card">
              <div class="stat-label">${ICONS.zap} Trend Strength</div>
              <div class="trend-strength-wrap">
                <div class="trend-bars">
                  ${[1,2,3,4,5].map(n=>`<span class="${n<=result.trendStrength?'on':''}" style="height:${n*16}%"></span>`).join('')}
                </div>
              </div>
            </div>
            <div class="card stat-card">
              <div class="stat-label">${ICONS.gauge} Confidence</div>
              <div class="stat-val ${result.confidence>=80?'green':result.confidence>=60?'amber':'red'}">${result.confidence}%</div>
            </div>
          </div>

          <div class="card summary-card">
            <h3>${ICONS.brain} Technical Analysis Summary</h3>
            <p>${esc(result.summary)}</p>
          </div>

          <div class="card summary-card">
            <h3>${ICONS.bolt} AI-Generated Market Insights</h3>
            <ul class="insight-list">
              ${result.insights.map(ins => `<li>${ICONS.check}<span>${esc(ins)}</span></li>`).join('')}
            </ul>
          </div>
        </div>

        <div>
          <div class="chart-side-img">
            <img src="${result.image}" alt="Uploaded chart">
          </div>

          <div class="card levels-card">
            <h3>${ICONS.target} Suggested Trade Levels</h3>
            <div class="level-row">
              <div class="level-left"><span class="level-dot entry"></span> Entry Point</div>
              <div class="level-val mono">$${result.entry}</div>
            </div>
            <div class="level-row">
              <div class="level-left"><span class="level-dot tp"></span> Take Profit</div>
              <div class="level-val mono" style="color:var(--green);">$${result.tp}</div>
            </div>
            <div class="level-row">
              <div class="level-left"><span class="level-dot sl"></span> Stop Loss</div>
              <div class="level-val mono" style="color:var(--red);">$${result.sl}</div>
            </div>
          </div>

          <div class="card" style="margin-top:20px; padding:20px;">
            <a href="#/upload" data-link class="btn btn-outline-green btn-block">${ICONS.upload} Analyze Another Chart</a>
          </div>
        </div>

      </div>
    </div>
  </div>`;
}

/* =========================================================
   PAGE: MARKET OVERVIEW
   ========================================================= */
let _currentMarketTab = 'crypto';
function pageMarkets(){
  const cats = [
    { key:'crypto', label:'Cryptocurrency' },
    { key:'forex', label:'Forex' },
    { key:'stocks', label:'Stocks & Indices' },
    { key:'commodities', label:'Commodities' },
  ];
  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / Market Overview</div>
        <h1>Market Overview</h1>
        <p>Live-style snapshot across crypto, forex, equities, and commodities. Tap any category to filter.</p>
      </div>
    </div>
    <div class="container" style="padding:40px 28px 96px;">
      <div class="market-tabs" id="marketTabs">
        ${cats.map(c => `<button class="market-tab ${c.key===_currentMarketTab?'active':''}" data-cat="${c.key}">${c.label}</button>`).join('')}
      </div>
      <div class="card asset-table-wrap" id="marketTableWrap"></div>
    </div>
  </div>`;
}

function renderMarketTable(){
  const wrap = document.getElementById('marketTableWrap');
  if(!wrap) return;
  const list = MARKET_DATA[_currentMarketTab];
  wrap.innerHTML = `
    <table class="asset-table">
      <thead><tr><th>Asset</th><th>Price</th><th>24h Change</th><th>Trend</th></tr></thead>
      <tbody>
        ${list.map(a => `
          <tr>
            <td>
              <div class="asset-name">
                <div class="asset-icon" style="background:${a.color}22; color:${a.color}; border:1px solid ${a.color}55;">${a.icon}</div>
                <div>
                  <div class="asset-title">${a.sym}</div>
                  <div class="asset-sub">${a.name}</div>
                </div>
              </div>
            </td>
            <td class="asset-price mono">${a.cat==='forex' || a.cat==='commodities' ? a.price.toFixed(4) : a.price.toLocaleString(undefined,{maximumFractionDigits:2})}</td>
            <td><span class="asset-chg ${a.chg>=0?'up':'down'}">${a.chg>=0?ICONS.arrowUp:ICONS.arrowDown} ${Math.abs(a.chg).toFixed(2)}%</span></td>
            <td>${sparkSvg(a.chg>=0)}</td>
          </tr>
        `).join('')}
      </tbody>
    </table>
  `;
}

function initMarketsPage(){
  renderMarketTable();
  document.querySelectorAll('.market-tab').forEach(btn=>{
    btn.addEventListener('click', ()=>{
      _currentMarketTab = btn.dataset.cat;
      document.querySelectorAll('.market-tab').forEach(b=>b.classList.remove('active'));
      btn.classList.add('active');
      renderMarketTable();
    });
  });
}

/* =========================================================
   PAGE: PRICING
   ========================================================= */
let _pricingAnnual = false;
function pagePricing(){
  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / Pricing</div>
        <h1>Simple, transparent pricing</h1>
        <p>Start free. Upgrade when you want deeper history, higher upload limits, and priority analysis.</p>
      </div>
    </div>
    <div class="container" style="padding:40px 28px 96px;">
      <div class="pricing-toggle">
        <span style="font-size:14px; color:var(--text-dim);">Monthly</span>
        <div class="switch" id="pricingSwitch"><div class="knob"></div></div>
        <span style="font-size:14px; color:var(--text-dim);">Annual</span>
        <span class="save-pill">Save 20%</span>
      </div>

      <div class="grid-3" id="planGrid"></div>

      <div class="card" style="margin-top:40px; padding:28px;">
        <h3 style="font-size:16px; margin-bottom:14px;">All plans include</h3>
        <div class="grid-3" style="gap:14px;">
          <div class="tip-row">${ICONS.check}<span>Buy / Sell / Hold signal on every chart</span></div>
          <div class="tip-row">${ICONS.check}<span>Confidence score &amp; risk classification</span></div>
          <div class="tip-row">${ICONS.check}<span>Entry, take-profit &amp; stop-loss levels</span></div>
        </div>
      </div>
    </div>
  </div>`;
}

function planData(annual){
  const mult = annual ? 0.8 : 1;
  const per = annual ? '/mo, billed yearly' : '/mo';
  return [
    { name:'Free', price:0, per, desc:'Try AI chart analysis with no commitment.', cta:'Get Started', variant:'btn-ghost',
      feats:[ ['5 analyses per month',true], ['Buy/Sell/Hold signal',true], ['Confidence score',true], ['Analysis history',false], ['Priority processing',false] ] },
    { name:'Pro', price:Math.round(19*mult), per, desc:'For active traders who analyze daily.', cta:'Start Pro Trial', variant:'btn-primary', featured:true,
      feats:[ ['Unlimited analyses',true], ['Full trade levels (TP/SL)',true], ['30-day analysis history',true], ['Priority processing speed',true], ['API access',false] ] },
    { name:'Premium', price:Math.round(49*mult), per, desc:'For desks and serious portfolio management.', cta:'Go Premium', variant:'btn-ghost',
      feats:[ ['Everything in Pro',true], ['Unlimited analysis history',true], ['Advanced risk analytics',true], ['API access',true], ['Dedicated support',true] ] },
  ];
}

function renderPlans(){
  const grid = document.getElementById('planGrid');
  if(!grid) return;
  const plans = planData(_pricingAnnual);
  grid.innerHTML = plans.map(p => `
    <div class="card plan-card ${p.featured?'featured':''}">
      ${p.featured ? '<span class="plan-badge">MOST POPULAR</span>' : ''}
      <div class="plan-name">${p.name}</div>
      <div class="plan-price">$${p.price}<span>${p.per}</span></div>
      <p class="plan-desc">${p.desc}</p>
      <ul class="plan-feats">
        ${p.feats.map(([label,on]) => `<li class="${on?'':'off'}">${on?ICONS.check:ICONS.x}<span>${label}</span></li>`).join('')}
      </ul>
      <a href="#/register" data-link class="btn ${p.variant} btn-block">${p.cta}</a>
    </div>
  `).join('');
}

function initPricingPage(){
  renderPlans();
  const sw = document.getElementById('pricingSwitch');
  sw.addEventListener('click', ()=>{
    _pricingAnnual = !_pricingAnnual;
    sw.classList.toggle('on', _pricingAnnual);
    renderPlans();
  });
}

/* =========================================================
   PAGE: ABOUT
   ========================================================= */
function pageAbout(){
  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / About Us</div>
        <h1>Built by traders, for traders</h1>
        <p>SignalForge exists to make chart reading faster — not to replace your judgment.</p>
      </div>
    </div>

    <section class="section">
      <div class="container">
        <div class="content-cols">
          <div>
            <h2 style="font-size:28px; margin-bottom:16px;">Our mission</h2>
            <p style="margin-bottom:14px;">Most traders spend hours scanning charts for the same handful of signals: trend direction, key levels, and momentum shifts. We built SignalForge to compress that process into seconds, using AI trained on candlestick structure and price-action patterns.</p>
            <p>We're not here to tell you what to do with your capital. We're here to give you a fast, structured second opinion — the kind a desk analyst might give you before market open.</p>
          </div>
          <div class="card" style="padding:0; overflow:hidden;">
            <div class="scan-chart-wrap" style="height:280px; background:linear-gradient(180deg,#101826,#0B111B);">
              <div class="scan-beam"></div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section class="section" style="padding-top:0;">
      <div class="container">
        <div class="section-head">
          <h2>What we stand for</h2>
        </div>
        <div class="card" style="padding:8px 28px;">
          <div class="value-row">
            <div class="value-num">01</div>
            <div><h4 style="font-size:15.5px; margin-bottom:4px;">Clarity over hype</h4><p>Every signal comes with the reasoning behind it — trend, structure, and risk — not just an arrow.</p></div>
          </div>
          <div class="value-row">
            <div class="value-num">02</div>
            <div><h4 style="font-size:15.5px; margin-bottom:4px;">You stay in control</h4><p>Our output is a decision-support tool. We never execute trades or manage funds on your behalf.</p></div>
          </div>
          <div class="value-row">
            <div class="value-num">03</div>
            <div><h4 style="font-size:15.5px; margin-bottom:4px;">Risk awareness first</h4><p>Confidence scores and risk levels are shown alongside every signal, never hidden behind the headline call.</p></div>
          </div>
        </div>
      </div>
    </section>

    <section class="section" style="padding-top:0;">
      <div class="container">
        <div class="section-head">
          <h2>Team</h2>
        </div>
        <div class="team-grid">
          <div class="card team-card"><div class="team-avatar">MR</div><h4>Maya Reyes</h4><div class="role">Co-Founder, CEO</div></div>
          <div class="card team-card"><div class="team-avatar">DK</div><h4>Daniel Kessler</h4><div class="role">Co-Founder, CTO</div></div>
          <div class="card team-card"><div class="team-avatar">AO</div><h4>Amara Osei</h4><div class="role">Head of ML Research</div></div>
          <div class="card team-card"><div class="team-avatar">JT</div><h4>Jonas Tran</h4><div class="role">Head of Product</div></div>
        </div>
      </div>
    </section>
  </div>`;
}

/* =========================================================
   PAGE: CONTACT
   ========================================================= */
function pageContact(){
  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / Contact Us</div>
        <h1>Get in touch</h1>
        <p>Questions about a signal, your subscription, or a partnership? We read everything that comes in.</p>
      </div>
    </div>

    <div class="container" style="padding:40px 28px 96px;">
      <div class="contact-grid">
        <div>
          <div class="card contact-info-card">
            <div class="ci-icon">${ICONS.mail}</div>
            <div><h4>Email support</h4><p>support@signalforge.ai — replies within 24 hours</p></div>
          </div>
          <div class="card contact-info-card">
            <div class="ci-icon">${ICONS.phone}</div>
            <div><h4>Phone</h4><p>+1 (415) 555-0148 — Mon–Fri, 9am–6pm ET</p></div>
          </div>
          <div class="card contact-info-card">
            <div class="ci-icon">${ICONS.pin}</div>
            <div><h4>Office</h4><p>548 Market Street, Suite 211, San Francisco, CA 94104</p></div>
          </div>
        </div>

        <div class="card" style="padding:28px;">
          <h3 style="font-size:17px; margin-bottom:20px;">Send us a message</h3>
          <form id="contactForm">
            <div class="form-group"><label>Full name</label><input class="form-input" type="text" placeholder="Jane Carter" required></div>
            <div class="form-group"><label>Email address</label><input class="form-input" type="email" placeholder="jane@example.com" required></div>
            <div class="form-group"><label>Message</label><textarea class="form-input" rows="4" placeholder="How can we help?" required></textarea></div>
            <button type="submit" class="btn btn-primary btn-block">Send Message</button>
          </form>
        </div>
      </div>
    </div>
  </div>`;
}

function initContactPage(){
  const form = document.getElementById('contactForm');
  if(!form) return;
  form.addEventListener('submit', e=>{
    e.preventDefault();
    toast("Message sent! We'll get back to you within 24 hours.");
    form.reset();
  });
}

/* =========================================================
   PAGE: LOGIN
   ========================================================= */
function pageLogin(){
  return `
  <div class="page">
    <div class="auth-shell">
      <div class="card auth-card">
        <div class="auth-head">
          <a href="#/" data-link class="nav-logo"><svg width="24" height="24" viewBox="0 0 28 28" fill="none"><path d="M3 21L9 13L14 17L24 5" stroke="#16D38F" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/><path d="M18 5H24V11" stroke="#16D38F" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/></svg><span>SignalForge</span></a>
          <h1>Welcome back</h1>
          <p>Log in to view your saved analyses and history.</p>
        </div>
        <form id="loginForm">
          <div class="form-group" id="loginEmailGroup">
            <label>Email address</label>
            <input class="form-input" type="email" id="loginEmail" placeholder="you@example.com">
            <div class="field-error">Enter a valid email address.</div>
          </div>
          <div class="form-group" id="loginPassGroup">
            <label>Password</label>
            <input class="form-input" type="password" id="loginPassword" placeholder="••••••••">
            <div class="field-error">Incorrect email or password.</div>
          </div>
          <div class="form-row-between">
            <label class="check-row"><input type="checkbox"> Remember me</label>
            <a href="#/forgot-password" data-link>Forgot password?</a>
          </div>
          <button type="submit" class="btn btn-primary btn-block btn-lg">Log In</button>
        </form>
        <div class="auth-divider">or</div>
        <button class="btn btn-ghost btn-block" id="demoLoginBtn">Continue with demo account</button>
        <div class="auth-foot">Don't have an account? <a href="#/register" data-link>Sign up free</a></div>
      </div>
    </div>
  </div>`;
}

function initLoginPage(){
  const form = document.getElementById('loginForm');
  const emailGroup = document.getElementById('loginEmailGroup');
  const passGroup = document.getElementById('loginPassGroup');

  document.getElementById('demoLoginBtn').addEventListener('click', ()=>{
    const users = getUsers();
    if(!users['demo@signalforge.ai']){
      users['demo@signalforge.ai'] = { name:'Demo Trader', email:'demo@signalforge.ai', password:'demo1234', plan:'Pro', joined: Date.now() };
      saveUsers(users);
    }
    setSession('demo@signalforge.ai');
    toast('Logged in as Demo Trader');
    renderNavActions();
    navigate('#/profile');
  });

  form.addEventListener('submit', e=>{
    e.preventDefault();
    emailGroup.classList.remove('invalid'); passGroup.classList.remove('invalid');
    const email = document.getElementById('loginEmail').value.trim().toLowerCase();
    const pass = document.getElementById('loginPassword').value;
    if(!/^\S+@\S+\.\S+$/.test(email)){ emailGroup.classList.add('invalid'); return; }
    const users = getUsers();
    const u = users[email];
    if(!u || u.password !== pass){ passGroup.classList.add('invalid'); return; }
    setSession(email);
    toast(`Welcome back, ${u.name}!`);
    renderNavActions();
    navigate('#/profile');
  });
}

/* =========================================================
   PAGE: REGISTER
   ========================================================= */
function pageRegister(){
  return `
  <div class="page">
    <div class="auth-shell">
      <div class="card auth-card">
        <div class="auth-head">
          <a href="#/" data-link class="nav-logo"><svg width="24" height="24" viewBox="0 0 28 28" fill="none"><path d="M3 21L9 13L14 17L24 5" stroke="#16D38F" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/><path d="M18 5H24V11" stroke="#16D38F" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/></svg><span>SignalForge</span></a>
          <h1>Create your account</h1>
          <p>Save your analysis history and unlock more uploads.</p>
        </div>
        <form id="registerForm">
          <div class="form-group" id="regNameGroup">
            <label>Full name</label>
            <input class="form-input" type="text" id="regName" placeholder="Jane Carter">
            <div class="field-error">Please enter your name.</div>
          </div>
          <div class="form-group" id="regEmailGroup">
            <label>Email address</label>
            <input class="form-input" type="email" id="regEmail" placeholder="you@example.com">
            <div class="field-error">This email is already registered.</div>
          </div>
          <div class="form-group" id="regPassGroup">
            <label>Password</label>
            <input class="form-input" type="password" id="regPassword" placeholder="At least 8 characters">
            <div class="field-error">Password must be at least 8 characters.</div>
          </div>
          <div class="form-group">
            <label class="check-row"><input type="checkbox" id="regTerms"> I agree to the <a href="#/legal/terms" data-link style="color:var(--green);">Terms</a> &amp; <a href="#/legal/privacy" data-link style="color:var(--green);">Privacy Policy</a></label>
          </div>
          <button type="submit" class="btn btn-primary btn-block btn-lg">Create Account</button>
        </form>
        <div class="auth-foot">Already have an account? <a href="#/login" data-link>Log in</a></div>
      </div>
    </div>
  </div>`;
}

function initRegisterPage(){
  const form = document.getElementById('registerForm');
  form.addEventListener('submit', e=>{
    e.preventDefault();
    ['regNameGroup','regEmailGroup','regPassGroup'].forEach(id=>document.getElementById(id).classList.remove('invalid'));

    const name = document.getElementById('regName').value.trim();
    const email = document.getElementById('regEmail').value.trim().toLowerCase();
    const pass = document.getElementById('regPassword').value;
    const terms = document.getElementById('regTerms').checked;

    let hasError = false;
    if(!name){ document.getElementById('regNameGroup').classList.add('invalid'); hasError = true; }
    const users = getUsers();
    if(!/^\S+@\S+\.\S+$/.test(email) || users[email]){ document.getElementById('regEmailGroup').classList.add('invalid'); hasError = true; }
    if(pass.length < 8){ document.getElementById('regPassGroup').classList.add('invalid'); hasError = true; }
    if(hasError) return;
    if(!terms){ toast('Please agree to the Terms & Privacy Policy.', 'error'); return; }

    users[email] = { name, email, password:pass, plan:'Free', joined: Date.now() };
    saveUsers(users);
    setSession(email);
    toast(`Welcome to SignalForge, ${name}!`);
    renderNavActions();
    navigate('#/profile');
  });
}

/* =========================================================
   PAGE: FORGOT PASSWORD
   ========================================================= */
function pageForgotPassword(){
  return `
  <div class="page">
    <div class="auth-shell">
      <div class="card auth-card" id="forgotCard">
        <div class="auth-head">
          <a href="#/" data-link class="nav-logo"><svg width="24" height="24" viewBox="0 0 28 28" fill="none"><path d="M3 21L9 13L14 17L24 5" stroke="#16D38F" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/><path d="M18 5H24V11" stroke="#16D38F" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/></svg><span>SignalForge</span></a>
          <h1>Reset your password</h1>
          <p>Enter the email on your account and we'll send a reset link.</p>
        </div>
        <form id="forgotForm">
          <div class="form-group">
            <label>Email address</label>
            <input class="form-input" type="email" id="forgotEmail" placeholder="you@example.com" required>
          </div>
          <button type="submit" class="btn btn-primary btn-block btn-lg">${ICONS.mail} Send Reset Link</button>
        </form>
        <div class="auth-foot">Remembered it? <a href="#/login" data-link>Back to log in</a></div>
      </div>
    </div>
  </div>`;
}

function initForgotPage(){
  const form = document.getElementById('forgotForm');
  form.addEventListener('submit', e=>{
    e.preventDefault();
    const card = document.getElementById('forgotCard');
    const email = document.getElementById('forgotEmail').value.trim();
    card.innerHTML = `
      <div class="auth-head">
        <div class="feature-icon" style="margin:0 auto 18px;">${ICONS.check}</div>
        <h1>Check your inbox</h1>
        <p>If an account exists for <strong style="color:var(--text);">${esc(email)}</strong>, a password reset link is on its way.</p>
      </div>
      <a href="#/login" data-link class="btn btn-primary btn-block btn-lg">Back to Log In</a>
    `;
  });
}

/* =========================================================
   PAGE: USER PROFILE
   ========================================================= */
let _profileTab = 'history';
function pageProfile(){
  const user = currentUser();
  if(!user){
    return `
    <div class="page">
      <div class="container" style="padding:90px 28px;">
        <div class="card" style="max-width:480px; margin:0 auto; text-align:center; padding:48px;">
          <div class="feature-icon" style="margin:0 auto 18px;">${ICONS.lock}</div>
          <h2 style="font-size:22px; margin-bottom:10px;">Log in to view your profile</h2>
          <p style="margin-bottom:24px;">Create an account or log in to see your saved analysis history.</p>
          <div style="display:flex; gap:12px; justify-content:center;">
            <a href="#/login" data-link class="btn btn-primary">Log In</a>
            <a href="#/register" data-link class="btn btn-ghost">Sign Up</a>
          </div>
        </div>
      </div>
    </div>`;
  }

  const history = getHistory(user.email);
  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / Profile</div>
        <h1>Your Account</h1>
        <p>Manage your profile, plan, and review your analysis history.</p>
      </div>
    </div>
    <div class="container" style="padding:40px 28px 96px;">
      <div class="profile-grid">
        <div class="card profile-card">
          <div class="avatar-circle">${initials(user.name)}</div>
          <h3>${esc(user.name)}</h3>
          <div class="p-email">${esc(user.email)}</div>
          <div class="plan-chip">${ICONS.credit} ${user.plan} Plan</div>
          <div class="profile-nav">
            <button data-tab="history" class="${_profileTab==='history'?'active':''}">${ICONS.history} Analysis History</button>
            <button data-tab="settings" class="${_profileTab==='settings'?'active':''}">${ICONS.settings} Account Settings</button>
            <button data-tab="billing" class="${_profileTab==='billing'?'active':''}">${ICONS.credit} Plan &amp; Billing</button>
            <button id="logoutBtn" style="color:var(--red); margin-top:10px;">${ICONS.logout} Log Out</button>
          </div>
        </div>

        <div id="profileContent"></div>
      </div>
    </div>
  </div>`;
}

function renderProfileContent(){
  const content = document.getElementById('profileContent');
  if(!content) return;
  const user = currentUser();
  if(!user) return;
  const history = getHistory(user.email);

  if(_profileTab === 'history'){
    content.innerHTML = `
      <div class="profile-stats-row">
        <div class="card stat-card"><div class="stat-label">Total Analyses</div><div class="stat-val">${history.length}</div></div>
        <div class="card stat-card"><div class="stat-label">Buy Signals</div><div class="stat-val green">${history.filter(h=>h.signal==='BUY').length}</div></div>
        <div class="card stat-card"><div class="stat-label">Sell Signals</div><div class="stat-val red">${history.filter(h=>h.signal==='SELL').length}</div></div>
      </div>
      <div class="card" style="padding:24px;">
        <h3 style="font-size:16px; margin-bottom:18px;">Recent analyses</h3>
        ${history.length === 0 ? `
          <div class="history-empty">
            <div class="feature-icon" style="margin:0 auto 16px;">${ICONS.chart}</div>
            <p style="margin-bottom:18px;">No analyses yet. Upload your first chart to get started.</p>
            <a href="#/upload" data-link class="btn btn-primary">${ICONS.upload} Upload a Chart</a>
          </div>
        ` : `
          <div class="history-list">
            ${history.map(h => `
              <div class="history-item">
                <img class="history-thumb" src="${h.image}" alt="${esc(h.asset)} chart">
                <div class="history-info">
                  <div class="h-top">
                    <span class="h-asset">${esc(h.asset)}</span>
                    <span class="history-badge ${h.signal.toLowerCase()}">${h.signal}</span>
                  </div>
                  <div class="h-date">${fmtDate(h.timestamp)} · ${h.confidence}% confidence · ${h.riskLevel} risk</div>
                </div>
              </div>
            `).join('')}
          </div>
        `}
      </div>
    `;
  } else if(_profileTab === 'settings'){
    content.innerHTML = `
      <div class="card" style="padding:28px;">
        <h3 style="font-size:16px; margin-bottom:20px;">Account settings</h3>
        <div class="form-group"><label>Full name</label><input class="form-input" value="${esc(user.name)}" id="settingsName"></div>
        <div class="form-group"><label>Email address</label><input class="form-input" value="${esc(user.email)}" disabled style="opacity:0.6;"></div>
        <button class="btn btn-primary" id="saveSettingsBtn">Save Changes</button>
      </div>
    `;
    document.getElementById('saveSettingsBtn').addEventListener('click', ()=>{
      const users = getUsers();
      users[user.email].name = document.getElementById('settingsName').value.trim() || user.name;
      saveUsers(users);
      toast('Profile updated.');
      renderApp();
    });
  } else if(_profileTab === 'billing'){
    const plans = ['Free','Pro','Premium'];
    content.innerHTML = `
      <div class="card" style="padding:28px;">
        <h3 style="font-size:16px; margin-bottom:6px;">Current plan</h3>
        <p style="margin-bottom:20px;">You're currently on the <strong style="color:var(--green);">${user.plan}</strong> plan.</p>
        <div class="grid-3" style="gap:14px;">
          ${plans.map(p => `
            <div class="card" style="padding:18px; ${p===user.plan?'border-color:var(--green-line);':''}">
              <div style="font-weight:700; margin-bottom:8px;">${p}</div>
              <button class="btn ${p===user.plan?'btn-outline-green':'btn-ghost'} btn-sm btn-block" data-plan="${p}" ${p===user.plan?'disabled':''}>${p===user.plan?'Current Plan':'Switch to '+p}</button>
            </div>
          `).join('')}
        </div>
      </div>
    `;
    content.querySelectorAll('[data-plan]').forEach(btn=>{
      btn.addEventListener('click', ()=>{
        const users = getUsers();
        users[user.email].plan = btn.dataset.plan;
        saveUsers(users);
        toast(`Switched to ${btn.dataset.plan} plan.`);
        renderApp();
      });
    });
  }
}

function initProfilePage(){
  const user = currentUser();
  if(!user) return;
  document.querySelectorAll('.profile-nav [data-tab]').forEach(btn=>{
    btn.addEventListener('click', ()=>{
      _profileTab = btn.dataset.tab;
      document.querySelectorAll('.profile-nav [data-tab]').forEach(b=>b.classList.remove('active'));
      btn.classList.add('active');
      renderProfileContent();
    });
  });
  const logoutBtn = document.getElementById('logoutBtn');
  if(logoutBtn) logoutBtn.addEventListener('click', ()=>{
    clearSession();
    toast('Logged out successfully.');
    renderNavActions();
    navigate('#/');
  });
  renderProfileContent();
}

/* =========================================================
   PAGE: LEGAL (Privacy / Terms / Risk)
   ========================================================= */
function pageLegal(slug){
  const content = {
    privacy: {
      title:'Privacy Policy',
      body:`
        <p>SignalForge ("we", "us") respects your privacy. This policy explains what we collect and how it's used.</p>
        <h4>Information we collect</h4>
        <p>Account details (name, email), uploaded chart images for the purpose of generating analysis, and basic usage data to improve the product.</p>
        <h4>How we use it</h4>
        <p>To provide AI analysis results, maintain your saved history if you're registered, and communicate important account updates.</p>
        <h4>Image handling</h4>
        <p>Uploaded charts are processed to generate your analysis. If you're logged in, you control which analyses remain in your saved history and can delete them at any time.</p>
        <h4>Your choices</h4>
        <p>You can request account deletion or data export at any time by contacting support@signalforge.ai.</p>
      `
    },
    terms: {
      title:'Terms & Conditions',
      body:`
        <p>By using SignalForge, you agree to the following terms.</p>
        <h4>Service description</h4>
        <p>SignalForge provides AI-generated educational commentary on uploaded chart images, including a directional signal, confidence score, and suggested trade levels.</p>
        <h4>Not financial advice</h4>
        <p>Nothing produced by SignalForge constitutes financial, investment, or trading advice. You are solely responsible for any trading decisions you make.</p>
        <h4>Acceptable use</h4>
        <p>You agree not to upload content you don't have rights to, attempt to reverse-engineer the analysis model, or use the service for unlawful purposes.</p>
        <h4>Subscription terms</h4>
        <p>Paid plans renew automatically until cancelled. You can cancel anytime from your account settings; access continues until the end of the billing period.</p>
      `
    },
    risk: {
      title:'Risk Disclaimer',
      body:`
        <p><strong>Trading involves substantial risk of loss.</strong> SignalForge's AI-generated signals — Buy, Sell, or Hold — are educational outputs based on pattern recognition applied to an uploaded image. They are not guarantees of future price movement.</p>
        <h4>No guarantee of accuracy</h4>
        <p>Confidence scores reflect the model's internal certainty given the chart provided, not the statistical probability of a profitable outcome.</p>
        <h4>Leverage and volatility</h4>
        <p>Forex, crypto, and CFD products are frequently traded with leverage and can result in losses exceeding your initial deposit.</p>
        <h4>Do your own research</h4>
        <p>Always verify any signal against your own analysis and risk tolerance, and consult a licensed financial advisor before making trading decisions.</p>
      `
    }
  };
  const c = content[slug] || content.privacy;
  return `
  <div class="page">
    <div class="page-head">
      <div class="container">
        <div class="breadcrumb"><a href="#/" data-link>Home</a> / ${c.title}</div>
        <h1>${c.title}</h1>
        <p>Last updated June 2026.</p>
      </div>
    </div>
    <div class="container" style="padding:40px 28px 96px;">
      <div class="card" style="padding:36px; max-width:760px; margin:0 auto;">
        <div style="display:flex; flex-direction:column; gap:16px;">${c.body}</div>
      </div>
    </div>
  </div>`;
}

/* =========================================================
   ROUTER
   ========================================================= */
const routes = {
  '/': { render: pageHome, init: () => { renderTicker(); } },
  '/upload': { render: pageUpload, init: initUploadPage },
  '/dashboard': { render: pageDashboard, init: ()=>{} },
  '/markets': { render: pageMarkets, init: initMarketsPage },
  '/pricing': { render: pagePricing, init: initPricingPage },
  '/about': { render: pageAbout, init: ()=>{} },
  '/contact': { render: pageContact, init: initContactPage },
  '/profile': { render: pageProfile, init: initProfilePage },
  '/login': { render: pageLogin, init: initLoginPage },
  '/register': { render: pageRegister, init: initRegisterPage },
  '/forgot-password': { render: pageForgotPassword, init: initForgotPage },
};

function parseHash(){
  let hash = location.hash || '#/';
  hash = hash.replace(/^#/, '');
  if(!hash.startsWith('/')) hash = '/' + hash;
  return hash;
}

function renderApp(){
  const app = document.getElementById('app');
  const path = parseHash();

  // legal sub-routes: /legal/privacy, /legal/terms, /legal/risk
  if(path.startsWith('/legal/')){
    const slug = path.split('/')[2] || 'privacy';
    app.innerHTML = pageLegal(slug);
    window.scrollTo(0,0);
    updateActiveNav(path);
    return;
  }

  const route = routes[path] || routes['/'];
  app.innerHTML = route.render();
  window.scrollTo(0,0);
  updateActiveNav(path);
  closeMobileMenu();
  try{ route.init(); }catch(e){ console.error('Route init error:', e); }
}

function navigate(hash){
  if(location.hash === hash){
    renderApp();
  } else {
    location.hash = hash;
  }
}

function updateActiveNav(path){
  document.querySelectorAll('.nav-links a[data-link]').forEach(a=>{
    const href = a.getAttribute('href').replace(/^#/, '');
    a.classList.toggle('active', href === path);
  });
}

function closeMobileMenu(){
  const links = document.getElementById('navLinks');
  if(links) links.classList.remove('open');
}

/* ---------- Nav actions (auth-aware) ---------- */
function renderNavActions(){
  const actions = document.getElementById('navActions');
  const user = currentUser();
  if(user){
    actions.innerHTML = `
      <a href="#/upload" data-link class="btn btn-primary btn-sm">${ICONS.upload} Upload</a>
      <a href="#/profile" data-link class="btn btn-ghost btn-sm" style="display:flex; align-items:center; gap:8px;">
        <span class="avatar-circle" style="width:26px; height:26px; font-size:11px; margin:0;">${initials(user.name)}</span>
        ${user.name.split(' ')[0]}
      </a>
    `;
  } else {
    actions.innerHTML = `
      <a href="#/login" data-link class="btn btn-ghost btn-sm">Log In</a>
      <a href="#/register" data-link class="btn btn-primary btn-sm">Sign Up Free</a>
    `;
  }
}

/* ---------- Global link delegation (data-link anchors) ---------- */
document.addEventListener('click', (e)=>{
  const link = e.target.closest('[data-link]');
  if(link){
    // allow default hash navigation, but re-render if same hash
    const href = link.getAttribute('href');
    if(href && href.startsWith('#')){
      setTimeout(()=>{ if(parseHash() === href.replace(/^#/, '') || (href === '#/' && parseHash() === '/')) renderApp(); }, 0);
    }
  }
});

/* ---------- Mobile nav burger ---------- */
document.getElementById('navBurger').addEventListener('click', ()=>{
  document.getElementById('navLinks').classList.toggle('open');
});

/* ---------- Boot ---------- */
window.addEventListener('hashchange', renderApp);
window.addEventListener('DOMContentLoaded', ()=>{
  renderNavActions();
  renderApp();
});
// In case DOMContentLoaded already fired (script at end of body, should be fine, but safe-guard):
if(document.readyState !== 'loading'){
  renderNavActions();
  renderApp();
}
