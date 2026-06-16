import { useState, useEffect, useRef, useCallback } from "react";
// ─── AI ────────────────────────────────────────────────────────────────────
async function callAI(system, user) {
try {
const r = await fetch("https://api.anthropic.com/v1/messages", {
method: "POST",
headers: { "Content-Type": "application/json" },
body: JSON.stringify({ model: "claude-sonnet-4-20250514", max_tokens: 1200, system, mes
});
if (!r.ok) throw new Error("api");
const d = await r.json();
const t = d.content?.[0]?.text || "";
if (t) return t;
throw new Error("empty");
} catch { return generateMock(system, user); }
}
function generateMock(system, user) {
const u = user.toLowerCase();
if (system.includes('"trend":"steigend|fallend|seitwärts"')) {
const raw = user.replace(/analysiere:|kontext:.*$/gi,"").trim();
const asset = raw.charAt(0).toUpperCase()+raw.slice(1).split(" ")[0];
const isCrypto = ["btc","eth","sol","bitcoin","ethereum","solana","bnb"].some(w=>u.includ
const idx = asset.length%5;
const trends=["steigend","seitwärts","steigend","fallend","steigend"];
const simples=["positiv","neutral","positiv","negativ","positiv"];
const rsiVals=[58,44,62,38,55];
const rsi=rsiVals[idx];
return JSON.stringify({ asset, trend:trends[idx], simplerTrend:simples[idx], rsi,
ma_signal:simples[idx]==="positiv"?"über MA50":"unter MA50",
volumen_signal:idx%3===0?"hoch":idx%3===1?"normal":"niedrig",
kurzfazit:`${asset} zeigt einen ${trends[idx]}en Trend. RSI(14) bei ${rsi} — ${rsi>60?"
support:isCrypto?"Vormonats-Tief als Unterstützung":"200-Tage-MA als starker Support",
widerstand:simples[idx]==="positiv"?"Letztes Swing-High als Hürde":"MA50 als Widerstand
kurzfrist_ausblick:simples[idx]==="positiv"?`Fortsetzung des Aufwärtstrends wahrscheinl
}
if (system.includes('"trend":"bullish|bearish|neutral"') && system.includes("szenario_bull"
let rawA = "";
const m1=user.match(/f[uü]r:\s*["']?([A-Za-z0-9\/\-\.]+)/i);
const m2=user.match(/:\s*["']([^"'\n]+)/);
if (m1) rawA=m1[1].trim(); else if (m2) rawA=m2[1].trim();
else rawA=u.split(/\s+/).find(w=>w.length>=2&&/^[a-z0-9\/]+$/i.test(w))||"Asset";
const asset=rawA.charAt(0).toUpperCase()+rawA.slice(1).split(/[\s,\.]/)[0];
const isCrypto=["btc","eth","sol","bitcoin","ethereum","solana","bnb"].some(w=>u.includes
const seed=asset.length+asset.charCodeAt(0);
const rsi=38+(seed%35); const isBull=rsi>50;
const trend=isBull?"bullish":rsi<42?"bearish":"neutral";
const targetPct=isCrypto?`+${10+seed%12}–${18+seed%10}%`:`+${5+seed%6}–${9+seed%6}%`;
const stopPct=isCrypto?`${6+seed%5}%`:`${3+seed%4}%`;
const timeframe=isCrypto?`${2+seed%4}–${5+seed%5} Wochen`:`${4+seed%5}–${8+seed%5} Wochen
return JSON.stringify({ asset, trend, risiko:isCrypto?"hoch":"mittel",
technisch:`${asset} notiert ${isBull?"oberhalb":"unterhalb"} der 50-Tage-Linie. RSI(14)
volumen:`Volumen ${10+seed%20}% ${isBull?"über":"unter"} dem 30-Tage-Durchschnitt. ${is
marktKap:isCrypto?"Top-Krypto nach Marktkapitalisierung. Ausreichend Liquidität.":"Larg
szenario_bull:`Hält ${asset} den Support, ist ein Anstieg von ${targetPct} in ${timefra
szenario_bear:`Verlust des MA50-Supports würde Korrektur von ${8+seed%8}% einleiten.`,
empfehlung:`Strategie: (1) EINSTIEG bei Pullback von 3–5%. (2) POSITIONSGRÖSSE max. ${i
score:Math.min(9,Math.max(5,7+(isBull?1:-1)+(seed%3-1))) });
}
if (system.includes('"stimmung":"bullish|bearish|neutral"')) {
return JSON.stringify({ stimmung:"bullish", zusammenfassung:"Globale Märkte resilient. Fe
}
if (system.includes('"bewertung":"gut"')) {
// Parse assets from prompt string
const FUNDAMENTALS = {
AAPL: { iv:210, pe:28, deRatio:1.8, brand:9, desc:"Starke Marke, moderater Verschuldu
MSFT: { iv:380, pe:34, deRatio:0.4, brand:9, desc:"Niedrige Verschuldung, Azure-Wachs
NVDA: { iv:180, pe:38, deRatio:0.3, brand:8, desc:"KGV historisch hoch, Verschuldung
TSLA: { iv:150, pe:55, deRatio:0.8, brand:7, desc:"Hohe Bewertung, Margendruck" },
AMZN: { iv:190, pe:40, deRatio:0.7, brand:9, desc:"AWS-Wachstum intakt" },
GOOGL: { iv:160, pe:22, deRatio:0.1, brand:9, desc:"Sehr niedrig verschuldet, starke M
SPY: { iv:680, pe:21, deRatio:0, brand:10,desc:"Breit diversifiziert, günstige Bew
META: { iv:480, pe:25, deRatio:0.1, brand:8, desc:"Niedrig verschuldet" },
VWRL: { iv:110, pe:18, deRatio:0, brand:10,desc:"Globaler ETF, sehr kostengünstig"
};
const CRYPTO_DATA = {
BTC: { ath:108000, rsi:44, cycle:"post-halving Konsolidierung" },
ETH: { ath:4800, rsi:38, cycle:"70% unter ATH — akkumulierbar" },
SOL: { ath:260, rsi:35, cycle:"68% unter ATH — günstig" },
BNB: { ath:720, rsi:52, cycle:"regulatorisches Risiko" },
};
// Use hardcoded DEMO data for analysis (safe, no external deps)
const PORTFOLIO = [
{symbol:"BTC", amount:0.432, buyPrice:42000, currentPrice:76200, sector:"Crypto"},
{symbol:"ETH", amount:3.21, buyPrice:2200, currentPrice:2300, sector:"Crypto"},
{symbol:"SOL", amount:42, buyPrice:95, currentPrice:84.1, sector:"Crypto"},
{symbol:"AAPL",amount:12, buyPrice:210, currentPrice:270.5, sector:"Tech"},
{symbol:"MSFT",amount:8, buyPrice:380, currentPrice:425, sector:"Tech"},
{symbol:"NVDA",amount:15, buyPrice:145, currentPrice:210.5, sector:"Tech"},
{symbol:"SPY", amount:10, buyPrice:580, currentPrice:714, sector:"ETF"},
{symbol:"VWRL",amount:20, buyPrice:105, currentPrice:118.2, sector:"ETF"},
];
const assets = PORTFOLIO.map(a=>({...a, gain:((a.currentPrice-a.buyPrice)/a.buyPrice*100)
const suggestions = [];
// Aktien: Margin of Safety
assets.filter(a=>!CRYPTO_DATA[a.symbol]).forEach(a=>{
const f=FUNDAMENTALS[a.symbol]; if(!f) return;
const mos=((f.iv-a.currentPrice)/f.iv*100);
if(mos>=20){
suggestions.push({typ:"kaufen",asset:a.symbol,menge:"Aufstocken",kurs:`$${a.currentPr
ziel:`Innerer Wert $${f.iv} — MoS ${mos.toFixed(0)}% Sicherheitsmarge`,investition:
priorität:mos>=30?"hoch":"mittel",
grund:`${f.desc}. MoS = ($${f.iv}-$${a.currentPrice.toFixed(0)})/$${f.iv}×100 = ${m
} else if(mos<-20&&a.gain>30){
suggestions.push({typ:"reduzieren",asset:a.symbol,menge:"25-30%",kurs:`$${a.currentPr
ziel:`Kurs ${Math.abs(mos).toFixed(0)}% über innerem Wert $${f.iv}`,erlös:`~€${Math
priorität:Math.abs(mos)>40?"hoch":"mittel",
grund:`${f.desc}. Kurs übersteigt fairen Wert um ${Math.abs(mos).toFixed(0)}%. KGV
}
});
// Crypto: RSI + ATH-Abstand
assets.filter(a=>CRYPTO_DATA[a.symbol]).forEach(a=>{
const d=CRYPTO_DATA[a.symbol]; if(!d) return;
const fromATH=((a.currentPrice-d.ath)/d.ath*100);
if(d.rsi<40&&fromATH<-50){
suggestions.push({typ:"kaufen",asset:a.symbol,menge:"Position aufbauen",kurs:`$${a.cu
ziel:`${Math.abs(fromATH).toFixed(0)}% unter ATH — RSI ${d.rsi} überverkauft`,inves
priorität:d.rsi<35?"hoch":"mittel",
grund:`${d.cycle}. RSI(14) ${d.rsi} — überverkauft. ATH $${d.ath.toLocaleString("de
} else if(a.symbol==="BNB"||(d.rsi>55&&a.gain>40)){
suggestions.push({typ:"verkaufen",asset:a.symbol,menge:"50-75%",kurs:`$${a.currentPri
ziel:"Risiko reduzieren",erlös:`~€${Math.round(a.amount*a.currentPrice*0.6)}`,
priorität:a.symbol==="BNB"?"hoch":"mittel",
grund:a.symbol==="BNB"?"Regulatorisches Risiko: SEC-Verfahren gegen Binance. :`YTD +${a.gain.toFixed(0)}%. RSI ${d.rsi} — Teilgewinnmitnahme, Trailing-Stop au
Kapita
}
});
// Diversifikation
if(!assets.some(a=>["XGLD","GLD"].includes(a.symbol))){
suggestions.push({typ:"kaufen",asset:"Gold ETC (XGLD)",menge:"3-5% Portfolio",kurs:"~$3
ziel:"Inflationsschutz & negative Tech-Korrelation",investition:"~€600-800",priorität
grund:"Kein Gold-Exposure. Negative Korrelation zu Tech in Krisen. Absicherung gegen
}
if(!assets.some(a=>a.sector==="Bonds")){
suggestions.push({typ:"kaufen",asset:"Short-Duration Bonds ETF",menge:"5-8% Portfolio",
ziel:"4-5% Rendite bei minimalem Risiko",investition:"~€800-1.200",priorität:"niedrig
grund:"Keine Anleihen im Portfolio. Fed-Zinspause: kurzlaufende Staatsanleihen mit 4-
}
const cryptoPct=assets.filter(a=>CRYPTO_DATA[a.symbol]).reduce((s,a)=>s+a.amount*a.curren
const total=assets.reduce((s,a)=>s+a.amount*a.currentPrice,0)||1;
const cryptoRatio=cryptoPct/total*100;
const score=Math.max(4,Math.min(9,Math.round(7-(cryptoRatio>30?1:0)-(cryptoRatio>50?1:0))
return JSON.stringify({
score,
bewertung:score>=8?"Überdurchschnittlich":score>=6?"Solide":"Optimierungsbedarf",
stärken:[
`${assets.length} Positionen über mehrere Sektoren`,
"Starke Marken (AAPL, MSFT, NVDA) mit niedrigem Verschuldungsgrad",
`Top-Performer: ${[...assets].sort((a,b)=>b.gain-a.gain)[0]?.symbol} +${[...assets].s
],
risiken:[
cryptoRatio>25?`Crypto-Anteil ${cryptoRatio.toFixed(0)}% übersteigt empfohlene 20%`:"
assets.some(a=>FUNDAMENTALS[a.symbol]?.pe>35)?`Überbewertung: ${assets.filter(a=>FUND
"Kein Gold/Rohstoff-Exposure als Inflationsschutz",
],
vorschläge:suggestions.slice(0,6),
makro:"Fed hält Zinsen Mai 2026 bei 4.25-4.5%. Zinssenkung frühestens Q4 2026. EZB bei
geopolitik:"US-China Exportbeschränkungen belasten Semiconductors (NVDA). Nahost disclaimer:"Keine Anlageberatung. Berechnungen basieren auf Schätzwerten. Eigene });
stützt
Due Di
}
return JSON.stringify({ error:"Unbekannte Anfrage" });
}
// ─── Fallback prices & seed candles ────────────────────────────────────────
const FALLBACK = {
BTC:{price:76200,prevClose:77400,currency:"USD",open:77100,high:78200,low:75900,volume:31e9
ETH:{price:2300,prevClose:2370,currency:"USD",open:2355,high:2390,low:2270,volume:12e9},
SOL:{price:84.1,prevClose:86.2,currency:"USD",open:85.8,high:88,low:83.8,volume:4e9},
BNB:{price:623,prevClose:612,currency:"USD",open:614,high:628,low:610,volume:1.9e9},
AAPL:{price:270.5,prevClose:263.4,currency:"USD",open:264,high:273.2,low:263,volume:42e6},
MSFT:{price:425,prevClose:418.1,currency:"USD",open:419.5,high:429.9,low:418.1,volume:30e6}
NVDA:{price:210.5,prevClose:206.8,currency:"USD",open:207.5,high:214.2,low:206,volume:38e6}
SPY:{price:714,prevClose:707.5,currency:"USD",open:709,high:718.5,low:707,volume:68e6},
VWRL:{price:118.2,prevClose:116.8,currency:"USD",open:117,high:119.1,low:116.5,volume:980e3
};
function seedCandles(symbol, days) {
const B={BTC:76200,ETH:2300,SOL:84,BNB:623,AAPL:270.5,MSFT:425,NVDA:210,TSLA:180,SPY:714,VW
const base=B[symbol]||100, vol=["BTC","ETH","SOL","BNB"].includes(symbol)?0.024:0.010;
const seed=symbol.split("").reduce((a,b)=>a+b.charCodeAt(0),0);
let price=base*0.80; const pts=[], now=Date.now();
for (let i=days;i>=0;i--) {
const x=Math.sin(seed*9301+i*49297)*0.5+0.5, y=Math.sin(seed*4177+i*23171)*0.5+0.5;
price=Math.max(price*(1+(x-0.482)*vol+0.0003),base*0.35);
const hi=Math.max(price,price*(1+y*0.005)), lo=Math.min(price,price*(1-x*0.005));
pts.push({t:now-i*86400000,o:price*(1-(x-0.5)*0.002),h:hi,l:lo,c:price});
}
return pts;
}
async function fetchYFCandles(symbol, rangeKey) {
const YF={BTC:"BTC-USD",ETH:"ETH-USD",SOL:"SOL-USD",BNB:"BNB-USD",AAPL:"AAPL",MSFT:"MSFT",N
const RCFG={"1d":{interval:"5m",range:"1d"},"5d":{interval:"15m",range:"5d"},"1mo":{interva
const ticker=YF[symbol]||symbol, {interval,range}=RCFG[rangeKey]||RCFG["1mo"];
const url=`https://query1.finance.yahoo.com/v8/finance/chart/${ticker}?interval=${interval}
for (const proxy of [`https://corsproxy.io/?${encodeURIComponent(url)}`,`https://api.allori
try {
const ctrl=new AbortController(); const tid=setTimeout(()=>ctrl.abort(),5000);
const res=await fetch(proxy,{signal:ctrl.signal}); clearTimeout(tid);
const raw=await res.json();
const body=raw.contents?JSON.parse(raw.contents):raw;
const result=body?.chart?.result?.[0]; if (!result) continue;
const q=result.indicators.quote[0], ts=result.timestamp;
const pts=ts.map((t,i)=>({t:t*1000,o:q.open[i],h:q.high[i],l:q.low[i],c:q.close[i]})).f
if (pts.length>5) return pts;
} catch { continue; }
}
return null;
}
// ─── Live prices hook ──────────────────────────────────────────────────────
async function fetchMultiQuote(symbols) {
const results = await Promise.all(symbols.map(async s => {
const fb=FALLBACK[s]; if (!fb) return [s,null];
try {
const pts=await fetchYFCandles(s,"1d");
if (pts&&pts.length>1) {
const last=pts[pts.length-1];
return [s,{...fb,price:last.c,open:pts[0].o,high:Math.max(...pts.map(p=>p.h)),low:Mat
}
} catch {}
return [s,fb];
}));
return Object.fromEntries(results);
}
function useLivePrices(symbols) {
const key=symbols.join(",");
const [prices,setPrices]=useState(()=>{
const init={};
symbols.forEach(s=>{ if(FALLBACK[s]) init[s]={...FALLBACK[s],symbol:s}; });
return init;
});
const refresh=useCallback(async()=>{
if (!symbols.length) return;
const q=await fetchMultiQuote(symbols);
setPrices(prev=>{const n={...prev};symbols.forEach(s=>{if(q[s])n[s]=q[s];}); return n;});
},[key]);
useEffect(()=>{ refresh(); const id=setInterval(refresh,60000); return()=>clearInterval(id)
return {prices,refresh};
}
// ─── Utilities ─────────────────────────────────────────────────────────────
const fmt=(n,d=2)=>new Intl.NumberFormat("de-DE",{minimumFractionDigits:d,maximumFractionDigi
const fmtEur=n=>new Intl.NumberFormat("de-DE",{style:"currency",currency:"EUR",maximumFractio
// ─── Brand Logos ───────────────────────────────────────────────────────────
const COIN_LOGOS={BTC:"https://assets.coingecko.com/coins/images/1/small/bitcoin.png",ETH:"ht
const PLAT_LOGOS={binance:"https://assets.coingecko.com/markets/images/52/small/binance.jpg",
function AssetLogo({symbol,size=28}){
const [err,setErr]=useState(false);
const url=COIN_LOGOS[symbol];
const colors={BTC:"#F7931A",ETH:"#627EEA",SOL:"#9945FF",BNB:"#F0B90B",default:"#6B7280"};
const c=colors[symbol]||colors.default;
if (!err&&url) return <img src={url} alt={symbol} onError={()=>setErr(true)} style={{width:
return <div style={{width:size,height:size,borderRadius:"50%",background:c,display:"flex",a
}
function PlatBadge({platform,size=24}){
const [err,setErr]=useState(false);
const url=PLAT_LOGOS[platform];
const cfg={binance:{bg:"#F0B90B",l:"B"},coinbase:{bg:"#0052FF",l:"C"},etoro:{bg:"#2ECC71",l
if (!err&&url) return <div style={{width:size,height:size,borderRadius:6,overflow:"hidden",
return <div style={{width:size,height:size,borderRadius:6,background:cfg.bg,display:"flex",
}
// ─── Helix Dot Matrix ──────────────────────────────────────────────────────
function HelixDotMatrix({phase=0,color="#e8e8f0",dotSize=10,gap=8}){
const COLS=5,ROWS=5,STRAND=1.0,BRIDGE=0.58,NEAR=0.24,BASE=0.08;
const dots=[];
for(let row=0;row<ROWS;row++){
const rp=phase*2*2*Math.PI+row*1.24;
const left=Math.round(1+Math.sin(rp)),right=4-left;
const bridgeOn=Math.cos(rp*2)>0.82;
for(let col=0;col<COLS;col++){
let op=BASE;
if(col===left||col===right)op=STRAND;
else if(bridgeOn&&col>left&&col<right)op=BRIDGE;
else if(Math.abs(col-left)===1||Math.abs(col-right)===1)op=NEAR;
dots.push({row,col,op});
}
}
const w=COLS*(dotSize+gap)-gap, h=ROWS*(dotSize+gap)-gap;
return(
<svg width={w} height={h} viewBox={`0 0 ${w} ${h}`}>
{dots.map(({row,col,op})=><circle key={`${row}-${col}`} cx={col*(dotSize+gap)+dotSize/2
</svg>
);
}
function HelixLoader({label="Analysiere…",sublabel="",steps=[],currentStep=0,color="#e8e8f0"}
const [phase,setPhase]=useState(0);
const rafRef=useRef(null), startRef=useRef(null);
useEffect(()=>{
const animate=ts=>{ if(!startRef.current)startRef.current=ts; setPhase(((ts-startRef.curr
rafRef.current=requestAnimationFrame(animate);
return()=>cancelAnimationFrame(rafRef.current);
},[]);
return(
<div style={{padding:"36px 24px",display:"flex",flexDirection:"column",alignItems:"center
<HelixDotMatrix phase={phase} color={color}/>
<div style={{textAlign:"center"}}>
<div style={{fontWeight:700,fontSize:13,color:"var(--text)",fontFamily:"'JetBrains Mo
{sublabel&&<div style={{fontSize:11,color:"var(--text3)",marginTop:4,fontFamily:"'Jet
</div>
{steps.length>0&&(
<div style={{width:"100%",maxWidth:280,display:"flex",flexDirection:"column",gap:5}}>
{steps.map((s,i)=>(
<div key={i} style={{display:"flex",alignItems:"center",gap:10,padding:"8px 12px"
<div style={{width:18,height:18,borderRadius:4,flexShrink:0,display:"flex",alig
{currentStep>i?"✓":String(i+1).padStart(2,"0")}
</div>
<span style={{flex:1,fontFamily:"'Space Grotesk',sans-serif"}}>{s}</span>
{currentStep===i&&<div style={{width:12,height:12,border:"1.5px solid rgba(0,0,
</div>
))}
</div>
)}
</div>
);
}
ctx.te
// ─── Canvas Chart ──────────────────────────────────────────────────────────
function drawChart(canvas,data,chartType){
if(!canvas||!data||!data.length)return;
const dpr=window.devicePixelRatio||1, W=canvas.offsetWidth, H=canvas.offsetHeight;
if(!W||!H)return;
canvas.width=W*dpr; canvas.height=H*dpr;
const ctx=canvas.getContext("2d"); ctx.scale(dpr,dpr);
ctx.clearRect(0,0,W,H);
const P={t:18,b:30,l:6,r:66}, cw=W-P.l-P.r, ch=H-P.t-P.b;
const hi=Math.max(...data.map(d=>d.h))*1.006, lo=Math.min(...data.map(d=>d.l))*0.994, rng=h
const toX=i=>P.l+(i/(data.length-1))*cw, toY=v=>P.t+ch-((v-lo)/rng)*ch;
[0,0.25,0.5,0.75,1].forEach(r=>{
const v=lo+rng*r, y=toY(v);
ctx.strokeStyle="rgba(255,255,255,0.05)"; ctx.lineWidth=1;
ctx.beginPath(); ctx.moveTo(P.l,y); ctx.lineTo(W-P.r,y); ctx.stroke();
const lbl=v>=1000?`$${(v/1000).toFixed(v>=10000?0:1)}k`:`$${v.toFixed(v<10?2:0)}`;
ctx.fillStyle="rgba(255,255,255,0.35)"; ctx.font=`9px 'JetBrains Mono',monospace`; ctx.fillText(lbl,W-P.r+4,y+3);
});
const up=data[data.length-1].c>=data[0].o, G="#A8A8B3", R="#ef4444", col=up?G:R;
if(chartType==="line"){
const grad=ctx.createLinearGradient(0,P.t,0,P.t+ch);
grad.addColorStop(0,up?"rgba(34,197,94,0.20)":"rgba(239,68,68,0.20)"); grad.addColorStop(
ctx.beginPath(); data.forEach((d,i)=>{const x=toX(i),y=toY(d.c);i===0?ctx.moveTo(x,y):ctx
ctx.lineTo(toX(data.length-1),P.t+ch); ctx.lineTo(P.l,P.t+ch); ctx.closePath(); ctx.fillS
ctx.beginPath(); data.forEach((d,i)=>{const x=toX(i),y=toY(d.c);i===0?ctx.moveTo(x,y):ctx
ctx.strokeStyle=col; ctx.lineWidth=2; ctx.lineJoin="round"; ctx.lineCap="round"; ctx.stro
const lx=toX(data.length-1),ly=toY(data[data.length-1].c);
ctx.beginPath(); ctx.arc(lx,ly,3.5,0,Math.PI*2); ctx.fillStyle=col; ctx.fill();
const lv=data[data.length-1].c, lb=lv>=1000?`$${(lv/1000).toFixed(lv>=10000?0:2)}k`:`$${l
ctx.fillStyle=col; ctx.fillRect(W-P.r+1,ly-9,64,16);
ctx.fillStyle="#0a0a0b"; ctx.font=`bold 8px 'JetBrains Mono',monospace`; ctx.textAlign="l
} else {
const bw=Math.max(1,Math.min(10,cw/data.length-1));
data.forEach((d,i)=>{
if(!d.o)return; const x=toX(i), dc=d.c>=d.o?G:R;
ctx.strokeStyle=dc; ctx.lineWidth=1;
ctx.beginPath(); ctx.moveTo(x,toY(d.h)); ctx.lineTo(x,toY(d.l)); ctx.stroke();
const top=toY(Math.max(d.o,d.c)), bh=Math.max(1,toY(Math.min(d.o,d.c))-top);
ctx.fillStyle=dc; ctx.fillRect(x-bw/2,top,bw,bh);
});
}
const step=Math.max(1,Math.floor(data.length/5));
ctx.fillStyle="rgba(255,255,255,0.30)"; ctx.font=`8px 'JetBrains Mono',monospace`; ctx.text
for(let i=0;i<data.length;i+=step){
const d=new Date(data[i].t);
const lbl=data.length<=2?d.toLocaleTimeString("de-DE",{hour:"2-digit",minute:"2-digit"}):
ctx.fillText(lbl,toX(i),H-8);
}
}
function AssetChart({symbol,chartType,range}){
const canvasRef=useRef(null);
const days={"1d":1,"5d":5,"1mo":30,"3mo":90,"1y":365,"all":730}[range]||30;
const [data,setData]=useState(()=>seedCandles(symbol,days));
const [live,setLive]=useState(false);
useEffect(()=>{ setData(seedCandles(symbol,days)); setLive(false); fetchYFCandles(symbol,ra
useEffect(()=>{ if(canvasRef.current&&data.length) drawChart(canvasRef.current,data,chartTy
return(
<div style={{width:"100%",height:"100%",position:"relative",background:"var(--bg)"}}>
<canvas ref={canvasRef} style={{width:"100%",height:"100%",display:"block"}}/>
<div style={{position:"absolute",top:8,right:8,display:"flex",alignItems:"center",gap:4
<div style={{width:5,height:5,borderRadius:"50%",background:live?"var(--green)":"rgba
<span style={{fontSize:8,color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace"
</div>
</div>
);
}
// ─── Asset Modal ───────────────────────────────────────────────────────────
function AssetModal({asset,livePrice,onClose}){
const [chartType,setChartType]=useState("line");
const [range,setRange]=useState("1mo");
const p=livePrice||FALLBACK[asset.symbol];
const chgAmt=p?(p.price-p.prevClose):0;
const chgPct=p&&p.prevClose?(chgAmt/p.prevClose*100):0;
const cp=p?.price||asset.buyPrice;
const cv=asset.amount*cp, bv=asset.amount*asset.buyPrice, g=cv-bv, gp=(g/bv)*100;
const RANGES=[{label:"1T",range:"1d"},{label:"1W",range:"5d"},{label:"1M",range:"1mo"},{lab
return(
<div className="overlay" onClick={e=>e.target===e.currentTarget&&onClose()}>
<div className="modal">
<div className="modal-handle"/>
<div className="modal-header">
<div style={{display:"flex",alignItems:"center",gap:12,marginBottom:12}}>
<AssetLogo symbol={asset.symbol} size={44}/>
<div style={{flex:1}}>
<div style={{fontWeight:800,fontSize:18,fontFamily:"'JetBrains Mono',monospace"
<div style={{fontSize:11,color:"var(--text3)",fontFamily:"'JetBrains Mono',mono
</div>
<button onClick={onClose} style={{background:"var(--bg2)",border:"1px solid var(-
</div>
<div style={{display:"flex",alignItems:"baseline",gap:12,flexWrap:"wrap"}}>
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:28,fontWeight:700}}
{p?.currency==="USD"?"$":"€"}{p?fmt(p.price):fmt(cp)}
</div>
<div style={{fontWeight:700,fontSize:14,color:chgAmt>=0?"var(--green)":"var(--red
{chgAmt>=0?"+":""}{fmt(chgAmt)} ({chgAmt>=0?"+":""}{fmt(chgPct)}%)
</div>
<div style={{display:"flex",alignItems:"center",gap:5,marginLeft:"auto"}}>
<span className="ldot"/><span style={{fontSize:10,color:"var(--text3)",fontFami
</div>
</div>
</div>
<div style={{padding:"14px 18px"}}>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marg
<div style={{display:"flex",background:"var(--bg)",border:"1px solid var(--border
{[["line","Linie"],["candle","Kerzen"]].map(([id,lbl])=>(
<button key={id} onClick={()=>setChartType(id)} style={{padding:"4px 12px",bo
))}
</div>
<div style={{display:"flex",gap:3}}>
{RANGES.map(r=>(
<button key={r.range} onClick={()=>setRange(r.range)} style={{padding:"4px 8p
{r.label}
</button>
))}
</div>
</div>
<div style={{height:260,borderRadius:8,overflow:"hidden",border:"1px solid var(--bo
<AssetChart key={`${asset.symbol}-${range}-${chartType}`} symbol={asset.symbol} c
</div>
{p&&(
<div className="g2" style={{gap:8,marginTop:12}}>
{[["Eröffnung",`$${fmt(p.open)}`],["Tageshoch",`$${fmt(p.high)}`],["Tagestief",
<div key={l} style={{padding:"9px 12px",background:"var(--bg2)",borderRadius:
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform:"
<div style={{fontWeight:700,fontSize:13,fontFamily:"'JetBrains Mono',monosp
</div>
))}
</div>
)}
<div style={{marginTop:10,padding:14,background:g>=0?"rgba(168,168,179,0.06)":"rgba
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform:"upperc
<div className="g2" style={{gap:8}}>
{[["Anzahl",asset.amount],["Kaufpreis",`$${fmt(asset.buyPrice)}`],["Akt. Wert",
<div key={l}>
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform:"
<div style={{fontWeight:700,fontSize:13,fontFamily:"'JetBrains Mono',monosp
</div>
))}
</div>
</div>
</div>
</div>
</div>
);
}
// ─── Detail Modals ─────────────────────────────────────────────────────────
function DiversityModal({assets,onClose}){
const sectors={};
assets.forEach(a=>{const k=a.subSector||a.sector;if(!sectors[k])sectors[k]=0;sectors[k]+=a.
const total=Object.values(sectors).reduce((a,b)=>a+b,0);
const sorted=Object.entries(sectors).sort((a,b)=>b[1]-a[1]);
const cols=["#3b82f6","#14b8a6","#f59e0b","#8b5cf6","#ef4444","#06b6d4","#ec4899"];
const regions={};
assets.forEach(a=>{if(!regions[a.region])regions[a.region]=0;regions[a.region]+=a.amount*(a
return(
<div className="overlay" onClick={e=>e.target===e.currentTarget&&onClose()}>
<div className="modal">
<div className="modal-handle"/>
<div className="modal-header">
<div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
<div><div style={{fontWeight:800,fontSize:18,fontFamily:"'JetBrains Mono',monospa
<button onClick={onClose} style={{background:"var(--bg2)",border:"1px solid var(-
</div>
</div>
<div style={{padding:"16px 20px"}}>
<div style={{fontWeight:700,fontSize:13,marginBottom:12,color:"var(--text)"}}>Wirts
{sorted.map(([name,val],i)=>{const pct=val/total*100;return(
<div key={name} style={{marginBottom:12}}>
<div style={{display:"flex",justifyContent:"space-between",fontSize:13,fontWeig
<span style={{display:"flex",alignItems:"center",gap:7}}><div style={{width:9
<span style={{color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace"}}>
</div>
<div className="track"><div className="fill" style={{width:`${pct}%`,background
</div>
);})}
<div style={{fontWeight:700,fontSize:13,marginTop:20,marginBottom:12,color:"var(--t
{Object.entries(regions).map(([r,v],i)=>{const pct=v/total*100;return(
<div key={r} style={{marginBottom:10}}>
<div style={{display:"flex",justifyContent:"space-between",fontSize:13,fontWeig
<span style={{color:"var(--text)"}}>{r}</span>
<span style={{color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace"}}>
</div>
</div>
);})}
</div>
</div>
</div>
<div className="track"><div className="fill" style={{width:`${pct}%`,background
);
}
function FeesModal({onClose}){
const plats=[{name:"Trade Republic",fee:"€1 pro Trade",crypto:false,depot:true,note:"Günsti
return(
<div className="overlay" onClick={e=>e.target===e.currentTarget&&onClose()}>
<div className="modal">
<div className="modal-handle"/>
<div className="modal-header">
<div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
<div><div style={{fontWeight:800,fontSize:18,fontFamily:"'JetBrains Mono',monospa
<button onClick={onClose} style={{background:"var(--bg2)",border:"1px solid var(-
</div>
</div>
<div style={{padding:"16px 20px"}}>
<div style={{padding:"11px 14px",background:"rgba(245,158,11,0.08)",border:"1px sol
Optimierungspotenzial: bis zu <strong style={{color:"var(--text)"}}>€12–18/Monat<
</div>
<div style={{display:"flex",flexDirection:"column",gap:8}}>
{plats.map(p=>(
<div key={p.name} style={{padding:"12px 14px",background:"var(--bg2)",borderRad
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center
<div style={{fontWeight:700,fontSize:13,color:"var(--text)"}}>{p.name}</div
<span className="chip chip-g" style={{fontSize:10}}>{p.fee}</span>
</div>
<div style={{fontSize:11,color:"var(--text3)",marginBottom:5}}>{p.note}</div>
<div style={{display:"flex",gap:5}}>{p.depot&&<span className="chip chip-b" s
</div>
))}
</div>
</div>
</div>
</div>
);
}
function BewertungModal({assets,onClose}){
const stats=assets.length>0?(()=>{const cur=assets.reduce((s,a)=>s+a.amount*(a.currentPrice
const scoreLabel=stats.gainPct>20?"Überdurchschnittlich":stats.gainPct>8?"Durchschnittlich"
const scoreColor=stats.gainPct>20?"var(--green)":stats.gainPct>8?"var(--gold)":"var(--red)"
const rows=[{l:"Diversität",score:8.2,desc:"Gut diversifiziert über 5 Sektoren, 3 Regionen
const rc=s=>s>=8?"var(--green)":s>=6?"var(--gold)":"var(--red)";
const overall=(rows.reduce((s,r)=>s+r.score,0)/rows.length).toFixed(1);
return(
<div className="overlay" onClick={e=>e.target===e.currentTarget&&onClose()}>
<div className="modal">
<div className="modal-handle"/>
<div style={{padding:"0 20px 14px",borderBottom:"1px solid rgba(255,255,255,0.06)"}}>
<div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
<div><div style={{fontWeight:800,fontSize:18,fontFamily:"'JetBrains Mono',monospa
<button onClick={onClose} style={{background:"var(--bg2)",border:"1px solid var(-
</div>
</div>
<div style={{padding:"16px 20px",overflowY:"auto"}}>
<div style={{display:"flex",alignItems:"center",gap:18,padding:"14px 16px",backgrou
<div style={{position:"relative",width:72,height:72,flexShrink:0}}>
<svg width="72" height="72" viewBox="0 0 72 72" style={{transform:"rotate(-90de
<circle cx="36" cy="36" r="30" fill="none" stroke="rgba(255,255,255,0.07)" st
<circle cx="36" cy="36" r="30" fill="none" stroke={scoreColor} strokeWidth="7
</svg>
<div style={{position:"absolute",inset:0,display:"flex",flexDirection:"column",
<div style={{fontSize:18,fontWeight:800,fontFamily:"'JetBrains Mono',monospac
<div style={{fontSize:8,color:"var(--text3)",fontFamily:"'JetBrains Mono',mon
</div>
</div>
<div style={{flex:1}}>
<div style={{fontSize:10,color:"var(--text3)",fontFamily:"'JetBrains Mono',mono
<div style={{fontWeight:800,fontSize:14,color:scoreColor,marginBottom:8}}>{scor
<div style={{display:"flex",gap:7,flexWrap:"wrap"}}>
{[["YTD",`+${fmt(stats.gainPct,1)}%`,scoreColor],["S&P 500","+9.4%","var(--te
<div key={l} style={{padding:"4px 8px",background:"var(--bg2)",borderRadius
<div style={{fontSize:8,color:"var(--text3)",fontFamily:"'JetBrains Mono'
<div style={{fontSize:13,fontWeight:800,fontFamily:"'JetBrains Mono',mono
</div>
))}
</div>
</div>
</div>
<div style={{display:"flex",flexDirection:"column",gap:10}}>
{rows.map(row=>(
<div key={row.l} style={{padding:"12px 14px",background:"var(--bg2)",borderRadi
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center
<span style={{fontSize:13,fontWeight:700,color:"var(--text)"}}>{row.l}</spa
<span style={{fontSize:14,fontWeight:800,fontFamily:"'JetBrains Mono',monos
</div>
<div style={{height:4,background:"rgba(255,255,255,0.07)",overflow:"hidden",m
<div style={{height:"100%",width:`${row.score/10*100}%`,background:rc(row.s
</div>
<div style={{fontSize:12,color:"var(--text3)",lineHeight:1.55,fontFamily:"'Sp
</div>
))}
</div>
</div>
</div>
</div>
);
}
// ─── Platform config ───────────────────────────────────────────────────────
const PLATFORMS=[
{id:"binance", name:"Binance", type:"api", desc:"Spot & Futures", fields:["api
{id:"coinbase",name:"Coinbase", type:"api", desc:"Advanced Trade", fields:["ap
{id:"etoro", name:"eToro", type:"api", desc:"Social Trading", fields:["ap
{id:"scalable",name:"Scalable Capital", type:"api", desc:"Broker & ETF", fields:["ap
{id:"phantom", name:"Phantom Wallet", type:"wallet", desc:"Solana / EVM", fields:["wa
];
const DEMO_ASSETS=[
{symbol:"BTC", name:"Bitcoin", sector:"Crypto",subSector:"Layer-1", region:"G
{symbol:"ETH", name:"Ethereum", sector:"Crypto",subSector:"Layer-1", region:"G
{symbol:"SOL", name:"Solana", sector:"Crypto",subSector:"Layer-1", region:"G
{symbol:"AAPL",name:"Apple Inc.", sector:"Tech", subSector:"Consumer Electronics",regi
{symbol:"MSFT",name:"Microsoft", sector:"Tech", subSector:"Cloud/AI", region:"U
{symbol:"NVDA",name:"NVIDIA", sector:"Tech", subSector:"Semiconductors", region:"U
{symbol:"SPY", name:"S&P 500 ETF", sector:"ETF", subSector:"Broad Market", region:"U
{symbol:"VWRL",name:"Vanguard World", sector:"ETF", subSector:"Global Equity", region:"G
];
const DEMO_TX=[
{date:"22.04.25",type:"Kauf", symbol:"BTC", amount:0.05,price:65800,fees:12.40},
{date:"18.04.25",type:"Verkauf",symbol:"ETH", amount:1.0, price:3510, fees:8.20},
{date:"15.04.25",type:"Kauf", symbol:"NVDA",amount:2, price:840, fees:6.00},
{date:"10.04.25",type:"Kauf", symbol:"SOL", amount:10, price:148, fees:3.50},
{date:"05.04.25",type:"Kauf", symbol:"SPY", amount:5, price:516, fees:5.10},
];
const DEMO_NEWS=[
{time:"3 Min.", tag:"MARKT", {time:"12 Min.",tag:"KRYPTO", {time:"28 Min.",tag:"POLITIK", text:"Fed signalisiert mögliche Zinssenkung im Q3 2025",
text:"Bitcoin ETF Zuflüsse erreichen Rekordhoch von $2.1B
text:"EU-Digitalmarktgesetz: Big Tech unter Druck",
{time:"41 Min.",tag:"WIRTSCHAFT", text:"Inflationsdaten USA: 2.8% — unter Erwartungen",
{time:"1 Std.", tag:"MARKT", text:"NVIDIA übertrifft Quartalsergebnisse um 18%",
{time:"2 Std.", tag:"KRYPTO", text:"Ethereum Pectra-Upgrade erfolgreich deployed",
{time:"3 Std.", tag:"WIRTSCHAFT", text:"EZB hält Leitzins stabil bis mindestens Juni",
{time:"4 Std.", tag:"MARKT", text:"S&P 500 schließt auf Allzeithoch: 5.264 Punkte",
];
// ─── Portfolio Engine ──────────────────────────────────────────────────────
function useEngine(){
const [connections,setConnections]=useState(()=>{try{return JSON.parse(localStorage.getItem
const [portfolio,setPortfolio]=useState([]);
const [syncStatus,setSyncStatus]=useState({});
const [lastSync,setLastSync]=useState(null);
const persist=c=>{localStorage.setItem("ybu_conn",JSON.stringify(c.map(x=>({id:x.id,platfor
const connect=async(platformId,creds)=>{
setSyncStatus(s=>({...s,[platformId]:"syncing"}));
await new Promise(r=>setTimeout(r,2000));
const p=PLATFORMS.find(x=>x.id===platformId);
const nc={id:Date.now().toString(),platform:platformId,name:p.name,connectedAt:new persist([...connections.filter(c=>c.platform!==platformId),nc]);
const items=DEMO_ASSETS.filter(a=>a.platform===platformId);
setPortfolio(prev=>[...prev.filter(a=>a.platform!==platformId),...items]);
setSyncStatus(s=>({...s,[platformId]:"ok"}));
setLastSync(new Date());
Date()
};
const disconnect=id=>{persist(connections.filter(c=>c.platform!==id));setPortfolio(prev=>pr
const syncAll=async()=>{
for(const c of connections){setSyncStatus(s=>({...s,[c.platform]:"syncing"}));await new P
setLastSync(new Date());
};
const importCSV=text=>{
const lines=text.trim().split("\n").slice(1);
const items=lines.map(l=>{const[symbol,name,amount,buyPrice,currentPrice]=l.split(",").ma
if(items.length){setPortfolio(prev=>[...prev.filter(a=>a.platform!=="csv"),...items]);per
return items.length;
};
useEffect(()=>{if(connections.length>0&&portfolio.length===0)connections.forEach(c=>{const
useEffect(()=>{if(!connections.length)return;const id=setInterval(syncAll,10*60*1000);retur
return{connections,portfolio,syncStatus,lastSync,connect,disconnect,syncAll,importCSV};
}
// ─── Portfolio Growth Chart ────────────────────────────────────────────────
function genHistory(assets,days=365){
const now=Date.now(), MS=86400000;
const cats={all:assets,stocks:assets.filter(a=>a.sector==="Tech"),crypto:assets.filter(a=>a
const series={};
for(const[key,items]of Object.entries(cats)){
if(!items.length){series[key]=[];continue;}
const sv=items.reduce((s,a)=>s+a.amount*a.buyPrice,0)||1;
const ev=items.reduce((s,a)=>s+a.amount*(a.currentPrice||a.buyPrice),0);
const pts=[];
for(let i=0;i<=days;i++){
const t=i/days, trend=Math.pow(ev/sv,t);
const noise=1+(Math.sin(i*0.31)*0.018+Math.sin(i*0.07)*0.025+(Math.random()-0.5)*0.012)
const crash=(i>80&&i<100)?0.93:(i>210&&i<230)?0.96:1;
pts.push({time:now-(days-i)*MS,value:sv*trend*noise*crash});
}
series[key]=pts;
}
const bs=series.all[0]?.value||1;
series.benchmark=series.all.map((p,i)=>{const t=i/days;return{time:p.time,value:bs*Math.pow
return series;
}
function PortfolioGrowthChart({assets}){
const canvasRef=useRef(null);
const [filter,setFilter]=useState("all");
const [mode,setMode]=useState("growth");
const [showBench,setShowBench]=useState(false);
const [range,setRange]=useState(365);
const [expanded,setExpanded]=useState(false);
const seriesRef=useRef(null);
useEffect(()=>{seriesRef.current=genHistory(assets);},[]);
const FILTERS=[{id:"all",label:"Gesamt"},{id:"stocks",label:"Aktien"},{id:"crypto",label:"C
const RANGES=[{label:"1M",days:30},{label:"3M",days:90},{label:"6M",days:180},{label:"1J",d
const currentSeries=(seriesRef.current?.[filter]||[]).slice(-range);
const lastVal=currentSeries[currentSeries.length-1]?.value||0;
const startVal=currentSeries[0]?.value||1;
const growthPct=(lastVal-startVal)/startVal*100;
const ath=currentSeries.length?Math.max(...currentSeries.map(p=>p.value)):lastVal;
const ddFromATH=(lastVal-ath)/ath*100;
let maxLoss=0,curL=0;
currentSeries.forEach((p,i)=>{if(i>0&&p.value<currentSeries[i-1].value){curL++;maxLoss=Math
const MSCI={30:1.1,90:2.8,180:5.4,365:11.2}[range]||11.2;
const alerts=[];
if(ddFromATH<-5)alerts.push({type:"warn",msg:`Portfolio ist ${fmt(Math.abs(ddFromATH),1)}%
if(growthPct>0)alerts.push({type:"ok",msg:`Wachstum: +${fmt(growthPct,1)}% in diesem Zeitra
useEffect(()=>{
const canvas=canvasRef.current; if(!canvas||!seriesRef.current)return;
const pts=currentSeries; const bench=(seriesRef.current.benchmark||[]).slice(-range);
if(!pts.length)return;
const dpr=window.devicePixelRatio||1, W=canvas.offsetWidth, H=canvas.offsetHeight;
if(!W||!H)return;
canvas.width=W*dpr; canvas.height=H*dpr;
const ctx=canvas.getContext("2d"); ctx.scale(dpr,dpr);
ctx.clearRect(0,0,W,H);
const PAD={t:16,b:30,l:8,r:60}, cw=W-PAD.l-PAD.r, ch=H-PAD.t-PAD.b;
const allV=showBench?[...pts.map(p=>p.value),...bench.map(p=>p.value)]:pts.map(p=>p.value
const maxP=Math.max(...allV)*1.02, minP=Math.min(...allV)*0.98, rng=maxP-minP||1;
const toX=i=>PAD.l+(i/(pts.length-1))*cw;
const toY=v=>PAD.t+ch-((v-minP)/rng)*ch;
[0,0.25,0.5,0.75,1].forEach(r=>{
const y=PAD.t+ch*r; ctx.strokeStyle="rgba(255,255,255,0.05)"; ctx.lineWidth=1;
ctx.beginPath();ctx.moveTo(PAD.l,y);ctx.lineTo(W-PAD.r,y);ctx.stroke();
const v=maxP-(maxP-minP)*r; const lbl=v>1000?`€${Math.round(v/1000)}k`:`€${Math.round(v
ctx.fillStyle="rgba(255,255,255,0.30)"; ctx.font=`9px 'JetBrains Mono',monospace`; ctx.
ctx.fillText(lbl,W-PAD.r+3,y+3);
});
const isUp=pts[pts.length-1].value>=pts[0].value, col=isUp?"#22c55e":"#ef4444";
if(mode==="growth"){
if(showBench&&bench.length>1){
ctx.beginPath(); bench.forEach((p,i)=>{const x=toX(i),y=toY(p.value);i===0?ctx.moveTo
ctx.strokeStyle="rgba(167,139,250,0.5)"; ctx.lineWidth=1.5; ctx.setLineDash([4,4]); c
}
const grad=ctx.createLinearGradient(0,PAD.t,0,PAD.t+ch);
grad.addColorStop(0,isUp?"rgba(34,197,94,0.20)":"rgba(239,68,68,0.20)"); grad.addColorS
ctx.beginPath(); pts.forEach((p,i)=>{const x=toX(i),y=toY(p.value);i===0?ctx.moveTo(x,y
ctx.lineTo(toX(pts.length-1),PAD.t+ch); ctx.lineTo(PAD.l,PAD.t+ch); ctx.closePath(); ct
ctx.beginPath(); pts.forEach((p,i)=>{const x=toX(i),y=toY(p.value);i===0?ctx.moveTo(x,y
ctx.strokeStyle=col; ctx.lineWidth=2; ctx.lineJoin="round"; ctx.stroke();
} else if(mode==="loss"){
const midY=toY(pts[0].value);
ctx.beginPath(); ctx.moveTo(PAD.l,midY); ctx.lineTo(W-PAD.r,midY);
ctx.strokeStyle="rgba(255,255,255,0.08)"; ctx.lineWidth=1; ctx.stroke();
for(let i=1;i<pts.length;i++){
const rt=(pts[i].value-pts[0].value)/pts[0].value; if(rt>=0)continue;
const x1=toX(i-1),x2=toX(i),y2=toY(pts[i].value);
ctx.beginPath(); ctx.moveTo(x1,midY); ctx.lineTo(x2,y2); ctx.lineTo(x2,midY); ctx.clo
ctx.fillStyle="rgba(239,68,68,0.25)"; ctx.fill();
ctx.beginPath(); ctx.moveTo(x1,midY); ctx.lineTo(x2,y2);
ctx.strokeStyle="#ef4444"; ctx.lineWidth=1.5; ctx.stroke();
}
} else {
let ath2=pts[0].value;
const ddPts=pts.map(p=>{ath2=Math.max(ath2,p.value);return{v:p.value,dd:(p.value-ath2)/
const minDD=Math.min(...ddPts.map(p=>p.dd));
const toYdd=v=>PAD.t+ch-((v-minDD)/(0-minDD+0.001))*ch;
ctx.beginPath(); ddPts.forEach((p,i)=>{const x=toX(i),y=toYdd(p.dd);i===0?ctx.moveTo(x,
ctx.lineTo(toX(pts.length-1),PAD.t+ch); ctx.lineTo(PAD.l,PAD.t+ch); ctx.closePath();
ctx.fillStyle="rgba(239,68,68,0.12)"; ctx.fill();
ctx.beginPath(); ddPts.forEach((p,i)=>{const x=toX(i),y=toYdd(p.dd);i===0?ctx.moveTo(x,
ctx.strokeStyle="#ef4444"; ctx.lineWidth=2; ctx.stroke();
}
const step=Math.max(1,Math.floor(pts.length/6));
ctx.fillStyle="rgba(255,255,255,0.30)"; ctx.font=`8px 'JetBrains Mono',monospace`; ctx.te
for(let i=0;i<pts.length;i+=step){const d=new Date(pts[i].time);ctx.fillText(d.toLocaleDa
},[filter,mode,range,showBench,expanded]);
return(
<div className="card" style={{marginBottom:12,overflow:"hidden"}}>
<div style={{padding:"14px 16px 0",display:"flex",justifyContent:"space-between",alignI
<div>
<div style={{fontWeight:700,fontSize:13}}>Portfolio Wachstum</div>
<div style={{fontSize:10,color:"var(--text3)",fontFamily:"'JetBrains Mono',monospac
</div>
<button onClick={()=>setExpanded(v=>!v)} style={{background:"transparent",border:"1px
{expanded?"Min":"Max"}
</button>
</div>
<div style={{padding:"10px 16px",display:"flex",gap:6,flexWrap:"wrap",alignItems:"cente
<div style={{display:"flex",gap:3}}>
{FILTERS.map(f=><button key={f.id} onClick={()=>setFilter(f.id)} style={{padding:"3
</div>
<div style={{display:"flex",gap:3,marginLeft:"auto"}}>
{[["growth","Wachstum"],["loss","Verluste"],["drawdown","Drawdown"]].map(([id,lbl])
</div>
</div>
<canvas ref={canvasRef} style={{width:"100%",height:expanded?280:160,display:"block"}}/
<div style={{padding:"7px 16px 10px",display:"flex",gap:6,alignItems:"center",borderTop
<div style={{display:"flex",gap:3}}>{RANGES.map(r=><button key={r.days} onClick={()=>
<button onClick={()=>setShowBench(v=>!v)} style={{marginLeft:"auto",padding:"3px 9px"
</div>
{expanded&&(
<div style={{padding:"0 16px 12px",display:"flex",gap:8,flexWrap:"wrap"}}>
{[{l:"Wachstum",v:`${growthPct>=0?"+":""}${fmt(growthPct,1)}%`,c:growthPct>=0?"var(
<div key={s.l} style={{flex:"1 1 100px",padding:"9px 12px",background:"var(--bg2)
<div style={{fontSize:8,fontWeight:700,color:"var(--text3)",textTransform:"uppe
<div style={{fontWeight:800,fontSize:15,fontFamily:"'JetBrains Mono',monospace"
</div>
))}
</div>
)}
{alerts.length>0&&(
<div style={{padding:"0 16px 12px",display:"flex",flexDirection:"column",gap:5}}>
{alerts.map((a,i)=>(
<div key={i} style={{display:"flex",gap:8,alignItems:"center",padding:"7px 11px",
<div style={{width:5,height:5,borderRadius:"50%",flexShrink:0,background:a.type
<span style={{fontSize:11,color:"var(--text2)",fontFamily:"'Space Grotesk',sans
</div>
))}
</div>
)}
</div>
);
}
// ─── CSS ───────────────────────────────────────────────────────────────────
const CSS=`
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;600;700
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{
/* Dark institutional finance palette */
--bg:#080808;
--bg2:#0f0f0f;
--bg3:#161616;
--white:#1a1a1a;
--border:rgba(255,255,255,0.07);
--border2:rgba(255,255,255,0.12);
--border3:rgba(255,255,255,0.22);
--text:#F0F0F0;
--text2:#A0A0A0;
--text3:#4A4A4A;
--accent:#A8A8B3;
--accent2:#8888A0;
--al:rgba(168,168,179,0.08);
--green:#22c55e;--gbg:rgba(34,197,94,0.07);--gbd:rgba(34,197,94,0.22);
--red:#ef4444;--rbg:rgba(239,68,68,0.08);--rbd:rgba(239,68,68,0.2);
--gold:#f59e0b;--goldbg:rgba(245,158,11,0.08);--goldbd:rgba(245,158,11,0.25);
--pur:#a78bfa;--purbg:rgba(167,139,250,0.08);
--glow:#A8A8B3;
--glow-blue:#3b82f6;
--r:8px;--rsm:5px;
--sh:0 4px 24px rgba(0,0,0,0.6),0 1px 4px rgba(0,0,0,0.4);
--sh-lg:0 16px 60px rgba(0,0,0,0.8),0 4px 16px rgba(0,0,0,0.5);
--sh-glow:0 0 30px rgba(168,168,179,0.1),0 4px 24px rgba(0,0,0,0.6);
}
html,body{
height:100%;
font-family:'Space Grotesk',system-ui,sans-serif;
background:var(--bg);
color:var(--text);
-webkit-font-smoothing:antialiased;
overflow-x:hidden;
scroll-behavior:smooth;
}
/* Cinematic noise texture */
html::before{
content:'';position:fixed;inset:0;z-index:0;pointer-events:none;
background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.
opacity:0.4;
}
::-webkit-scrollbar{width:2px}::-webkit-scrollbar-thumb{background:rgba(168,168,179,0.25);bor
.card{
background:var(--bg2);
border:1px solid var(--border);
border-radius:var(--r);
box-shadow:var(--sh);
position:relative;
overflow:hidden;
}
/* Lightning border animation on cards */
.card::before{
content:'';position:absolute;inset:0;border-radius:var(--r);
background:linear-gradient(135deg,transparent 40%,rgba(168,168,179,0.07) 50%,transparent 60
opacity:0;transition:opacity .3s;pointer-events:none;
1px rg
}
.card:hover::before{opacity:1}
.card-tap{cursor:pointer;transition:all .2s}
.card-tap:hover{transform:translateY(-2px);box-shadow:0 8px 32px rgba(0,0,0,0.6),0 0 0 .card-tap:active{transform:scale(.99)}
.btn{display:inline-flex;align-items:center;gap:7px;border:none;cursor:pointer;font-family:'S
.btn:disabled{opacity:.3;cursor:not-allowed}
.btn-blue{
background:linear-gradient(135deg,#14b8a6,#0d9488);
color:#000;padding:12px 24px;font-size:14px;font-weight:700;
box-shadow:0 0 20px rgba(168,168,179,0.25),0 4px 12px rgba(0,0,0,0.4);
}
.btn-blue:not(:disabled):hover{transform:translateY(-2px);box-shadow:0 0 35px rgba(168,168,17
.btn-white{background:rgba(255,255,255,0.06);color:var(--text);border:1px solid var(--border2
.btn-white:hover{background:rgba(255,255,255,0.1);border-color:var(--border3)}
.btn-ghost{background:transparent;color:var(--text2);padding:8px 14px;font-size:13px;border:1
.btn-ghost:hover{background:var(--bg3);border-color:var(--border);color:var(--text)}
.btn-danger{background:var(--rbg);color:var(--red);border:1px solid var(--rbd);padding:7px 14
.btn-lg{padding:14px 32px!important;font-size:15px!important}
.btn-sm{padding:7px 14px!important;font-size:12px!important}
.inp{width:100%;padding:12px 14px;border:1px solid var(--border2);border-radius:var(--rsm);fo
.inp:focus{border-color:rgba(168,168,179,0.4);box-shadow:0 0 0 3px rgba(168,168,179,0.07);bac
.inp::placeholder{color:var(--text3)}.inp-mono{font-family:'JetBrains Mono',monospace;font-si
.inp-ok{border-color:var(--green)!important;box-shadow:0 0 0 3px rgba(168,168,179,0.09)!impor
.lbl{display:block;font-size:10px;font-weight:600;color:var(--text3);text-transform:uppercase
.chip{display:inline-flex;align-items:center;gap:4px;padding:3px 9px;border-radius:999px;font
.chip-g{background:var(--gbg);color:var(--green);border-color:var(--gbd)}
.chip-r{background:var(--rbg);color:var(--red);border-color:var(--rbd)}
.chip-b{background:var(--al);color:var(--accent);border-color:rgba(34,197,94,.2)}
.chip-gold{background:var(--goldbg);color:var(--gold);border-color:var(--goldbd)}
.chip-gray{background:var(--bg3);color:var(--text2);border-color:var(--border)}
.chip-pur{background:var(--purbg);color:var(--pur);border-color:rgba(167,139,250,.2)}
.tag{padding:2px 8px;border-radius:999px;font-size:9px;font-weight:800;letter-spacing:.07em;t
.tag-bull{background:rgba(168,168,179,0.09);color:var(--green);border:1px solid rgba(168,168,
.tag-bear{background:rgba(185,28,28,0.08);color:var(--red);border:1px solid rgba(185,28,28,0.
.tag-neu{background:var(--bg2);color:var(--text3);border:1px solid var(--border)}
@keyframes fadeUp{from{opacity:0;transform:translateY(16px)}to{opacity:1;transform:translateY
@keyframes fadeIn{from{opacity:0}to{opacity:1}}
@keyframes scaleIn{from{opacity:0;transform:scale(.95)}to{opacity:1;transform:scale(1)}}
@keyframes spin{to{transform:rotate(360deg)}}
@keyframes pulse{0%,100%{opacity:1;transform:scale(1)}50%{opacity:.4;transform:scale(1.5)}}
@keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-8px)}}
@keyframes ticker{0%{transform:translateX(0)}100%{transform:translateX(-50%)}}
@keyframes glow{0%,100%{box-shadow:0 0 0 0 rgba(37,99,235,.3)}50%{box-shadow:0 0 16px 4px rgb
@keyframes countUp{from{opacity:0;transform:translateY(5px)}to{opacity:1;transform:translateY
@keyframes slideUp{from{opacity:0;transform:translateY(40px)}to{opacity:1;transform:translate
@keyframes revealUp{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:translat
@keyframes revealFade{from{opacity:0}to{opacity:1}}
@keyframes otpPop{0%{transform:scale(1)}45%{transform:scale(1.12)}100%{transform:scale(1)}}
@keyframes otpGlowPulse{0%,100%{opacity:.55}50%{opacity:1}}
@keyframes otpAllGlow{0%,100%{opacity:.6;transform:scale(1)}50%{opacity:1;transform:scale(1.0
@keyframes checkmarkDraw{from{stroke-dashoffset:48}to{stroke-dashoffset:0}}
@keyframes verifiedGlow{0%{opacity:0;transform:scale(.85)}100%{opacity:1;transform:scale(1)}}
@keyframes verifiedRing{0%{opacity:.5;transform:scale(.7)}100%{opacity:0;transform:scale(1.8)
@keyframes bgFlash{0%{opacity:0}50%{opacity:.6}100%{opacity:0}}
@keyframes otpShake{0%,100%{transform:translateX(0)}20%{transform:translateX(-5px)}40%{transf
@keyframes otpDotBounce{0%,100%{transform:translateX(-50%) scaleX(1)}50%{transform:translateX
@keyframes otpSuccess{0%{transform:scale(1)}50%{transform:scale(1.08)}100%{transform:scale(1)
@keyframes otpFadeSlide{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:trans
@keyframes slideInLeft{from{opacity:0;transform:translateX(-16px)}to{opacity:1;transform:tran
@keyframes noteReveal{from{opacity:0;transform:perspective(900px) rotateY(-25deg) translateY(
.scroll-reveal{opacity:0;transform:translateY(28px);transition:opacity .7s cubic-bezier(.16,1
.scroll-reveal.visible{opacity:1;transform:translateY(0)}
.scroll-reveal-left{opacity:0;transform:translateX(-24px);transition:opacity .7s cubic-bezier
.scroll-reveal-left.visible{opacity:1;transform:translateX(0)}
.scroll-reveal-note{opacity:0;transform:perspective(900px) rotateY(-20deg);transition:opacity
.scroll-reveal-note.visible{opacity:1;transform:perspective(900px) rotateY(0deg)}
@keyframes chartDraw{from{stroke-dashoffset:1200}to{stroke-dashoffset:0}}
.au{animation:fadeUp .4s ease both}.ai{animation:fadeIn .25s ease both}.as{animation:scaleIn
.d1{animation-delay:.05s}.d2{animation-delay:.1s}.d3{animation-delay:.15s}.d4{animation-delay
.nav{position:fixed;top:0;left:0;right:0;z-index:200;height:56px;background:rgba(8,8,8,0.85);
.nav::before{content:'';position:absolute;inset:0;background:linear-gradient(90deg,rgba(168,1
.nav-inner{position:relative;z-index:1;max-width:1280px;margin:0 auto;padding:0 20px;height:1
.logo{font-family:'Space Grotesk',sans-serif;font-size:16px;font-weight:700;color:#fff;cursor
.logo-acc{color:rgba(200,200,210,0.7)}
.nav-btn{background:transparent;border:1px solid var(--border);color:var(--text2);border-radi
.nav-btn:hover{background:rgba(255,255,255,0.06);border-color:var(--border2);color:var(--text
.nav-btn-solid{background:linear-gradient(135deg,#A8A8B3,#888899);border:none;color:#000;bord
.nav-btn-solid:hover{box-shadow:0 0 35px rgba(168,168,179,0.35);transform:translateY(-1px)}
.sidebar{position:fixed;top:56px;left:0;bottom:0;width:220px;background:rgba(8,8,8,0.95);back
.sb-sec{font-size:9px;font-weight:700;color:var(--text3);text-transform:uppercase;letter-spac
.sb-item{display:flex;align-items:center;gap:9px;padding:8px 12px;border-radius:2px;color:var
.sb-item:hover{background:var(--bg2);color:var(--text)}
.sb-item.active{background:rgba(168,168,179,0.07);color:var(--green);font-weight:600;border:1
.sb-item.active::before{content:'';position:absolute;left:0;top:50%;transform:translateY(-50%
.mob-nav{display:none;position:fixed;bottom:0;left:0;right:0;z-index:200;background:rgba(8,8,
.mob-nav-inner{display:flex;justify-content:space-around}
.mob-item{display:flex;flex-direction:column;align-items:center;gap:2px;padding:6px 14px;bord
.mob-item.active{color:var(--green)}.mob-item:not(.active){color:var(--text3)}
.mob-label{font-size:9px;font-weight:600;font-family:'JetBrains Mono',monospace;text-transfor
.main{margin-left:220px;margin-top:56px;padding:28px 32px;min-height:calc(100vh - 56px);backg
.pg-title{font-family:'Space Grotesk',sans-serif;font-size:24px;font-weight:700;letter-spacin
.pg-sub{font-size:13px;color:var(--text3);margin-bottom:24px;font-family:'Space Grotesk',sans
.g2{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.g3{display:grid;grid-template-columns:repeat(3,1fr);gap:10px}
.g4{display:grid;grid-template-columns:repeat(4,1fr);gap:9px}
.stat{padding:16px 18px}
.stat-l{font-size:9px;font-weight:700;color:var(--text3);text-transform:uppercase;letter-spac
.stat-v{font-size:22px;font-weight:700;letter-spacing:-.02em;line-height:1.1;animation:countU
.stat-d{font-size:11px;font-weight:600;margin-top:4px;display:flex;align-items:center;gap:4px
.pos{color:#22c55e}.neg{color:var(--red)}.neu{color:var(--text3)}
.track{height:2px;background:rgba(255,255,255,0.06);overflow:hidden;margin-top:5px}
.fill{height:100%;transition:width .9s cubic-bezier(.34,1.56,.64,1)}
.tbl{width:100%;border-collapse:collapse}
.tbl th{font-size:9px;font-weight:700;color:var(--text3);text-transform:uppercase;letter-spac
.tbl td{font-size:13px;padding:12px 14px;border-bottom:1px solid rgba(255,255,255,.04);color:
.tbl tr:last-child td{border-bottom:none}.tbl tr:hover td{background:var(--bg3);cursor:pointe
.ticker-wrap{overflow:hidden;background:rgba(168,168,179,0.07);border-bottom:1px solid rgba(1
.ticker-inner{display:inline-flex;animation:ticker 35s linear infinite}
.ticker-item{display:inline-flex;align-items:center;gap:8px;padding:0 18px;font-size:10px;fon
.spn{width:14px;height:14px;border:1.5px solid rgba(255,255,255,0.08);border-top-color:var(--
.ldot{width:6px;height:6px;border-radius:50%;background:var(--green);box-shadow:0 0 8px rgba(
.overlay{position:fixed;inset:0;background:rgba(0,0,0,0.7);backdrop-filter:blur(12px);z-index
.modal{background:var(--bg2);border:1px solid var(--border2);border-top:1px solid rgba(168,16
.modal-handle{width:36px;height:3px;background:rgba(255,255,255,0.15);border-radius:99px;marg
.modal-header{padding:0 20px 14px;border-bottom:1px solid var(--border)}
.chart-toggle{display:flex;background:var(--bg3);border:1px solid var(--border);border-radius
.chart-toggle button{flex:1;padding:5px 12px;border:none;border-radius:4px;font-size:9px;font
.chart-toggle button.active{background:var(--bg3);color:var(--text);border:1px solid var(--bo
.sec-b{background:rgba(168,168,179,0.05);border:1px solid rgba(168,168,179,0.16);border-radiu
.steps{display:flex;align-items:center;margin-bottom:28px}
.step-n{width:26px;height:26px;border-radius:5px;display:flex;align-items:center;justify-cont
.step-n.done{background:#22c55e;color:#000}.step-n.act{background:#fff;color:#000;box-shadow:
.step-ln{flex:1;height:1px;background:var(--border)}.step-ln.done{background:var(--green)}
.drop{border:1px dashed var(--border2);border-radius:var(--r);padding:22px;text-align:center;
.drop:hover{border-color:rgba(168,168,179,0.3);background:rgba(168,168,179,0.04)}
.plt-tile{padding:13px;border-radius:var(--r);border:1px solid var(--border);background:var(-
.plt-tile:hover{border-color:var(--border2);background:rgba(255,255,255,0.04)}.plt-tile.sel{b
.sc{display:inline-flex;align-items:center;gap:6px;padding:4px 10px;border-radius:999px;font-
.sc-ok{background:rgba(168,168,179,0.07);color:var(--green);border:1px solid var(--gbd)}
.av{width:32px;height:32px;border-radius:var(--r);background:rgba(168,168,179,0.09);border:1p
.av:hover{background:rgba(168,168,179,0.16);box-shadow:0 0 16px rgba(168,168,179,0.25)}
.price-n{font-family:'JetBrains Mono',monospace;font-size:52px;font-weight:700;letter-spacing
.fl{display:flex;align-items:center;gap:8px;font-size:13px;margin-bottom:9px}
.fl-ck{width:16px;height:16px;border-radius:3px;background:rgba(168,168,179,0.1);border:1px s
.hero{display:flex;align-items:flex-start;padding:80px 24px 60px;gap:48px;max-width:1240px;ma
.hero-title{font-family:'Space Grotesk',sans-serif;font-size:clamp(32px,5vw,68px);font-weight
.hero-sub{font-size:14px;color:var(--text3);line-height:1.7;margin-bottom:28px;max-width:440p
.eyebrow{display:inline-flex;align-items:center;gap:7px;padding:4px 12px;border-radius:999px;
@media(max-width:1024px){.g4{grid-template-columns:1fr 1fr}}
@media(max-width:768px){
.sidebar{display:none}.mob-nav{display:block}
.main{margin-left:0;padding:14px 14px calc(80px + env(safe-area-inset-bottom,0px));backgrou
.g4,.g3,.g2{grid-template-columns:1fr}.hero{flex-direction:column;padding:72px 16px 40px;ga
.pg-title{font-size:18px}
}
@media(max-width:400px){.main{padding:12px 10px calc(80px + env(safe-area-inset-bottom,0px))}
/* Ambient dashboard glow */
@keyframes ambientGlow{0%,100%{opacity:.4;transform:scale(1)}50%{opacity:.6;transform:scale(1
.ambient-orb{animation:ambientGlow 8s ease-in-out infinite;pointer-events:none}
/* Card border lightning sweep */
@keyframes borderSweep{0%{background-position:0% 50%}100%{background-position:200% 50%}}
/* Number counter glow */
.stat-v{text-shadow:0 0 30px rgba(168,168,179,0.12)}
.pos{text-shadow:0 0 12px rgba(168,168,179,0.3)}
.neg{text-shadow:0 0 12px rgba(239,68,68,0.4)}
/* Glassmorphism panels */
.glass{background:rgba(255,255,255,0.03);backdrop-filter:blur(20px);-webkit-backdrop-filter:b
/* Glow border utility */
.glow-border{box-shadow:0 0 0 1px rgba(168,168,179,0.16),0 0 20px rgba(168,168,179,0.07),inse
/* Premium tag */
.tag-premium{background:linear-gradient(135deg,rgba(168,168,179,0.1),rgba(168,168,179,0.05));
`;
// ─── Ticker ─────────────────────────────────────────────────────────────────
const TICKER_SYMS=["BTC","ETH","SOL","AAPL","MSFT","NVDA","SPY"];
function TickerBar(){
const {prices}=useLivePrices(TICKER_SYMS);
const items=TICKER_SYMS.map(s=>{
const p=prices[s];
if(!p)return{s,price:"...",chg:"",up:null};
const chg=((p.price-p.prevClose)/p.prevClose)*100;
return{s,price:p.price>1000?`$${(p.price/1000).toFixed(1)}k`:`$${fmt(p.price,2)}`,chg:`${
});
return(
<div className="ticker-wrap">
<div className="ticker-inner">
{[0,1].map(ri=>(
<span key={ri}>
{items.map(({s,price,chg,up})=>(
<span key={s+ri} className="ticker-item">
<AssetLogo symbol={s} size={14}/>
<span style={{opacity:.6}}>{s}</span>
<span>{price}</span>
{chg&&<span style={{color:up?"#86efac":"#fca5a5"}}>{chg}</span>}
<span style={{opacity:.25,margin:"0 4px"}}>·</span>
</span>
))}
</span>
))}
</div>
</div>
);
}
// ─── Overview ──────────────────────────────────────────────────────────────
function Overview({engine}){
const items=engine.portfolio.length>0?engine.portfolio:DEMO_ASSETS;
const {prices,refresh}=useLivePrices(items.map(a=>a.symbol));
const enriched=items.map(a=>({...a,currentPrice:prices[a.symbol]?.price||a.currentPrice||a.
const cur=enriched.reduce((s,a)=>s+a.amount*a.currentPrice,0);
const buy=enriched.reduce((s,a)=>s+a.amount*a.buyPrice,0);
const gain=cur-buy, gainPct=(gain/buy)*100;
const sectors=[...new Set(enriched.map(a=>a.sector))];
const segs=sectors.map((s,i)=>{const val=enriched.filter(a=>a.sector===s).reduce((t,a)=>t+a
const [modal,setModal]=useState(null);
const [activeAsset,setActiveAsset]=useState(null);
return(
<div>
{modal==="diversity"&&<DiversityModal assets={enriched} onClose={()=>setModal(null)}/>}
{modal==="fees"&&<FeesModal onClose={()=>setModal(null)}/>}
{modal==="bewertung"&&<BewertungModal assets={enriched} onClose={()=>setModal(null)}/>}
{activeAsset&&<AssetModal asset={activeAsset} livePrice={prices[activeAsset.symbol]} on
<div style={{display:"flex",alignItems:"flex-start",justifyContent:"space-between",marg
0 12px
<div>
<h1 className="pg-title" style={{background:"linear-gradient(135deg,#fff 60%,rgba(3
<div style={{display:"flex",alignItems:"center",gap:7,marginTop:4}}>
<span className="ldot"/><span style={{fontSize:11,color:"var(--text3)",fontFamily
</div>
</div>
<div style={{display:"flex",gap:8,flexWrap:"wrap"}}>
{engine.connections.slice(0,2).map(c=><div key={c.id} className="sc sc-ok"><PlatBad
<button className="btn btn-white btn-sm" onClick={()=>{engine.syncAll();refresh();}
</div>
</div>
<div className="g4" style={{marginBottom:12}}>
{[{l:"Gesamtwert",v:fmtEur(cur),d:`+${fmtEur(gain)}`,pos:true},{l:"Tagesgewinn",v:"+€
<div key={s.l} className={`card stat au d${i+1}`}>
<div className="stat-l">{s.l}</div><div className="stat-v">{s.v}</div>
<div className={`stat-d ${s.pos?"pos":"neg"}`} style={{textShadow:s.pos?"0 </div>
))}
</div>
<PortfolioGrowthChart assets={enriched}/>
<div className="g2" style={{marginBottom:12}}>
<div className="card" style={{padding:18}}>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marg
<div style={{fontWeight:700,fontSize:13}}>Performance</div>
<span className="chip chip-g">+{fmt(gainPct,1)}%</span>
</div>
<div style={{fontSize:10,color:"var(--text3)",fontFamily:"'JetBrains Mono',monospac
<div style={{display:"flex",alignItems:"flex-end",gap:5,height:60}}>
{[72,68,74,80,78,Math.round(cur/1000)].map((v,i,arr)=>{const mx=Math.max(...arr);
<div key={i} style={{flex:1,display:"flex",flexDirection:"column",gap:3,alignIt
<div style={{width:"100%",height:`${(v/mx)*56}px`,borderRadius:"3px 3px 0 0",
<div style={{fontSize:8,color:"var(--text3)",fontFamily:"'JetBrains Mono',mon
</div>
);})}
</div>
</div>
<div className="card card-tap" style={{padding:18}} onClick={()=>setModal("diversity"
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marg
<div style={{fontWeight:700,fontSize:13}}>Diversität</div>
<span style={{fontSize:10,color:"var(--text3)",fontFamily:"'JetBrains Mono',monos
</div>
<div style={{fontSize:10,color:"var(--text3)",fontFamily:"'JetBrains Mono',monospac
<div style={{display:"flex",gap:14,alignItems:"center"}}>
<div style={{position:"relative"}}>
<svg width="88" height="88" viewBox="0 0 88 88" style={{transform:"rotate(-90de
<circle cx="44" cy="44" r="36" fill="none" stroke="rgba(255,255,255,0.07)" st
{(()=>{let off=0;const circ=2*Math.PI*36;return segs.map((s,i)=>{const d=(s.p
</svg>
<div style={{position:"absolute",inset:0,display:"flex",flexDirection:"column",
<div style={{fontWeight:800,fontSize:16,fontFamily:"'JetBrains Mono',monospac
<div style={{fontSize:8,color:"var(--text3)",fontFamily:"'JetBrains Mono',mon
</div>
</div>
<div style={{flex:1}}>
{segs.map(s=>(
<div key={s.label} style={{marginBottom:7}}>
<div style={{display:"flex",justifyContent:"space-between",fontSize:11,font
<span style={{display:"flex",alignItems:"center",gap:5}}><div style={{wid
<span style={{color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace
</div>
<div className="track"><div className="fill" style={{width:`${s.pct}%`,back
</div>
))}
</div>
</div>
</div>
</div>
<div className="g3" style={{marginBottom:12}}>
<div className="card card-tap stat" onClick={()=>setModal("bewertung")}>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marg
{[["Diversität","Überdurch.","chip-g"],["Performance","Überdurch.","chip-g"],["Risi
<div key={l} style={{display:"flex",justifyContent:"space-between",alignItems:"ce
<span style={{fontSize:11,color:"var(--text3)"}}>{l}</span><span className={`ch
</div>
))}
</div>
<div className="card card-tap stat" onClick={()=>setModal("fees")}>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marg
<div style={{fontSize:9,color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace
<div style={{fontSize:20,fontWeight:700,fontFamily:"'JetBrains Mono',monospace",mar
<span className="chip chip-gold" style={{fontSize:9}}>Optimierung möglich</span>
</div>
<div className="card stat">
<div style={{fontWeight:700,fontSize:12,marginBottom:10}}>Regionen</div>
{[["USA",62,"#3b82f6"],["Global",65,"#14b8a6"],["EU",22,"#f59e0b"]].map(([l,p,col])
<div key={l} style={{marginBottom:7}}>
<div style={{display:"flex",justifyContent:"space-between",fontSize:11,fontWeig
<div className="track"><div className="fill" style={{width:`${p}%`,background:c
</div>
))}
</div>
</div>
<div className="card" style={{overflow:"hidden",marginBottom:12}}>
<div style={{padding:"14px 16px 0",display:"flex",justifyContent:"space-between",alig
<div style={{fontWeight:700,fontSize:13}}>Assets ({enriched.length})</div>
<div style={{fontSize:10,color:"var(--text3)",fontFamily:"'JetBrains Mono',monospac
</div>
<div style={{overflowX:"auto"}}>
<table className="tbl">
<thead><tr><th>Asset</th><th>Plattform</th><th>Menge</th><th>Kauf</th><th>Kurs</t
<tbody>
{enriched.map(a=>{
const cv=a.amount*a.currentPrice,bv=a.amount*a.buyPrice,g=cv-bv,gp=(g/bv)*100
const p=prices[a.symbol], chgPct=p&&p.prevClose?((p.price-p.prevClose)/p.prev
return(
<tr key={a.symbol} onClick={()=>setActiveAsset(a)}>
<td><div style={{display:"flex",alignItems:"center",gap:9}}><AssetLogo sy
<td><PlatBadge platform={a.platform} size={20}/></td>
<td style={{fontWeight:600,fontFamily:"'JetBrains Mono',monospace"}}>{a.a
<td style={{color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace"}
<td><div style={{fontWeight:700,fontFamily:"'JetBrains Mono',monospace"}}
<td style={{fontWeight:700,fontFamily:"'JetBrains Mono',monospace"}}>{fmt
<td><div className={g>=0?"pos":"neg"} style={{fontWeight:700,fontSize:11,
</tr>
);
})}
</tbody>
</table>
</div>
</div>
<div className="card" style={{overflow:"hidden"}}>
<div style={{padding:"14px 16px 0",fontWeight:700,fontSize:13}}>Transaktionen</div>
<div style={{overflowX:"auto"}}>
<table className="tbl">
<thead><tr><th>Datum</th><th>Typ</th><th>Asset</th><th>Menge</th><th>Preis</th><t
<tbody>
{DEMO_TX.map((t,i)=>(
<tr key={i}>
<td style={{color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace"}}>
<td><span className={`chip ${t.type==="Kauf"?"chip-g":"chip-r"}`} style={{f
<td style={{fontWeight:700,fontFamily:"'JetBrains Mono',monospace"}}>{t.sym
<td style={{fontFamily:"'JetBrains Mono',monospace"}}>{t.amount}</td>
<td style={{fontFamily:"'JetBrains Mono',monospace"}}>{fmtEur(t.price)}</td
<td style={{fontWeight:600,fontFamily:"'JetBrains Mono',monospace"}}>{fmtEu
<td style={{color:"var(--text3)",fontFamily:"'JetBrains Mono',monospace"}}>
</tr>
))}
</tbody>
</table>
</div>
</div>
</div>
);
}
// ─── Live Feed ─────────────────────────────────────────────────────────────
function LiveFeed({isPremium}){
const [insight,setInsight]=useState(null);
const [loading,setLoading]=useState(false);
const [insightStep,setInsightStep]=useState(0);
const [news,setNews]=useState(DEMO_NEWS);
const [newsLoading,setNewsLoading]=useState(true);
const [newsLive,setNewsLive]=useState(false);
// ── Fetch real financial news via RSS feeds ────────────────────────────
useEffect(()=>{
const fetchNews=async()=>{
setNewsLoading(true);
const feeds=[
{url:"https://feeds.marketwatch.com/marketwatch/topstories/",tag:"MARKT"},
{url:"https://www.coindesk.com/arc/outboundfeeds/rss/",tag:"KRYPTO"},
{url:"https://feeds.reuters.com/reuters/businessNews",tag:"WIRTSCHAFT"},
];
const results=[];
for(const feed of feeds){
try{
const proxyUrl=`https://api.allorigins.win/get?url=${encodeURIComponent(feed.url)}`
const ctrl=new AbortController();
const timeout=setTimeout(()=>ctrl.abort(),5000);
const res=await fetch(proxyUrl,{signal:ctrl.signal});
clearTimeout(timeout);
const data=await res.json();
const xml=new DOMParser().parseFromString(data.contents,"text/xml");
const items=Array.from(xml.querySelectorAll("item")).slice(0,4);
items.forEach(item=>{
const title=item.querySelector("title")?.textContent||"";
const pubDate=item.querySelector("pubDate")?.textContent;
if(title){
const ageMs=pubDate?Date.now()-new Date(pubDate).getTime():0;
const ageMin=Math.floor(ageMs/60000);
const timeLabel=ageMin<1?"jetzt":ageMin<60?`${ageMin} Min.`:ageMin<1440?`${Math
const lower=title.toLowerCase();
const sentiment=/surge|gain|rally|rise|jump|high|boost|growth/.test(lower)?"bul
:/drop|fall|crash|loss|decline|low|cut|risk/.test(lower)?"bearish":"neutral";
results.push({time:timeLabel,tag:feed.tag,text:title,sentiment});
}
});
}catch(e){/* feed failed, skip */}
}
if(results.length>0){
results.sort(()=>Math.random()-0.5);
setNews(results.slice(0,10));
setNewsLive(true);
} else {
setNews(DEMO_NEWS);
setNewsLive(false);
}
setNewsLoading(false);
};
fetchNews();
const iv=setInterval(fetchNews,180000); // refresh every 3 min
return()=>clearInterval(iv);
},[]);
const getInsight=async()=>{
setLoading(true);setInsight(null);setInsightStep(0);
const sd=ms=>new Promise(r=>setTimeout(r,ms));
sd(600).then(()=>setInsightStep(1));sd(1500).then(()=>setInsightStep(2));sd(2600).then(()
try{
const headlines=news.slice(0,6).map(n=>n.text).join(". ");
const[raw]=await Promise.all([callAI(`Makro-Analyst. JSON: {"stimmung":"bullish|bearish
setInsightStep(4);setInsight(JSON.parse(raw.replace(/```json|```/g,"").trim()));
}catch{setInsight({error:true});}
await sd(200);setLoading(false);
};
return(
<div>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marg
<div><h1 className="pg-title" style={{background:"linear-gradient(135deg,#fff 60%,rgb
{isPremium&&<button className="btn btn-blue btn-sm" onClick={getInsight} disabled={lo
</div>
{isPremium&&loading&&<div className="card" style={{overflow:"hidden",marginBottom:12}}>
{isPremium&&insight&&!insight.error&&!loading&&(
<div className="card ai" style={{padding:18,marginBottom:12,background:"rgba(255,255,
<div style={{display:"flex",gap:10}}>
<div style={{flex:1}}>
<div style={{display:"flex",gap:8,alignItems:"center",marginBottom:8,flexWrap:"
<span style={{fontWeight:700,fontSize:13,fontFamily:"'JetBrains Mono',monospa
<span className={`chip ${insight.stimmung==="bullish"?"chip-g":insight.stimmu
</div>
<div style={{fontSize:13,color:"var(--text2)",lineHeight:1.65,marginBottom:10,f
<div className="g2" style={{gap:8}}>
{[["Entwicklung",insight.wichtigste_entwicklung],["Portfolio",insight.portfol
<div key={l} style={{padding:"9px 11px",background:"var(--bg2)",borderRadiu
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform
<div style={{fontSize:12,lineHeight:1.5,color:"rgba(255,255,255,0.7)",fon
</div>
))}
</div>
</div>
</div>
</div>
)}
{!isPremium&&(
<div style={{padding:"12px 14px",background:"rgba(245,158,11,0.08)",border:"1px solid
<span style={{flex:1,fontSize:13,color:"var(--gold)",fontFamily:"'Space Grotesk',sa
</div>
)}
<div style={{display:"flex",alignItems:"center",gap:8,marginBottom:10}}>
{newsLoading?(
<><div className="spn"/><span style={{fontSize:11,color:"var(--text3)",fontFamily:"
):(
<>
<span className="ldot" style={{background:newsLive?"#22c55e":"var(--gold)"}}/>
<span style={{fontSize:11,color:"var(--text3)",fontFamily:"'JetBrains Mono',monos
{newsLive?"Live-Feed aktiv":"Demo-Daten (Live-Feed nicht erreichbar)"}
</span>
</>
)}
</div>
<div style={{display:"flex",flexDirection:"column",gap:8}}>
{news.map((n,i)=>(
<div key={i} className={`card au d${Math.min(i+1,5)}`} style={{padding:"12px 16px",
<span className={`tag ${n.sentiment==="bullish"?"tag-bull":n.sentiment==="bearish
<div style={{flex:1}}><div style={{fontSize:13,fontWeight:500,lineHeight:1.5,font
<span style={{fontSize:14}}>{n.sentiment==="bullish"?"▲":n.sentiment==="bearish"?
</div>
))}
</div>
</div>
);
}
// ─── Chart Analysis ────────────────────────────────────────────────────────
function ChartAnalysis({isPremium,onTogglePremium}){
const [basicAsset,setBasicAsset]=useState(""), [basicRes,setBasicRes]=useState(null), [bas
const [input,setInput]=useState(""), [result,setResult]=useState(null), [loading,setLoading
const runBasic=async()=>{
setBasicLoad(true);setBasicRes(null);setBasicStep(0);
const sd=ms=>new Promise(r=>setTimeout(r,ms));
sd(800).then(()=>setBasicStep(1));sd(2000).then(()=>setBasicStep(2));sd(3400).then(()=>se
const[raw]=await Promise.all([callAI(`Finanzanalyst. JSON: {"asset":"Name","trend":"steig
try{setBasicStep(5);setBasicRes(JSON.parse(raw.replace(/```json|```/g,"").trim()));}catch
await sd(200);setBasicLoad(false);
};
const runPremium=async()=>{
setLoading(true);setResult(null);setLoadStep(0);
const sd=ms=>new Promise(r=>setTimeout(r,ms));
sd(600).then(()=>setLoadStep(1));sd(1300).then(()=>setLoadStep(2));sd(2100).then(()=>setL
const[raw]=await Promise.all([callAI(`Technischer Finanzanalyst. JSON: {"asset":"Name","t
try{setLoadStep(5);setResult(JSON.parse(raw.replace(/```json|```/g,"").trim()));}catch{se
await sd(200);setLoading(false);
};
return(
<div>
<h1 className="pg-title" style={{background:"linear-gradient(135deg,#fff 60%,rgba(34,19
<p className="pg-sub">{isPremium?"Tiefenanalyse · Szenarien · Strategie":"Trendanalyse
<div className="card" style={{marginBottom:14,overflow:"hidden"}}>
<div style={{padding:"16px 18px 0"}}>
<div style={{display:"flex",gap:8,alignItems:"center",marginBottom:12}}>
<div style={{fontWeight:700,fontSize:13}}>Trendanalyse</div>
<span className="chip chip-b" style={{fontSize:9}}>Basic</span>
</div>
<div style={{display:"flex",gap:8,flexWrap:"wrap",marginBottom:basicLoad||basicRes?
<input className="inp" placeholder="z.B. Bitcoin, AAPL, ETH/USDT…" value={basicAs
<button className="btn btn-blue btn-sm" onClick={runBasic} disabled={basicLoad||!
</div>
</div>
{basicLoad&&<div style={{borderTop:"1px solid var(--border)"}}><HelixLoader label={`A
{basicRes&&!basicRes.error&&!basicLoad&&(
<div style={{padding:"14px 18px 18px",borderTop:"1px solid var(--border)"}}>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",ma
<div style={{fontWeight:800,fontSize:16,fontFamily:"'JetBrains Mono',monospace"
<div style={{display:"flex",gap:6,flexWrap:"wrap"}}>
<span className={`chip ${basicRes.simplerTrend==="positiv"?"chip-g":basicRes.
{basicRes.ma_signal&&<span className={`chip ${basicRes.ma_signal==="über MA50
</div>
</div>
<div style={{fontSize:13,color:"var(--text2)",lineHeight:1.7,marginBottom:12,padd
{(basicRes.rsi||basicRes.volumen_signal)&&(
<div className="g2" style={{gap:8,marginBottom:10}}>
{basicRes.rsi&&<div style={{padding:"9px 12px",background:"var(--bg2)",border
{basicRes.volumen_signal&&<div style={{padding:"9px 12px",background:"var(--b
</div>
)}
{(basicRes.support||basicRes.widerstand)&&(
<div className="g2" style={{gap:8,marginBottom:10}}>
{basicRes.support&&<div style={{padding:"9px 12px",background:"rgba(168,168,1
{basicRes.widerstand&&<div style={{padding:"9px 12px",background:"rgba(239,68
</div>
)}
</div>
{basicRes.kurzfrist_ausblick&&<div style={{padding:"10px 13px",background:"rgba(2
)}
{basicRes?.error&&!basicLoad&&<div style={{margin:"0 18px 18px",padding:"9px 13px",ba
</div>
{!isPremium?(
<div className="card" style={{padding:40,textAlign:"center"}}>
<div style={{width:44,height:44,borderRadius:10,background:"var(--bg2)",border:"1px
<h3 style={{fontFamily:"'JetBrains Mono',monospace",fontSize:16,marginBottom:8}}>KI
<p style={{color:"var(--text3)",maxWidth:360,margin:"0 auto 20px",fontSize:13,lineH
<div style={{display:"flex",justifyContent:"center"}}>
<WithdrawalUpgradeButton onTogglePremium={()=>onTogglePremium()} onClose={()=>{}}
</div>
</div>
):(
<div className="g2" style={{gap:14}}>
<div>
<div className="card" style={{padding:18}}>
<div style={{display:"flex",gap:8,alignItems:"center",marginBottom:14}}>
<div style={{fontWeight:700,fontSize:13}}>KI-Tiefenanalyse</div>
<span className="chip chip-pur" style={{fontSize:9}}>Premium</span>
</div>
<label className="lbl">Asset</label>
<input className="inp" placeholder="Bitcoin, AAPL, ETH/USDT…" value={input} onC
<button className="btn btn-blue" style={{width:"100%",justifyContent:"center"}}
</div>
</div>
<div>
{!result&&!loading&&<div className="card" style={{padding:40,textAlign:"center",c
{loading&&<div className="card" style={{overflow:"hidden"}}><HelixLoader label="T
{result&&!result.error&&(
<div style={{display:"flex",flexDirection:"column",gap:10}}>
<div className="card as" style={{padding:"16px 18px"}}>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"cent
<div style={{fontWeight:800,fontSize:17,fontFamily:"'JetBrains Mono',mono
<div style={{display:"flex",gap:6}}>
<span className={`chip ${result.trend==="bullish"?"chip-g":result.trend
<span className={`chip ${result.risiko==="niedrig"?"chip-g":result.risi
</div>
</div>
{[["Technische Analyse",result.technisch],["Volumen",result.volumen],["Mark
<div key={l} style={{marginBottom:12}}>
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransfo
<div style={{fontSize:13,color:"var(--text2)",lineHeight:1.65,fontFamil
</div>
))}
</div>
<div className="g2" style={{gap:10}}>
<div style={{padding:13,background:"rgba(168,168,179,0.05)",borderRadius:8,
<div style={{padding:13,background:"rgba(239,68,68,0.06)",borderRadius:8,bo
</div>
<div style={{padding:14,background:"rgba(255,255,255,0.04)",borderRadius:8,bo
<div style={{padding:"8px 12px",background:"var(--bg2)",borderRadius:7,border
</div>
)}
</div>
</div>
)}
</div>
);
}
// ─── Optimization ──────────────────────────────────────────────────────────
function Optimization({isPremium,portfolio,onTogglePremium}){
const [result,setResult]=useState(null);
const [loading,setLoading]=useState(false);
const [optStep,setOptStep]=useState(0);
const [liveP,setLiveP]=useState({});
// ── Fundamentals (intrinsic value, D/E ratio, brand score) ─────────────────
const FUND = {
AAPL: {iv:218, pe:28, de:1.8, brand:9, name:"Apple"},
MSFT: {iv:375, pe:34, de:0.4, brand:9, name:"Microsoft"},
NVDA: {iv:175, pe:38, de:0.3, brand:8, name:"NVIDIA"},
TSLA: {iv:145, pe:55, de:0.8, brand:7, name:"Tesla"},
AMZN: {iv:195, pe:40, de:0.7, brand:9, name:"Amazon"},
GOOGL:{iv:165, pe:22, de:0.1, brand:9, name:"Alphabet"},
META: {iv:490, pe:25, de:0.1, brand:8, name:"Meta"},
SPY: {iv:680, pe:21, de:0, brand:10,name:"S&P 500 ETF"},
VWRL: {iv:112, pe:18, de:0, brand:10,name:"Vanguard World ETF"},
};
// ATH & RSI estimates for crypto
const CRYPTO = {
BTC: {ath:108000, rsiEst:44, name:"Bitcoin"},
ETH: {ath:4800, rsiEst:38, name:"Ethereum"},
SOL: {ath:260, rsiEst:35, name:"Solana"},
BNB: {ath:720, rsiEst:52, name:"BNB", regRisk:true},
};
// ── Fetch live prices from Yahoo Finance ─────────────────────────────────
const fetchLive = async (symbols) => {
const prices = {};
await Promise.all(symbols.map(async sym => {
try {
const ticker = {BTC:"BTC-USD",ETH:"ETH-USD",SOL:"SOL-USD",BNB:"BNB-USD"}[sym] || sym;
const url = `https://query1.finance.yahoo.com/v8/finance/chart/${ticker}?interval=1d&
const proxy = `https://corsproxy.io/?${encodeURIComponent(url)}`;
const ctrl = new AbortController();
setTimeout(()=>ctrl.abort(), 4000);
const res = await fetch(proxy, {signal:ctrl.signal});
const raw = await res.json();
const p = raw?.chart?.result?.[0]?.meta?.regularMarketPrice;
if (p) prices[sym] = p;
} catch {}
}));
return prices;
};
// ── Core analysis engine ──────────────────────────────────────────────────
const analyse = (items, live) => {
const assets = items.map(a => ({
...a,
currentPrice: live[a.symbol] || a.currentPrice || a.buyPrice,
gain: ((live[a.symbol]||a.currentPrice||a.buyPrice) - a.buyPrice) / a.buyPrice * }));
100,
const total = assets.reduce((s,a)=>s+a.amount*a.currentPrice,0) || 1;
const cryptoAssets = assets.filter(a=>CRYPTO[a.symbol]);
const stockAssets = assets.filter(a=>FUND[a.symbol]);
const cryptoPct = cryptoAssets.reduce((s,a)=>s+a.amount*a.currentPrice,0)/total*100;
const suggestions = [];
// ── Aktien: Margin of Safety ────────────────────────────────────────────
stockAssets.forEach(a => {
const f = FUND[a.symbol];
if (!f) return;
const mos = (f.iv - a.currentPrice) / f.iv * 100;
const posValue = a.amount * a.currentPrice;
const posWeight = posValue / total * 100;
if (mos >= 20) {
suggestions.push({
typ:"kaufen", asset:a.symbol,
menge:`+${Math.round(posValue*0.3/a.currentPrice)} Stück`,
kurs:`$${a.currentPrice.toFixed(0)}`,
ziel:`MoS ${mos.toFixed(0)}% — Innerer Wert $${f.iv}`,
investition:`~€${Math.round(posValue*0.3)}`,
priorität:mos>=30?"hoch":"mittel",
grund:`${f.name}: MoS = ($${f.iv}−$${a.currentPrice.toFixed(0)})/$${f.iv}×100 = ${m
});
} else if (mos < -20 && a.gain > 25) {
suggestions.push({
typ:"reduzieren", asset:a.symbol,
menge:"25–30%",
kurs:`$${a.currentPrice.toFixed(0)}`,
ziel:`Gewinne sichern — ${Math.abs(mos).toFixed(0)}% über innerem Wert`,
erlös:`~€${Math.round(posValue*0.27)}`,
priorität:Math.abs(mos)>40?"hoch":"mittel",
grund:`${f.name}: Kurs $${a.currentPrice.toFixed(0)} liegt ${Math.abs(mos).toFixed(
});
}
});
// ── Crypto: RSI + ATH-Abstand ───────────────────────────────────────────
cryptoAssets.forEach(a => {
const d = CRYPTO[a.symbol];
if (!d) return;
const fromATH = (a.currentPrice - d.ath) / d.ath * 100;
const posValue = a.amount * a.currentPrice;
if (d.regRisk && posValue > total * 0.05) {
suggestions.push({
typ:"verkaufen", asset:a.symbol,
menge:"100%",
kurs:`$${a.currentPrice.toFixed(0)}`,
ziel:"Regulatorisches Risiko eliminieren",
erlös:`~€${Math.round(posValue)}`,
priorität:"hoch",
grund:`${d.name}: Aktive SEC/CFTC-Verfahren gegen Binance. Hohe regulatorische Unsi
});
} else if (!d.regRisk && d.rsiEst < 40 && fromATH < -50) {
suggestions.push({
typ:"kaufen", asset:a.symbol,
menge:"Position aufbauen",
kurs:`$${a.currentPrice.toFixed(0)}`,
ziel:`${Math.abs(fromATH).toFixed(0)}% unter ATH — RSI überverkauft`,
investition:"~€500–1.000",
priorität:d.rsiEst<35?"hoch":"mittel",
grund:`${d.name}: RSI(14) ≈${d.rsiEst} (überverkauft <40). ${Math.abs(fromATH).toFi
});
} else if (!d.regRisk && a.gain > 50 && d.rsiEst > 60) {
suggestions.push({
typ:"verkaufen", asset:a.symbol,
menge:"40–50%",
kurs:`$${a.currentPrice.toFixed(0)}`,
ziel:"Teilgewinnmitnahme nach starker Rally",
erlös:`~€${Math.round(posValue*0.45)}`,
priorität:"mittel",
grund:`${d.name}: +${a.gain.toFixed(0)}% seit Kauf, RSI ≈${d.rsiEst} (erhöht). 40–5
});
}
});
// ── Fehlende Diversifikation ────────────────────────────────────────────
const hasGold = assets.some(a=>["XGLD","GLD","IAU"].includes(a.symbol));
const hasGlobal = assets.some(a=>["VWRL","VTI","IWDA"].includes(a.symbol));
if (!hasGold) {
suggestions.push({
typ:"kaufen", asset:"Gold ETC (XGLD)",
menge:"3–5% Portfolio",
kurs:"~$32",
ziel:"Inflationsschutz & negative Tech-Korrelation",
investition:`~€${Math.round(total*0.04)}`,
priorität:"mittel",
grund:"Kein Gold-Exposure. Gold korreliert negativ zu Tech-Aktien in Krisen (ρ });
≈ −0.3
}
if (!hasGlobal) {
suggestions.push({
typ:"kaufen", asset:"MSCI World ETF (VWRL)",
menge:"5–8% Portfolio",
kurs:"~$118",
ziel:"Globale Diversifikation, niedrige Kosten (TER 0.22%)",
investition:`~€${Math.round(total*0.06)}`,
priorität:"niedrig",
grund:"Breite Marktabdeckung (>3.500 Aktien, 23 Länder). Reduziert Klumpenrisiko bei
});
}
// ── Portfolio Score ─────────────────────────────────────────────────────
let score = 7;
if (cryptoPct > 40) score -= 2;
else if (cryptoPct > 25) score -= 1;
if (hasGold) score += 0.5;
if (stockAssets.some(a=>FUND[a.symbol]?.de < 0.5)) score += 0.5;
score = Math.max(3, Math.min(9, Math.round(score)));
const topGainer = [...assets].sort((a,b)=>b.gain-a.gain)[0];
const topStock = stockAssets.find(a=>FUND[a.symbol]?.brand>=9);
return {
score,
bewertung: score>=8?"Überdurchschnittlich":score>=6?"Solide":"Optimierungsbedarf",
stärken:[
topGainer ? `${topGainer.symbol} Topperformer: +${topGainer.gain.toFixed(0)}% seit Ka
topStock ? `${FUND[topStock.symbol].name}: D/E ${FUND[topStock.symbol].de} (niedrig),
`${assets.length} Positionen über ${[...new Set(assets.map(a=>a.sector))].length} Sek
],
risiken:[
cryptoPct>25 ? `Crypto-Anteil ${cryptoPct.toFixed(0)}% > empfohlene 20% — erhöhte Vol
assets.some(a=>FUND[a.symbol]?.pe>35) ? `Hohe KGVs: ${assets.filter(a=>FUND[a.symbol]
!hasGold ? "Kein Gold/Rohstoff-Exposure — fehlendes Inflationsschutz-Element" : "Rohs
],
vorschläge: suggestions.slice(0,6),
livePrices: Object.fromEntries(assets.map(a=>[a.symbol, a.currentPrice])),
makro:"Fed hält Zinsen Mai 2026 bei 4.25–4.5%. Erste Senkung frühestens Q4 2026. EZB be
geopolitik:"US-China Tech-Exportbeschränkungen (Halbleiter, KI-Chips) belasten NVDA/AMD
disclaimer:"Keine Anlageberatung. MoS-Berechnung basiert auf Schätzwerten. Eigene Due D
};
};
const run = async () => {
setLoading(true); setResult(null); setOptStep(0);
const sd = ms => new Promise(r=>setTimeout(r,ms));
const items = portfolio.length>0 ? portfolio : DEMO_ASSETS;
const symbols = [...new Set(items.map(a=>a.symbol))];
setOptStep(1);
// Fetch live prices
const live = await fetchLive(symbols);
setLiveP(live);
setOptStep(2); await sd(400);
setOptStep(3); await sd(400);
setOptStep(4); await sd(300);
setOptStep(5); await sd(300);
try {
const analysis = analyse(items, live);
setOptStep(6);
await sd(300);
setResult(analysis);
} catch(e) {
console.error(e);
setResult({error:true});
}
setLoading(false);
};
if(!isPremium) return(
<div>
<h1 className="pg-title" style={{background:"linear-gradient(135deg,#fff 60%,rgba(34,19
<p className="pg-sub">Value-Investing · Margin of Safety · Live-Daten</p>
<div className="card" style={{padding:44,textAlign:"center"}}>
<h3 style={{fontFamily:"'JetBrains Mono',monospace",fontSize:16,marginBottom:8}}>Prem
<p style={{color:"var(--text3)",maxWidth:360,margin:"0 auto 22px",fontSize:13,lineHei
<div style={{display:"flex",justifyContent:"center"}}><WithdrawalUpgradeButton onTogg
</div>
</div>
);
return(
<div>
<h1 className="pg-title" style={{background:"linear-gradient(135deg,#fff 60%,rgba(34,19
<p className="pg-sub">Live-Kurse · Margin of Safety · Value-Investing</p>
{!result&&!loading&&(
<div className="card" style={{padding:44,textAlign:"center"}}>
<div style={{fontSize:32,marginBottom:12}}> </div>
<h3 style={{fontFamily:"'JetBrains Mono',monospace",fontSize:17,marginBottom:8}}>Po
<p style={{color:"var(--text3)",maxWidth:400,margin:"0 auto 8px",fontSize:13,lineHe
Lädt Live-Kurse · Berechnet Margin of Safety · Analysiert Risiken
</p>
<p style={{color:"var(--text3)",maxWidth:400,margin:"0 auto 26px",fontSize:11,lineH
MoS = (Innerer Wert − Kurs) / Innerer Wert × 100
</p>
</div>
<button className="btn btn-blue btn-lg" onClick={run}>Optimierung starten</button>
)}
{loading&&(
<div className="card" style={{overflow:"hidden"}}>
<HelixLoader
label="Portfolio analysieren"
sublabel="Live-Kurse · Margin of Safety · Value-Investing"
steps={["Live-Kurse abrufen","Margin of Safety berechnen","Crypto RSI analysieren
currentStep={optStep}
/>
</div>
)}
{result&&!result.error&&(
<div>
{/* Score + Live prices strip */}
<div className="g3" style={{marginBottom:12}}>
{[
{l:"Score",v:`${result.score}/10`,d:result.bewertung,pos:result.score>=7},
{l:"Stärken",v:result.stärken?.length,d:"identifiziert",pos:true},
{l:"Maßnahmen",v:result.vorschläge?.length,d:"empfohlen",pos:null},
].map((s,i)=>(
<div key={i} className="card stat">
<div className="stat-l">{s.l}</div>
<div className="stat-v">{s.v}</div>
<div className={`stat-d ${s.pos===true?"pos":s.pos===false?"neg":"neu"}`}>{s.
</div>
))}
</div>
{/* Live prices used */}
{Object.keys(result.livePrices||{}).length>0&&(
<div className="card" style={{padding:"10px 16px",marginBottom:12,overflowX:"auto
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform:"uppe
<span className="ldot" style={{marginRight:6}}/> Live-Kurse verwendet
</div>
<div style={{display:"flex",gap:16,flexWrap:"wrap"}}>
{Object.entries(result.livePrices).map(([sym,p])=>(
<div key={sym} style={{display:"flex",alignItems:"center",gap:6}}>
<AssetLogo symbol={sym} size={18}/>
<span style={{fontSize:11,fontFamily:"'JetBrains Mono',monospace",color:"
<span style={{fontSize:11,fontFamily:"'JetBrains Mono',monospace",color:"
</div>
))}
</div>
</div>
)}
<div className="g2" style={{gap:12,marginBottom:12}}>
<div className="card" style={{padding:18}}>
<div style={{fontWeight:700,fontSize:13,marginBottom:12}}>Portfolioanalyse</div
<div style={{fontSize:9,fontWeight:700,color:"var(--green)",textTransform:"uppe
{result.stärken?.map((s,i)=><div key={i} style={{fontSize:12,padding:"5px 0",bo
<div style={{fontSize:9,fontWeight:700,color:"var(--red)",textTransform:"upperc
{result.risiken?.map((r,i)=><div key={i} style={{fontSize:12,padding:"5px 0",bo
</div>
<div style={{display:"flex",flexDirection:"column",gap:10}}>
<div className="card" style={{padding:16,flex:1}}>
<div style={{fontWeight:700,fontSize:12,marginBottom:8,fontFamily:"'JetBrains
<div style={{fontSize:12,color:"var(--text2)",lineHeight:1.65,fontFamily:"'Sp
</div>
<div className="card" style={{padding:16}}>
<div style={{fontWeight:700,fontSize:12,marginBottom:8,fontFamily:"'JetBrains
<div style={{fontSize:12,color:"var(--text2)",lineHeight:1.65,fontFamily:"'Sp
</div>
</div>
</div>
{/* Suggestions */}
<div className="card" style={{padding:18,marginBottom:10}}>
<div style={{fontWeight:700,fontSize:13,marginBottom:12}}>Empfohlene Maßnahmen</d
<div style={{display:"flex",flexDirection:"column",gap:8}}>
{result.vorschläge?.map((v,i)=>{
const isVerkauf = v.typ==="verkaufen"||v.typ==="reduzieren";
const isKauf = v.typ==="kaufen"||v.typ==="erhöhen";
const bg = isVerkauf?"rgba(239,68,68,0.05)":isKauf?"rgba(34,197,94,0.0
const border = isVerkauf?"rgba(239,68,68,0.2)":isKauf?"rgba(34,197,94,0.2)
const icon = v.typ==="verkaufen"?"↓ SELL":v.typ==="kaufen"?"↑ BUY":v.typ
const iconCol = isVerkauf?"#ef4444":"#22c55e";
return(
<div key={i} style={{padding:"14px 16px",borderRadius:8,background:bg,borde
<div style={{display:"flex",justifyContent:"space-between",alignItems:"fl
<div style={{display:"flex",alignItems:"center",gap:8,flexWrap:"wrap"}}
<span style={{fontSize:9,fontWeight:800,color:iconCol,fontFamily:"'Je
<span style={{fontWeight:800,fontSize:14,fontFamily:"'JetBrains Mono'
{v.menge&&<span style={{fontSize:11,color:"rgba(255,255,255,0.4)",fon
<span className={`chip ${v.priorität==="hoch"?"chip-r":v.priorität===
</div>
<div style={{textAlign:"right"}}>
{v.kurs&&<div style={{fontSize:11,fontFamily:"'JetBrains Mono',monosp
{(v.erlös||v.investition)&&<div style={{fontSize:13,fontWeight:700,fo
</div>
</div>
<div style={{fontSize:10,color:"rgba(255,255,255,0.3)",fontFamily:"'JetBr
<div style={{fontSize:12,color:"rgba(255,255,255,0.65)",fontFamily:"'Spac
</div>
);
})}
</div>
</div>
{/* Cashflow summary */}
{(()=>{
const sells = result.vorschläge?.filter(v=>v.typ==="verkaufen"||v.typ==="reduzier
const buys = result.vorschläge?.filter(v=>v.typ==="kaufen"||v.typ==="erhöhen")||
return sells.length>0||buys.length>0 ? (
<div style={{padding:"14px 16px",background:"rgba(255,255,255,0.03)",borderRadi
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform:"up
<div className="g2" style={{gap:8}}>
<div style={{padding:"10px 12px",background:"rgba(239,68,68,0.05)",border:"
<div style={{fontSize:9,color:"rgba(239,68,68,0.7)",fontFamily:"'JetBrain
<div style={{fontSize:16,fontWeight:800,fontFamily:"'JetBrains Mono',mono
</div>
<div style={{padding:"10px 12px",background:"rgba(34,197,94,0.05)",border:"
<div style={{fontSize:9,color:"rgba(34,197,94,0.7)",fontFamily:"'JetBrain
<div style={{fontSize:16,fontWeight:800,fontFamily:"'JetBrains Mono',mono
</div>
</div>
</div>
) : null;
})()}
<div style={{padding:"8px 12px",background:"var(--bg3)",borderRadius:7,border:"1px
<button className="btn btn-white btn-sm" onClick={()=>setResult(null)}>Neue Analyse
</div>
)}
{result?.error&&(
<div className="card" style={{padding:20}}>
<p style={{color:"var(--red)",fontSize:13,marginBottom:10}}>Analyse fehlgeschlagen.
<button className="btn btn-white btn-sm" onClick={()=>setResult(null)}>Zurück</butt
</div>
)}
</div>
);
}
// ─── Connect View ──────────────────────────────────────────────────────────
function ConnectView({engine,isPremium}){
const [sel,setSel]=useState(null), [creds,setCreds]=useState({}), [status,setStatus]=useSta
const plat=PLATFORMS.find(p=>p.id===sel);
const isConn=id=>engine.connections.some(c=>c.platform===id);
const allowed=isPremium?PLATFORMS:PLATFORMS.slice(0,3), locked=isPremium?[]:PLATFORMS.slice
const doConnect=async()=>{setStatus("connecting");try{await engine.connect(sel,creds);setSt
const handleCSV=e=>{const file=e.target.files?.[0];if(!file)return;const reader=new FileRea
return(
<div>
<h1 className="pg-title" style={{background:"linear-gradient(135deg,#fff 60%,rgba(34,19
<p className="pg-sub">{isPremium?"Alle 5 Plattformen":"Basic: 3 Plattformen · Premium:
<div className="sec-b" style={{marginBottom:16}}>
<span style={{fontSize:18,flexShrink:0}}> </span>
<div><div style={{fontWeight:700,fontSize:12,color:"var(--green)",marginBottom:2}}>Si
</div>
<div className="g2" style={{gap:14}}>
<div>
<div className="card" style={{padding:16,marginBottom:10}}>
<div style={{fontWeight:700,fontSize:12,marginBottom:10,fontFamily:"'JetBrains Mo
<div style={{display:"flex",flexDirection:"column",gap:7}}>
{allowed.map(p=>(
<div key={p.id} onClick={()=>{setSel(p.id);setStatus(null);setCreds({});}} cl
<PlatBadge platform={p.id} size={24}/><div style={{flex:1}}><div style={{fo
{isConn(p.id)&&<span className="sc sc-ok" style={{fontSize:9}}>✓</span>}
{engine.syncStatus[p.id]==="syncing"&&<div className="spn"/>}
</div>
))}
{locked.length>0&&(<>{<div style={{fontSize:9,fontWeight:700,color:"var(--text3
</div>
</div>
<div className="card" style={{padding:16}}>
<div style={{fontWeight:700,fontSize:12,marginBottom:8,fontFamily:"'JetBrains Mon
<div style={{fontSize:10,color:"var(--text3)",marginBottom:8,fontFamily:"'JetBrai
{csvOk>0&&<div style={{padding:"7px 10px",background:"var(--gbg)",border:"1px sol
{csvErr&&<div style={{padding:"7px 10px",background:"var(--rbg)",borderRadius:6,f
<label className="drop" style={{display:"block"}}><input type="file" accept=".csv
</div>
</div>
<div>
{!sel?<div className="card" style={{padding:40,textAlign:"center",color:"var(--text
<div className="card as" style={{padding:22}}>
<div style={{display:"flex",alignItems:"center",gap:12,marginBottom:16}}><PlatB
{isConn(sel)?(
<div>
<div style={{padding:"10px 12px",background:"var(--gbg)",border:"1px <div style={{display:"flex",gap:8}}><button className="btn btn-white </div>
):(
<div>
solid
btn-sm
<div style={{padding:"9px 12px",background:"var(--bg2)",borderRadius:7,bord
{plat?.fields.map(f=>(
<div key={f} style={{marginBottom:10}}>
<label className="lbl">{f.replace(/_/g," ").toUpperCase()}</label>
<div style={{position:"relative"}}>
<input className={`inp inp-mono ${creds[f]&&creds[f].length>8?"inp-ok
{creds[f]&&creds[f].length>8&&<div style={{position:"absolute",right:
</div>
</div>
))}
<div style={{padding:"7px 10px",background:"var(--goldbg)",border:"1px soli
{status==="ok"&&<div style={{padding:"9px 12px",background:"var(--gbg)",bor
{status==="error"&&<div style={{padding:"9px 12px",background:"var(--rbg)",
<button className="btn btn-blue" style={{width:"100%",justifyContent:"cente
</div>
)}
</div>
)}
</div>
</div>
</div>
);
}
// ─── Landing ───────────────────────────────────────────────────────────────
// ── Scroll reveal hook ─────────────────────────────────────────────────────
function useScrollReveal(){
useEffect(()=>{
const els=document.querySelectorAll('.scroll-reveal,.scroll-reveal-left,.scroll-reveal-no
const obs=new IntersectionObserver((entries)=>{
entries.forEach(e=>{ if(e.isIntersecting) e.target.classList.add('visible'); });
},{threshold:0.12,rootMargin:"0px 0px -40px 0px"});
els.forEach(el=>obs.observe(el));
return()=>obs.disconnect();
},[]);
}
// ── Cinematic particle system ─────────────────────────────────────────────
function ParticleField(){
const ref=useRef(null);
useEffect(()=>{
const canvas=ref.current; if(!canvas)return;
const ctx=canvas.getContext('2d');
let W=canvas.offsetWidth, H=canvas.offsetHeight;
canvas.width=W; canvas.height=H;
const particles=Array.from({length:60},()=>({
x:Math.random()*W, y:Math.random()*H,
vx:(Math.random()-.5)*0.3, vy:(Math.random()-.5)*0.3,
r:Math.random()*1.5+0.3,
op:Math.random()*0.4+0.1,
}));
let raf;
const draw=()=>{
ctx.clearRect(0,0,W,H);
particles.forEach(p=>{
p.x+=p.vx; p.y+=p.vy;
if(p.x<0)p.x=W; if(p.x>W)p.x=0;
if(p.y<0)p.y=H; if(p.y>H)p.y=0;
ctx.beginPath(); ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
ctx.fillStyle=`rgba(168,168,179,${p.op})`; ctx.fill();
});
// Draw connections
for(let i=0;i<particles.length;i++){
for(let j=i+1;j<particles.length;j++){
const dx=particles[i].x-particles[j].x, dy=particles[i].y-particles[j].y;
const d=Math.sqrt(dx*dx+dy*dy);
if(d<120){
ctx.beginPath();
ctx.moveTo(particles[i].x,particles[i].y);
ctx.lineTo(particles[j].x,particles[j].y);
ctx.strokeStyle=`rgba(168,168,179,${0.04*(1-d/120)})`;
ctx.lineWidth=0.5; ctx.stroke();
}
}
}
raf=requestAnimationFrame(draw);
};
draw();
const resize=()=>{ W=canvas.offsetWidth;H=canvas.offsetHeight;canvas.width=W;canvas.heigh
window.addEventListener('resize',resize);
return()=>{ cancelAnimationFrame(raf); window.removeEventListener('resize',resize); };
},[]);
return <canvas ref={ref} style={{position:'absolute',inset:0,width:'100%',height:'100%',opa
}
// ── Animated grid background ──────────────────────────────────────────────
function GridBg(){
return(
<div style={{position:'absolute',inset:0,overflow:'hidden',pointerEvents:'none'}}>
<svg width="100%" height="100%" style={{position:'absolute',inset:0,opacity:.04}}>
<defs>
<pattern id="grid" width="60" height="60" patternUnits="userSpaceOnUse">
<path d="M 60 0 L 0 0 0 60" fill="none" stroke="rgba(34,197,94,1)" strokeWidth="0
</pattern>
</defs>
<rect width="100%" height="100%" fill="url(#grid)"/>
</svg>
{/* Radial glow center */}
<div style={{position:'absolute',top:'40%',left:'50%',transform:'translate(-50%,-50%)',
{/* Top left glow */}
<div style={{position:'absolute',top:-100,left:-100,width:400,height:400,background:'ra
</div>
);
}
function Landing({onEnter,onLogin}){
const [scrollY,setScrollY]=useState(0);
const [showWithdrawal,setShowWithdrawal]=useState(null);
useScrollReveal();
useEffect(()=>{
const fn=()=>setScrollY(window.scrollY);
window.addEventListener('scroll',fn,{passive:true});
return()=>window.removeEventListener('scroll',fn);
},[]);
const chfRot=Math.sin(scrollY*0.008)*16;
/* ── MiniChart — grows left to right ── */
function MiniChart(){
const ref=useRef(null);
useEffect(()=>{
const canvas=ref.current;if(!canvas)return;
const W=canvas.offsetWidth,H=90;
const dpr=window.devicePixelRatio||1;
canvas.width=W*dpr;canvas.height=H*dpr;
const ctx=canvas.getContext('2d');ctx.scale(dpr,dpr);
ctx.clearRect(0,0,W,H);
const pts=[0.12,0.18,0.22,0.28,0.35,0.42,0.38,0.52,0.60,0.68,0.74,0.85];
const toX=i=>W*i/(pts.length-1),toY=v=>(1-v)*(H-4)+2;
const g=ctx.createLinearGradient(0,0,0,H);
g.addColorStop(0,'rgba(34,197,94,0.22)');g.addColorStop(1,'rgba(34,197,94,0)');
ctx.beginPath();pts.forEach((v,i)=>{const x=toX(i),y=toY(v);i===0?ctx.moveTo(x,y):ctx.l
ctx.lineTo(W,H);ctx.lineTo(0,H);ctx.closePath();ctx.fillStyle=g;ctx.fill();
ctx.beginPath();pts.forEach((v,i)=>{const x=toX(i),y=toY(v);i===0?ctx.moveTo(x,y):ctx.l
ctx.strokeStyle='#22c55e';ctx.lineWidth=2.5;ctx.lineJoin='round';ctx.lineCap='round';ct
const lx=toX(pts.length-1),ly=toY(pts[pts.length-1]);
ctx.beginPath();ctx.arc(lx,ly,4,0,Math.PI*2);ctx.fillStyle='#22c55e';ctx.fill();
ctx.beginPath();ctx.arc(lx,ly,9,0,Math.PI*2);ctx.fillStyle='rgba(34,197,94,0.18)';ctx.f
},[]);
return <canvas ref={ref} style={{width:'100%',height:90,display:'block'}}/>;
}
/* ── CHF Banknote ── */
function CHFNote({rotation=0}){
const W=320,H=190;
const bg=[],glob=[];
for(let y=5;y<H;y+=6)for(let x=5;x<W;x+=6)bg.push({x,y,op:0.02+Math.sin(x*0.22+y*0.15)*0.
const gx=218,gy=H/2,gr=62;
for(let dy=-gr;dy<=gr;dy+=4.5)for(let dx=-gr;dx<=gr;dx+=4.5){
const d=Math.sqrt(dx*dx+dy*dy);if(d<gr){const onL=(d%7)<2.5,isE=d>gr*0.82;glob.push({x:
return(
<div style={{width:'100%',maxWidth:W,aspectRatio:`${W}/${H}`,background:'linear-gradien
<svg width="100%" height="100%" viewBox={`0 0 ${W} ${H}`} style={{position:'absolute'
{bg.map((d,i)=><circle key={i} cx={d.x} cy={d.y} r={1.4} fill="#1B4332" opacity={d.
<rect x={0} y={0} width={70} height={H} fill="rgba(107,184,138,0.35)"/>
<rect x={W*.55} y={0} width={W*.45} height={H} fill="rgba(142,202,160,0.2)"/>
<rect x={0} y={H*.72} width={W*.55} height={H*.28} fill="rgba(255,255,255,0.35)"/>
{glob.map((d,i)=><circle key={"g"+i} cx={d.x} cy={d.y} r={d.r} fill="#0d2b1a" opaci
<circle cx={gx} cy={gy} r={gr} fill="none" stroke="#0d2b1a" strokeWidth={1} opacity
<rect x={20} y={16} width={20} height={5} rx={1} fill="#155c35" opacity={.9}/>
<rect x={27} y={9} width={5} height={20} rx={1} fill="#155c35" opacity={.9}/>
{[0,1,2,3,4,5].map(i=><rect key={i} x={4} y={50+i*6} width={8} height={2} rx={1} fi
<text x={28} y={H-22} fontFamily="serif" fontSize="44" fontWeight="700" fill="#155c
<text x={46} y={28} fontFamily="monospace" fontSize="10" fontWeight="700" fill="#15
<text x={W-8} y={18} fontFamily="sans-serif" fontSize="7.5" fontWeight="600" fill="
<text x={28} y={H-8} fontFamily="sans-serif" fontSize="6.5" fontWeight="700" fill="
</svg>
<div style={{position:'absolute',inset:0,background:`linear-gradient(${100+rotation*3
</div>
);
}
return(
<div style={{background:'#050505',minHeight:'100vh',overflowX:'hidden',color:'#fff'}}>
{showWithdrawal&&<WithdrawalModal plan={showWithdrawal} onAccept={()=>{setShowWithdrawa
{/* Nav */}
<nav className="nav" style={{background:'rgba(5,5,5,0.7)'}}>
<div className="nav-inner">
<div className="logo">YBU<span className="logo-acc"> Finance</span></div>
<div style={{display:'flex',gap:8,alignItems:'center'}}>
<button className="nav-btn" onClick={()=>document.getElementById('pricing')?.scro
<button className="nav-btn" onClick={onLogin||onEnter}>Login</button>
<button className="nav-btn-solid" onClick={onEnter}>Start →</button>
</div>
</div>
</nav>
<div style={{paddingTop:56}}><TickerBar/></div>
{/* ── HERO — Oxaley style: huge text + gradient orb ── */}
<div style={{position:'relative',minHeight:'100vh',overflow:'hidden',display:'flex',ali
{/* Big teal/green gradient orb like Oxaley's purple */}
<div style={{position:'absolute',top:'-10%',left:'-5%',width:'70vw',height:'70vw',max
background:'radial-gradient(ellipse at 40% 50%,rgba(20,184,166,0.35) 0%,rgba(16,185
borderRadius:'50%',filter:'blur(60px)',pointerEvents:'none',
transform:`translateY(${scrollY*0.06}px)`}}/>
<div style={{position:'absolute',top:'5%',right:'-10%',width:'40vw',height:'40vw',max
background:'radial-gradient(circle,rgba(6,182,212,0.15) 0%,transparent 70%)',
borderRadius:'50%',filter:'blur(40px)',pointerEvents:'none',
transform:`translateY(${scrollY*0.04}px)`}}/>
{/* Grid */}
<div style={{position:'absolute',inset:0,backgroundImage:'linear-gradient(rgba(255,25
<div style={{maxWidth:1100,margin:'0 auto',padding:'0 24px',width:'100%',position:'re
{/* Eyebrow */}
<div className="scroll-reveal" style={{display:'inline-flex',alignItems:'center',ga
<span style={{width:6,height:6,borderRadius:'50%',background:'#14b8a6',boxShadow:
<span style={{fontSize:11,fontWeight:600,color:'rgba(20,184,166,0.9)',fontFamily:
</div>
{/* Massive headline — Oxaley style */}
<div className="scroll-reveal" style={{animationDelay:'.08s'}}>
<h1 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:'clamp(48px,9vw,110p
The AI that
</h1>
<h1 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:'clamp(48px,9vw,110p
analyses your
</h1>
<h1 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:'clamp(48px,9vw,110p
portfolio.
</h1>
</div>
<div className="scroll-reveal" style={{display:'flex',alignItems:'flex-start',gap:4
<div style={{maxWidth:400}}>
<p style={{fontSize:17,color:'rgba(255,255,255,0.45)',lineHeight:1.7,marginBott
Connect Binance, Coinbase, eToro & more. AI analyses live — market, economy,
</p>
<div style={{display:'flex',gap:12,flexWrap:'wrap',marginBottom:48}}>
<button className="btn btn-blue btn-lg" onClick={onEnter} style={{borderRadiu
<button onClick={()=>document.getElementById('pricing')?.scrollIntoView({beha
style={{display:'inline-flex',alignItems:'center',gap:8,padding:'14px 24px'
onMouseOver={e=>{e.currentTarget.style.color='#fff';e.currentTarget.style.b
onMouseOut={e=>{e.currentTarget.style.color='rgba(255,255,255,0.5)';e.curre
View pricing
</button>
</div>
<p style={{fontSize:11,color:'rgba(255,255,255,0.2)',letterSpacing:'.06em',font
</div>
{/* Portfolio card */}
<div className="scroll-reveal" style={{flex:'1 1 320px',minWidth:280,animationDel
<div style={{background:'rgba(255,255,255,0.04)',backdropFilter:'blur(20px)',bo
<div style={{height:1,background:'linear-gradient(90deg,transparent,rgba(20,1
<div style={{padding:'22px 22px 16px'}}>
<div style={{fontSize:10,color:'rgba(255,255,255,0.3)',fontFamily:"'JetBrai
<div style={{display:'flex',alignItems:'baseline',gap:12,marginBottom:16}}>
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:32,fontWeig
<div style={{fontSize:13,color:'#22c55e',fontWeight:700,display:'flex',al
<span style={{width:5,height:5,borderRadius:'50%',background:'#22c55e',
</div>
</div>
<MiniChart/>
<div style={{display:'flex',justifyContent:'space-between',marginTop:6,font
{['Nov','Dez','Jan','Feb','Mär','Apr'].map(m=><span key={m}>{m}</span>)}
</div>
</div>
<div style={{borderTop:'1px solid rgba(255,255,255,0.05)',padding:'12px 22px'
{[{s:'BTC',c:'+1.2%',up:true},{s:'ETH',c:'-0.8%',up:false},{s:'NVDA',c:'+2.
<div key={a.s} style={{display:'flex',alignItems:'center',gap:6,flexShrin
<AssetLogo symbol={a.s} size={18}/>
<div style={{fontSize:10,color:a.up?'#22c55e':'#ef4444',fontFamily:"'Je
</div>
))}
</div>
</div>
</div>
</div>
{/* Scroll indicator */}
<div className="scroll-reveal" style={{position:'absolute',right:24,bottom:-40,disp
<div style={{fontSize:9,color:'rgba(255,255,255,0.2)',fontFamily:"'JetBrains Mono
<div style={{width:1,height:48,background:'linear-gradient(180deg,rgba(20,184,166
</div>
</div>
</div>
{/* ── Spacer + stats strip ── */}
<div style={{padding:'100px 24px 80px',maxWidth:1100,margin:'0 auto'}}>
<div className="scroll-reveal" style={{display:'flex',gap:0,borderTop:'1px solid rgba
{[['5+','Connected Platforms'],['AES-256','Encryption'],['Real-Time','AI Analysis']
<div key={l} style={{flex:1,padding:'0 28px',borderLeft:i>0?'1px solid rgba(255,2
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:22,fontWeight:700
<div style={{fontSize:11,color:'rgba(255,255,255,0.3)',fontFamily:"'Space Grote
</div>
))}
</div>
</div>
{/* ── PLATFORMS ── */}
<div style={{borderTop:'1px solid rgba(255,255,255,0.05)',borderBottom:'1px solid rgba(
<div style={{maxWidth:1100,margin:'0 auto',display:'flex',alignItems:'center',gap:28,
<div style={{fontSize:9,color:'rgba(255,255,255,0.18)',textTransform:'uppercase',le
{PLATFORMS.map(p=><div key={p.id} style={{display:'flex',alignItems:'center',gap:7,
</div>
</div>
{/* ── FEATURES ── */}
<div style={{padding:'100px 24px',maxWidth:1100,margin:'0 auto'}}>
<div className="scroll-reveal" style={{marginBottom:56}}>
<h2 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:'clamp(30px,6vw,56px)'
<span style={{color:'#fff'}}>Everything</span><br/>
<span style={{color:'rgba(255,255,255,0.2)'}}>you need.</span>
</h2>
</div>
{[
<div style={{display:'grid',gridTemplateColumns:'repeat(auto-fit,minmax(260px,1fr))',
{n:'01',t:'AI Chart Analysis',d:'Technical indicators, Bull/Bear scenarios and ri
{n:'02',t:'Multi-Platform Sync',d:'Binance, Coinbase, eToro, Scalable, Phantom —
{n:'03',t:'Portfolio Optimisation',d:'Macro analysis with geopolitical context an
{n:'04',t:'Diversification',d:'Economic sectors, regions and benchmark comparison
{n:'05',t:'Fee Optimisation',d:'Platform comparison and switch recommendations.'}
{n:'06',t:'Live News Feed',d:'Markets and crypto — filtered for your portfolio.'}
].map((f,i)=>(
<div key={f.n} className="scroll-reveal" style={{padding:'28px 24px',background:'
onMouseOver={e=>{e.currentTarget.style.background='rgba(20,184,166,0.05)';e.cur
onMouseOut={e=>{e.currentTarget.style.background='rgba(255,255,255,0.025)';e.cu
<div style={{fontSize:9,color:'rgba(20,184,166,0.5)',fontFamily:"'JetBrains Mon
<div style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:17,fontWeight:700
<div style={{fontSize:13,color:'rgba(255,255,255,0.3)',lineHeight:1.65,fontFami
</div>
))}
</div>
</div>
{/* ── PRICING ── */}
<div style={{padding:'72px 20px',position:'relative',overflow:'hidden'}} id="pricing">
<div style={{position:'absolute',inset:0,background:'radial-gradient(ellipse at 50% 0
<div style={{maxWidth:680,margin:'0 auto',position:'relative',zIndex:1}}>
<div className="scroll-reveal" style={{marginBottom:40}}>
<h2 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:'clamp(28px,6vw,52px
<p style={{fontSize:14,color:'rgba(255,255,255,0.3)',fontFamily:"'Space Grotesk',
</div>
<div className="scroll-reveal" style={{display:'flex',flexDirection:'column',gap:2}
{/* Basic */}
<div style={{background:'rgba(255,255,255,0.03)',backdropFilter:'blur(20px)',bord
<div style={{display:'flex',justifyContent:'space-between',alignItems:'flex-sta
<div>
<div style={{fontSize:9,fontWeight:700,textTransform:'uppercase',letterSpac
<div style={{display:'flex',alignItems:'baseline',gap:2}}>
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:40,fontWeig
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:22,fontWeig
<div style={{fontSize:12,color:'rgba(255,255,255,0.22)',fontFamily:"'Spac
</div>
</div>
<button onClick={()=>setShowWithdrawal("basic")} style={{padding:'9px 18px',b
onMouseOver={e=>{e.currentTarget.style.borderColor='rgba(255,255,255,0.28)'
onMouseOut={e=>{e.currentTarget.style.borderColor='rgba(255,255,255,0.12)';
Get started
</button>
</div>
<div style={{display:'flex',flexWrap:'wrap',gap:'5px 18px'}}>
{['3 Platforms','Portfolio Analysis','Live News Feed','Basic Trends','Fee Tra
<div key={f} style={{display:'flex',alignItems:'center',gap:7}}>
<svg width="11" height="8" viewBox="0 0 11 8"><path d="M1 4l3 3 6-6" stro
<span style={{fontSize:12,color:'rgba(255,255,255,0.35)',fontFamily:"'Spa
</div>
))}
</div>
</div>
{/* Premium */}
<div style={{background:'rgba(20,184,166,0.05)',backdropFilter:'blur(20px)',borde
<div style={{position:'absolute',top:0,left:0,right:0,height:1,background:'line
<div style={{position:'absolute',top:-80,right:-80,width:200,height:200,backgro
<div style={{display:'flex',justifyContent:'space-between',alignItems:'flex-sta
<div>
<div style={{display:'flex',alignItems:'center',gap:8,marginBottom:8}}>
<div style={{fontSize:9,fontWeight:700,textTransform:'uppercase',letterSp
<span style={{fontSize:10}}> </span>
</div>
<div style={{display:'flex',alignItems:'baseline',gap:2}}>
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:40,fontWeig
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:22,fontWeig
<div style={{fontSize:12,color:'rgba(255,255,255,0.22)',fontFamily:"'Spac
</div>
</div>
<button onClick={()=>setShowWithdrawal("premium")} style={{padding:'9px 18px'
onMouseOver={e=>{e.currentTarget.style.boxShadow='0 0 35px rgba(20,184,166,
onMouseOut={e=>{e.currentTarget.style.boxShadow='0 0 20px rgba(20,184,166,0
Start →
</button>
</div>
<div style={{display:'flex',flexWrap:'wrap',gap:'5px 18px',position:'relative',
{['All 5 Platforms','AI Chart Analysis','Portfolio Optimisation','Macro & Geo
<div key={f} style={{display:'flex',alignItems:'center',gap:7}}>
<svg width="11" height="8" viewBox="0 0 11 8"><path d="M1 4l3 3 6-6" stro
<span style={{fontSize:12,color:'rgba(255,255,255,0.6)',fontFamily:"'Spac
</div>
))}
</div>
</div>
</div>
</div>
</div>
<div style={{padding:'18px 24px',display:'flex',justifyContent:'space-between',alignIte
<div style={{fontSize:12,color:'rgba(255,255,255,0.18)',fontFamily:"'Space Grotesk',s
<div style={{fontSize:11,color:'rgba(255,255,255,0.12)',fontFamily:"'JetBrains Mono',
</div>
</div>
);
}
// ─── Onboarding ────────────────────────────────────────────────────────────
// ─── Animated OTP Boxes ────────────────────────────────────────────────────
function OTPBoxes({otp,otpRefs,otpVerified,otpError,onInput,onKey}){
const [focused,setFocused]=useState(null);
const [popIdx,setPopIdx]=useState(null);
const [shaking,setShaking]=useState(false);
const [allFilled,setAllFilled]=useState(false);
const [showVerified,setShowVerified]=useState(false);
const prevOtp=useRef([...otp]);
// Pop animation on digit entry
useEffect(()=>{
otp.forEach((d,i)=>{
if(d&&!prevOtp.current[i]){
setPopIdx(i);
setTimeout(()=>setPopIdx(p=>p===i?null:p),280);
}
});
prevOtp.current=[...otp];
setAllFilled(otp.every(d=>d));
},[otp]);
// Shake on error
useEffect(()=>{
if(otpError){
setAllFilled(false);
setShaking(true);
const t=setTimeout(()=>setShaking(false),500);
return()=>clearTimeout(t);
}
},[otpError]);
// Delay verified screen slightly so all-glow phase is visible first
useEffect(()=>{
if(otpVerified){
const t=setTimeout(()=>setShowVerified(true),350);
return()=>clearTimeout(t);
} else setShowVerified(false);
},[otpVerified]);
50% 45
// ── Verified screen ────────────────────────────────────────────────────
if(showVerified) return(
<div style={{position:"relative",padding:"24px 0 12px",textAlign:"center",minHeight:188,d
{/* Soft background flash */}
<div style={{position:"absolute",inset:-40,background:"radial-gradient(circle at <div style={{position:"relative",animation:"verifiedGlow .45s cubic-bezier(.22,1,.36,1)
{/* Expanding rings */}
{[0,1,2].map(i=>(
<div key={i} style={{
position:"absolute",top:"50%",left:"50%",width:90,height:90,borderRadius:22,
border:"1px solid rgba(168,168,179,0.35)",
transform:"translate(-50%,-50%) scale(.7)",
animation:`verifiedRing 1.1s ${.15+i*.18}s cubic-bezier(.22,1,.36,1) forwards`,
pointerEvents:"none",
}}/>
))}
{/* Checkmark box */}
<div style={{
width:90,height:90,borderRadius:22,margin:"0 auto 18px",
background:"rgba(168,168,179,0.07)",
border:"2px solid rgba(168,168,179,0.5)",
display:"flex",alignItems:"center",justifyContent:"center",
boxShadow:"0 0 28px rgba(168,168,179,0.35),0 0 56px rgba(168,168,179,0.18)",
position:"relative",
}}>
<div style={{position:"absolute",inset:0,borderRadius:22,boxShadow:"0 0 28px rgba(1
<svg width="40" height="40" viewBox="0 0 40 40" fill="none" style={{position:"relat
<path d="M8 20l8 8 16-16" stroke="rgba(200,200,210,0.95)" strokeWidth="3" fill="n
strokeLinecap="round" strokeLinejoin="round"
strokeDasharray="48" strokeDashoffset="48"
style={{animation:"checkmarkDraw .5s .25s cubic-bezier(.22,1,.36,1) forwards"}}
</svg>
</div>
<div style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:18,fontWeight:700,color
Verified successfully
</div>
</div>
</div>
);
return(
<div style={{
display:"flex",gap:12,justifyContent:"center",marginBottom:20,minHeight:96,
animation:shaking?"otpShake .45s ease":"none",
}}>
{otp.map((digit,i)=>{
const isFocused=focused===i;
const hasDigit=!!digit;
const isPop=popIdx===i;
const borderCol = allFilled
?"rgba(168,168,179,0.65)"
:isFocused?"rgba(168,168,179,0.7)"
:hasDigit?"rgba(168,168,179,0.32)"
:"rgba(255,255,255,0.08)";
const baseShadow = allFilled
?"0 0 18px rgba(168,168,179,0.45),0 0 36px rgba(168,168,179,0.2)"
:isFocused?"0 0 12px rgba(168,168,179,0.35),0 0 22px rgba(168,168,179,0.15)"
:"0 2px 8px rgba(0,0,0,0.25)";
return(
<div key={i} style={{
position:"relative",
transform:isPop?"scale(1.1)":allFilled?"scale(1.02)":"scale(1)",
transition:"transform .25s cubic-bezier(.34,1.56,.64,1)",
willChange:"transform",
}}>
{/* Floating cursor dot */}
<div style={{
position:"absolute",top:-11,left:"50%",
width:isFocused?20:hasDigit?7:4,
height:4,borderRadius:99,
background:isFocused?"rgba(168,168,179,0.85)":hasDigit?"rgba(168,168,179,0.4)":
transition:"width .25s cubic-bezier(.34,1.56,.64,1), background .25s ease",
transform:"translateX(-50%)",
}}/>
{/* Glow halo layer (separate from input to avoid animation conflicts) */}
<div style={{
position:"absolute",inset:-2,borderRadius:18,
boxShadow:baseShadow,
opacity:allFilled?1:isFocused?1:0,
animation:allFilled?"otpAllGlow 1.4s ease-in-out infinite":isFocused?"otpGlowPu
transition:"opacity .25s ease, box-shadow .25s ease",
pointerEvents:"none",
}}/>
<input
ref={otpRefs[i]}
type="text" inputMode="numeric" maxLength={1}
value={digit}
onChange={e=>onInput(e.target.value,i)}
onKeyDown={e=>onKey(e,i)}
onFocus={()=>setFocused(i)}
onBlur={()=>setFocused(null)}
style={{
position:"relative",
width:64, height:76,
textAlign:"center",
fontSize:32, fontWeight:800,
fontFamily:"'JetBrains Mono',monospace",
background: allFilled||isFocused
?"rgba(168,168,179,0.07)"
:hasDigit?"rgba(168,168,179,0.05)"
:"rgba(255,255,255,0.03)",
border:`2px solid ${borderCol}`,
borderRadius:16,
color:"rgba(255,255,255,0.92)",
outline:"none",
cursor:"text",
boxShadow:"0 2px 8px rgba(0,0,0,0.25)",
transition:"background .2s ease, border-color .2s ease",
caretColor:"transparent",
WebkitTapHighlightColor:"transparent",
WebkitUserSelect:"none",
userSelect:"none",
WebkitAppearance:"none",
appearance:"none",
WebkitTextFillColor:"rgba(255,255,255,0.92)",
}}
autoComplete="one-time-code"
/>
{/* Bottom fill bar */}
<div style={{position:"absolute",bottom:0,left:6,right:6,height:2,borderRadius:"0
<div style={{
height:"100%",
width:hasDigit||isFocused?"100%":"0%",
background:allFilled?"rgba(168,168,179,0.7)":isFocused?"rgba(168,168,179,0.6)
transition:"width .25s cubic-bezier(.22,1,.36,1), background .25s ease",
}}/>
</div>
</div>
);
})}
</div>
);
}
function WithdrawalUpgradeButton({onTogglePremium,onClose,label,wide}){
const [showW,setShowW]=useState(false);
return(
<>
{showW&&<WithdrawalModal plan="premium" onAccept={()=>{setShowW(false);onTogglePremium(
<button className="btn btn-blue btn-lg" style={wide?{}:{width:"100%",justifyContent:"ce
</>
);
}
function Onboarding({onDone,engine}){
const [step,setStep]=useState(1);
const [form,setForm]=useState({name:"",email:"",pass:""});
const [agb,setAgb]=useState(false);
const [privacy,setPrivacy]=useState(false);
const [showAgb,setShowAgb]=useState(false);
const [showPrivacy,setShowPrivacy]=useState(false);
const [selPlat,setSelPlat]=useState(null);
const [creds,setCreds]=useState({});
const [connecting,setConnecting]=useState(false);
const [ok,setOk]=useState(false);
const [otp,setOtp]=useState(["","","",""]);
const [otpCode,setOtpCode]=useState("");
const [otpError,setOtpError]=useState("");
const [otpSent,setOtpSent]=useState(false);
const [otpVerified,setOtpVerified]=useState(false);
const [resendTimer,setResendTimer]=useState(0);
const otpRefs=[useRef(),useRef(),useRef(),useRef()];
const plat=PLATFORMS.find(p=>p.id===selPlat);
const doConnect=async()=>{setConnecting(true);await engine.connect(selPlat,creds);setConnec
const sendOTP=()=>{
const code=String(Math.floor(1000+Math.random()*9000));
setOtpCode(code); setOtp(["","","",""]); setOtpError(""); setOtpSent(true); setResendTime
let t=30; const iv=setInterval(()=>{t--;setResendTimer(t);if(t<=0)clearInterval(iv);},100
};
const handleOtpInput=(val,idx)=>{
if(!/^[0-9]*$/.test(val))return;
const next=[...otp]; next[idx]=val.slice(-1); setOtp(next); setOtpError("");
if(val&&idx<3) otpRefs[idx+1].current?.focus();
const full=next.join("");
if(full.length===4){
setTimeout(()=>{
if(full===otpCode){setOtpVerified(true);setTimeout(()=>setStep(2),1200);}
else{setOtpError("Falscher Code. Bitte erneut versuchen.");setOtp(["","","",""]);setT
},200);
}
};
const handleOtpKey=(e,idx)=>{
if(e.key==="Backspace"&&!otp[idx]&&idx>0) otpRefs[idx-1].current?.focus();
};
const AGB_TEXT=`Terms and Conditions – YBU Finance
1. Scope
These Terms apply to YBU Finance and all related services.
2. Services Provided
Portfolio tracking, financial data analysis, AI-powered insights. For informational purposes
3. Disclaimer of Liability
YBU Finance does not guarantee accuracy, completeness, or timeliness of data. Investing invol
4. Acceptable Use
No attacks, automated overloading, or unauthorized distribution of content.
5. AI Analysis Features
AI analyses are based on algorithms and historical data. Results may be inaccurate.
6. Privacy
Data processed per YBU Finance Privacy Policy.
7. Changes
Terms may be updated at any time.`;
const PRIVACY_TEXT=`Privacy Policy – YBU Finance
1. Introduction
We value your privacy and protect your personal data.
2. Data We Collect
Account info, portfolio data, device/browser info, analytics and cookies.
3. How We Use Data
Provide services, AI analysis, security, updates.
4. Data Sharing
We do not sell data. Shared only with required service providers or when legally required.
5. Data Security
Reasonable technical and organizational measures to protect data.
6. Cookies
Used to improve experience and analyze traffic.
7. Changes
Policy updates become effective upon publication.`;
const DocModal=({title,text,onClose})=>(
<div style={{position:"fixed",inset:0,background:"rgba(0,0,0,0.85)",backdropFilter:"blur(
<div style={{background:"var(--bg2)",border:"1px solid var(--border2)",borderRadius:"12
<div style={{padding:"14px 20px",borderBottom:"1px solid var(--border)",display:"flex
<div style={{fontWeight:700,fontSize:14}}>{title}</div>
<button onClick={onClose} style={{background:"rgba(255,255,255,0.06)",border:"1px s
</div>
<div style={{overflowY:"auto",padding:"18px 20px",flex:1}}>
<pre style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:12,color:"rgba(255,25
</div>
<div style={{padding:"14px 20px",borderTop:"1px solid var(--border)",flexShrink:0}}>
<button className="btn btn-blue" style={{width:"100%",justifyContent:"center"}} onC
</div>
</div>
</div>
);
return(
<div style={{minHeight:"100vh",background:"var(--bg)",display:"flex",flexDirection:"colum
{showAgb&&<DocModal title="Allgemeine Geschäftsbedingungen" text={AGB_TEXT} onClose={()
{showPrivacy&&<DocModal title="Datenschutzerklärung" text={PRIVACY_TEXT} onClose={()=>s
<nav className="nav"><div className="nav-inner"><div className="logo">YBU<span classNam
<div style={{width:"100%",maxWidth:520}}>
<div className="steps">
{[1,2,3].map((s,i)=>{
const numStep=step==="otp"?1.5:typeof step==="string"?2:step;
const isDone=numStep>s, isAct=Math.round(numStep)===s||(step==="otp"&&s===1);
return(
<div key={s} style={{display:"flex",alignItems:"center",flex:i<2?1:"0 0 auto"}}
<div className={`step-n ${isDone?"done":isAct?"act":"idle"}`}>{isDone?"✓":s}<
{i<2&&<div className={`step-ln ${isDone?"done":""}`}/>}
</div>
);
})}
</div>
<div className="as">
{step===1&&(
<div className="card" style={{padding:26}}>
<h2 className="pg-title" style={{marginBottom:4}}>Account erstellen</h2>
<p style={{color:"var(--text3)",fontSize:12,marginBottom:20,fontFamily:"'JetBra
<div style={{display:"flex",flexDirection:"column",gap:11}}>
<div><label className="lbl">Name</label><input className="inp" placeholder="M
<div><label className="lbl">E-Mail</label><input className="inp" type="email"
<div><label className="lbl">Passwort</label><input className="inp" type="pass
<div style={{padding:"14px",background:"var(--bg3)",borderRadius:8,border:"1p
<label style={{display:"flex",alignItems:"flex-start",gap:10,cursor:"pointe
<div onClick={()=>setAgb(v=>!v)} style={{width:18,height:18,borderRadius:
{agb&&<svg width="10" height="8" viewBox="0 0 10 8"><path d="M1 4l2.5 2
</div>
<span style={{fontSize:12,color:"rgba(255,255,255,0.55)",lineHeight:1.5,f
</label>
<label style={{display:"flex",alignItems:"flex-start",gap:10,cursor:"pointe
<div onClick={()=>setPrivacy(v=>!v)} style={{width:18,height:18,borderRad
{privacy&&<svg width="10" height="8" viewBox="0 0 10 8"><path d="M1 4l2
</div>
<span style={{fontSize:12,color:"rgba(255,255,255,0.55)",lineHeight:1.5,f
</label>
</div>
<button className="btn btn-blue" style={{marginTop:6,justifyContent:"center"}
</div>
</div>
)}
{step==="otp"&&(
<div className="card as" style={{padding:"32px 24px",textAlign:"center",overflow:
<div style={{position:"absolute",top:-60,left:"50%",transform:"translateX(-50%)
<div style={{position:"relative",zIndex:1}}>
<div style={{width:52,height:52,borderRadius:"50%",background:"rgba(168,168,1
<h2 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:20,fontWeight:70
<p style={{fontSize:13,color:"rgba(255,255,255,0.4)",lineHeight:1.6,marginBot
Code an <strong style={{color:"rgba(255,255,255,0.7)"}}>{form.email}</stron
</p>
{otpCode&&!otpVerified&&(
<div style={{display:"inline-flex",alignItems:"center",gap:8,padding:"7px 1
<span style={{color:"rgba(255,255,255,0.25)"}}>DEMO:</span>
<span style={{color:"rgba(168,168,179,0.9)",fontWeight:700,letterSpacing:
</div>
)}
<OTPBoxes otp={otp} otpRefs={otpRefs} otpVerified={otpVerified} otpError={otp
{otpError&&<div style={{padding:"8px 12px",background:"rgba(239,68,68,0.06)",
<div style={{fontSize:12,color:"rgba(255,255,255,0.25)",fontFamily:"'Space Gr
Code nicht erhalten?{" "}
{resendTimer>0
?<span style={{color:"rgba(168,168,179,0.35)"}}>Erneut in {resendTimer}s<
:<span onClick={sendOTP} style={{color:"rgba(168,168,179,0.8)",cursor:"po
</div>
<button className="btn btn-ghost" style={{fontSize:12,color:"rgba(255,255,255
</div>
</div>
)}
{step===2&&<div>
<div className="card" style={{padding:22,marginBottom:10}}>
<h2 className="pg-title" style={{marginBottom:4}}>Portfolio verbinden</h2>
<p style={{color:"var(--text3)",fontSize:12,marginBottom:14,fontFamily:"'JetBra
<div className="sec-b" style={{marginBottom:14}}><span style={{fontSize:16}}>
<div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7,marginBot
{PLATFORMS.map(p=><div key={p.id} onClick={()=>{setSelPlat(p.id);setCreds({})
</div>
{selPlat&&plat&&<div style={{padding:14,background:"var(--bg2)",borderRadius:8,
<div style={{fontSize:10,color:"rgba(255,255,255,0.45)",padding:"7px 10px",bo
{plat.fields.map(f=><div key={f} style={{marginBottom:9}}><label className="l
<div style={{padding:"7px 10px",background:"var(--goldbg)",border:"1px solid
{ok&&<div style={{padding:"8px 10px",background:"var(--gbg)",border:"1px soli
<button className="btn btn-blue" style={{width:"100%",justifyContent:"center"
</div>}
</div>
<button className="btn btn-ghost" style={{width:"100%",justifyContent:"center"}}
</div>}
{step===3&&<div className="card" style={{overflow:"hidden"}}>
<HelixLoader label="Account wird eingerichtet" sublabel="Portfolio wird synchroni
<div style={{padding:"0 24px 24px"}}>
<button className="btn btn-blue btn-lg" style={{width:"100%",justifyContent:"ce
const result=registerAccount({name:form.name||"User",email:form.email||`user$
const account=result.account||{id:Date.now().toString(),name:form.name||"User
saveSession(account);
onDone(account);
}}>Dashboard öffnen →</button>
</div>
</div>}
</div>
</div>
</div>
);
}
// ─── Withdrawal Policy Modal ───────────────────────────────────────────────
function WithdrawalModal({plan,onAccept,onClose}){
const [accepted,setAccepted]=useState(false);
const price=plan==="premium"?"9,99€":"2,99€";
const planName=plan==="premium"?"Premium":"Basic";
return(
<div className="overlay" onClick={e=>e.target===e.currentTarget&&onClose()}>
<div className="modal" style={{maxWidth:520}}>
<div className="modal-handle"/>
<div style={{padding:"0 20px 14px",borderBottom:"1px solid rgba(255,255,255,0.06)"}}>
<div style={{fontWeight:800,fontSize:16,fontFamily:"'Space Grotesk',sans-serif",mar
<div style={{fontSize:11,color:"var(--text3)",fontFamily:"'JetBrains Mono',monospac
</div>
<div style={{padding:"16px 20px",overflowY:"auto",maxHeight:"55vh"}}>
{/* Plan summary */}
<div style={{padding:"12px 14px",background:"rgba(168,168,179,0.05)",border:"1px so
<div>
<div style={{fontSize:9,color:"var(--text3)",fontFamily:"'JetBrains Mono',monos
<div style={{fontWeight:700,fontSize:14,fontFamily:"'JetBrains Mono',monospace"
</div>
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:22,fontWeight:700,c
</div>
{/* Policy text */}
<div style={{display:"flex",flexDirection:"column",gap:14}}>
{[
{n:"§1",t:"Widerrufsrecht",d:`Sie haben das Recht, diese Bestellung innerhalb v
{n:"§2",t:"Widerruf ausüben",d:`Um Ihr Widerrufsrecht auszuüben, müssen Sie uns
{n:"§3",t:"Teilweise Nutzung",d:`Haben Sie verlangt, dass die Dienstleistungen
{n:"§4",t:"Nach der Widerrufsfrist",d:`Nach Ablauf der 14-Tage-Frist kann das A
{n:"§5",t:"Sofortiger Leistungsbeginn",d:`Mit dem Kauf stimmen Sie ausdrücklich
].map(s=>(
<div key={s.n} style={{padding:"12px 14px",background:"var(--bg3)",borderRadius
<div style={{display:"flex",gap:10,alignItems:"flex-start"}}>
<span style={{fontSize:9,fontWeight:800,color:"var(--text3)",fontFamily:"'J
<div>
<div style={{fontWeight:700,fontSize:12,marginBottom:4,fontFamily:"'Space
<div style={{fontSize:12,color:"rgba(255,255,255,0.55)",lineHeight:1.65,f
</div>
</div>
</div>
))}
<div style={{padding:"10px 12px",background:"rgba(245,158,11,0.06)",border:"1px s
Keine Anlageberatung. YBU Finance bietet ausschließlich Datenanalysen und To
</div>
</div>
</div>
{/* Accept + CTA */}
<div style={{padding:"14px 20px",borderTop:"1px solid rgba(255,255,255,0.06)"}}>
<label style={{display:"flex",alignItems:"flex-start",gap:10,cursor:"pointer",margi
<div onClick={()=>setAccepted(v=>!v)} style={{width:18,height:18,borderRadius:4,b
{accepted&&<svg width="10" height="8" viewBox="0 0 10 8"><path d="M1 4l2.5 2.5
</div>
<span style={{fontSize:12,color:"rgba(255,255,255,0.55)",lineHeight:1.55,fontFami
Ich habe die Widerrufsbelehrung gelesen und akzeptiere die Bedingungen. Ich sti
</span>
</label>
<button
onClick={()=>accepted&&onAccept()}
disabled={!accepted}
style={{width:"100%",padding:"13px",background:accepted?"linear-gradient(135deg,#
{accepted?`${planName} Plan für ${price}/Monat abonnieren →`:"Bitte zuerst akzept
</button>
<button onClick={onClose} style={{width:"100%",padding:"9px",background:"transparen
</div>
</div>
</div>
);
}
// ─── Profile Modal ─────────────────────────────────────────────────────────
function ProfileModal({isPremium,onClose,onTogglePremium,account,onLogout}){
const [cancelConfirm,setCancelConfirm]=useState(false), [cancelled,setCancelled]=useState(f
const handleCancel=()=>{if(!cancelConfirm){setCancelConfirm(true);return;}setCancelled(true
return(
<div className="overlay" onClick={e=>e.target===e.currentTarget&&onClose()}>
<div className="modal" style={{maxWidth:440}}>
<div className="modal-handle"/>
<div style={{padding:"0 20px 14px",borderBottom:"1px solid rgba(255,255,255,0.06)"}}>
<div style={{display:"flex",alignItems:"center",gap:12}}>
<div style={{width:48,height:48,borderRadius:9,background:"var(--bg2)",border:"1p
<div style={{flex:1}}><div style={{fontWeight:800,fontSize:16,fontFamily:"'JetBra
<button onClick={onClose} style={{background:"var(--bg2)",border:"1px solid var(-
</div>
</div>
<div style={{padding:"16px 20px",display:"flex",flexDirection:"column",gap:10}}>
<div style={{padding:"13px 15px",background:"var(--bg2)",borderRadius:9,border:"1px
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform:"upperc
{[["Name",account?.name||"Yanic"],["E-Mail",account?.email||"yanic@example.com"],
<div key={l} style={{display:"flex",justifyContent:"space-between",alignItems:"
<span style={{fontSize:12,color:"var(--text3)",fontFamily:"'JetBrains Mono',m
<span style={{fontSize:12,fontWeight:600,fontFamily:"'JetBrains Mono',monospa
</div>
))}
</div>
<div style={{padding:"13px 15px",background:"var(--bg2)",borderRadius:9,border:`1px
<div style={{fontSize:9,fontWeight:700,color:"var(--text3)",textTransform:"upperc
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",ma
<div><div style={{fontWeight:700,fontSize:14,fontFamily:"'JetBrains Mono',monos
<span className={isPremium?"chip chip-gold":"chip chip-b"} style={{fontSize:10}
</div>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"center",pa
<span style={{fontSize:11,color:"var(--text3)",fontFamily:"'JetBrains Mono',mon
<span style={{fontSize:12,fontWeight:600,fontFamily:"'JetBrains Mono',monospace
</div>
</div>
{isPremium&&!cancelled&&(cancelConfirm?(
<div style={{padding:"12px 14px",background:"var(--rbg)",borderRadius:9,border:"1
<div style={{fontSize:12,fontWeight:600,color:"var(--red)",marginBottom:10,font
<div style={{display:"flex",gap:8}}><button className="btn btn-danger btn-sm" s
</div>
):(
<button className="btn btn-white" style={{width:"100%",justifyContent:"center",co
))}
{cancelled&&<div style={{padding:"10px 13px",background:"var(--gbg)",borderRadius:8
{!isPremium&&<WithdrawalUpgradeButton onTogglePremium={onTogglePremium} onClose={on
<button className="btn btn-ghost" style={{width:"100%",justifyContent:"center",colo
<div style={{fontSize:10,color:"var(--text3)",textAlign:"center",fontFamily:"'JetBr
</div>
</div>
</div>
);
}
// ─── Auth System ───────────────────────────────────────────────────────────
const AUTH_KEY = "ybu_accounts_v1";
const SESSION_KEY = "ybu_session_v1";
function getAccounts() {
try { return JSON.parse(localStorage.getItem(AUTH_KEY)||"[]"); } catch { return []; }
}
function saveAccounts(accounts) {
localStorage.setItem(AUTH_KEY, JSON.stringify(accounts));
}
function getSession() {
try { return JSON.parse(localStorage.getItem(SESSION_KEY)||"null"); } catch { return null;
}
function saveSession(session) {
if (session) localStorage.setItem(SESSION_KEY, JSON.stringify(session));
else localStorage.removeItem(SESSION_KEY);
}
function registerAccount({name, email, password, plan}) {
const accounts = getAccounts();
if (accounts.find(a=>a.email.toLowerCase()===email.toLowerCase()))
return {error:"E-Mail bereits registriert."};
const account = {
id: Date.now().toString(),
name, email: email.toLowerCase(), password,
plan: plan||"basic",
createdAt: new Date().toISOString(),
subscriptionActive: true,
nextBilling: new Date(Date.now()+30*86400000).toISOString(),
paymentHistory: [{date:new Date().toISOString(),amount:plan==="premium"?9.99:2.99,status:
};
accounts.push(account);
saveAccounts(accounts);
return {ok:true, account};
}
function loginAccount({email, password}) {
const accounts = getAccounts();
const account = accounts.find(a=>a.email===email.toLowerCase()&&a.password===password);
if (!account) return {error:"E-Mail oder Passwort falsch."};
if (!account.subscriptionActive) return {error:"suspended", account};
return {ok:true, account};
}
function suspendAccount(id) {
const accounts = getAccounts();
const idx = accounts.findIndex(a=>a.id===id);
if (idx!==-1) { accounts[idx].subscriptionActive=false; saveAccounts(accounts); }
}
function reactivateAccount(id, plan) {
const accounts = getAccounts();
const idx = accounts.findIndex(a=>a.id===id);
if (idx!==-1) {
accounts[idx].subscriptionActive=true;
accounts[idx].plan=plan||accounts[idx].plan;
accounts[idx].nextBilling=new Date(Date.now()+30*86400000).toISOString();
accounts[idx].paymentHistory.push({date:new Date().toISOString(),amount:plan==="premium"?
saveAccounts(accounts);
return accounts[idx];
}
}
// ─── Login Screen ───────────────────────────────────────────────────────────
function LoginScreen({onLogin, onRegister}) {
const [email,setEmail]=useState("");
const [password,setPassword]=useState("");
const [error,setError]=useState("");
const [loading,setLoading]=useState(false);
const [suspended,setSuspended]=useState(null);
const [showReactivate,setShowReactivate]=useState(false);
const handleLogin = async () => {
if(!email||!password){setError("Bitte alle Felder ausfüllen.");return;}
setLoading(true); setError("");
await new Promise(r=>setTimeout(r,600));
const result = loginAccount({email,password});
setLoading(false);
if (result.ok) { saveSession(result.account); onLogin(result.account); }
else if (result.error==="suspended") { setSuspended(result.account); }
else setError(result.error);
};
const handleReactivate = (plan) => {
if (!suspended) return;
const updated = reactivateAccount(suspended.id, plan);
if (updated) { saveSession(updated); onLogin(updated); }
};
if (suspended) return(
<div style={{minHeight:"100vh",background:"var(--bg)",display:"flex",flexDirection:"colum
<nav className="nav"><div className="nav-inner"><div className="logo" onClick={onRegist
<div style={{width:"100%",maxWidth:460}}>
<div className="card" style={{padding:28,textAlign:"center",border:"1px solid rgba(23
<div style={{width:52,height:52,borderRadius:"50%",background:"rgba(239,68,68,0.08)
<h2 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:20,fontWeight:700,marg
<p style={{fontSize:13,color:"var(--text3)",lineHeight:1.6,fontFamily:"'Space Grote
Dein Zugang für <strong style={{color:"var(--text)"}}>{suspended.email}</strong>
</p>
{!showReactivate ? (
<div style={{display:"flex",flexDirection:"column",gap:10}}>
<button className="btn btn-blue" style={{justifyContent:"center"}} onClick={()=
<button className="btn btn-ghost" style={{justifyContent:"center"}} onClick={()
</div>
) : (
<div>
<div style={{fontSize:12,color:"var(--text3)",marginBottom:14,fontFamily:"'JetB
<div style={{display:"flex",flexDirection:"column",gap:8,marginBottom:16}}>
{[{id:"basic",name:"Basic",price:"2,99€"},{id:"premium",name:"Premium",price:
<button key={p.id} onClick={()=>handleReactivate(p.id)} style={{padding:"14
onMouseOver={e=>{e.currentTarget.style.borderColor="rgba(168,168,179,0.4)
onMouseOut={e=>{e.currentTarget.style.borderColor=p.id==="premium"?"rgba(
<span>{p.name}</span>
<span style={{fontFamily:"'JetBrains Mono',monospace",fontWeight:700}}>{p
</button>
))}
</div>
</div>
<button className="btn btn-ghost" style={{justifyContent:"center",width:"100%"}
)}
</div>
</div>
</div>
);
return(
<div style={{minHeight:"100vh",background:"var(--bg)",display:"flex",flexDirection:"colum
{/* Ambient glow */}
<div style={{position:"absolute",top:"20%",left:"50%",transform:"translateX(-50%)",widt
<nav className="nav"><div className="nav-inner"><div className="logo" onClick={onRegist
<div style={{width:"100%",maxWidth:420,position:"relative",zIndex:1}}>
<div style={{textAlign:"center",marginBottom:32}}>
<div style={{fontFamily:"'JetBrains Mono',monospace",fontSize:9,fontWeight:700,colo
<h1 style={{fontFamily:"'Space Grotesk',sans-serif",fontSize:"clamp(28px,6vw,38px)"
</div>
<div className="card" style={{padding:28}}>
<div style={{display:"flex",flexDirection:"column",gap:14}}>
<div>
<label className="lbl">E-Mail</label>
<input className="inp" type="email" placeholder="deine@email.de" value={email}
onChange={e=>{setEmail(e.target.value);setError("");}}
onKeyDown={e=>e.key==="Enter"&&handleLogin()}
style={{background:"rgba(255,255,255,0.04)"}}/>
</div>
<div>
<label className="lbl">Passwort</label>
<input className="inp" type="password" placeholder="Dein Passwort" value={passw
onChange={e=>{setPassword(e.target.value);setError("");}}
onKeyDown={e=>e.key==="Enter"&&handleLogin()}
style={{background:"rgba(255,255,255,0.04)"}}/>
</div>
{error&&(
<div style={{padding:"10px 12px",background:"rgba(239,68,68,0.06)",border:"1px
{error}
</div>
)}
<button className="btn btn-blue" style={{justifyContent:"center",marginTop:4}} on
{loading?<><div className="spn"/>Anmelden…</>:"Anmelden →"}
</button>
</div>
<div style={{marginTop:20,paddingTop:18,borderTop:"1px solid rgba(255,255,255,0.06)
<span style={{fontSize:13,color:"var(--text3)",fontFamily:"'Space Grotesk',sans-s
<span onClick={onRegister} style={{fontSize:13,color:"rgba(168,168,179,0.9)",font
</div>
</div>
<div style={{marginTop:14,textAlign:"center",fontSize:11,color:"rgba(255,255,255,0.18
KEINE ANLAGEBERATUNG · YBU FINANCE
</div>
</div>
</div>
);
}
// ─── Root App ──────────────────────────────────────────────────────────────
export default function App(){
const [page,setPage]=useState(()=>{
// Check for existing session on load
const session=getSession();
if(session&&session.subscriptionActive) return "dashboard";
return "landing";
});
const [tab,setTab]=useState("overview");
const [currentAccount,setCurrentAccount]=useState(()=>getSession());
const [isPremium,setIsPremium]=useState(()=>{
const s=getSession();
return s?s.plan==="premium":true;
});
const [showProfile,setShowProfile]=useState(false);
const engine=useEngine();
const handleLogin=(account)=>{
setCurrentAccount(account);
setIsPremium(account.plan==="premium");
setPage("dashboard");
};
const handleLogout=()=>{
saveSession(null);
setCurrentAccount(null);
setPage("landing");
};
const handleRegisterSuccess=(account)=>{
saveSession(account);
setCurrentAccount(account);
setIsPremium(account.plan==="premium");
setPage("dashboard");
};
const navItems=[
{id:"overview",label:"Übersicht",icon:<svg width="15" height="15" viewBox="0 0 15 15" fil
{id:"live", label:"News", icon:<svg width="15" height="15" viewBox="0 0 15 15" fil
{id:"chart", label:"Charts", icon:<svg width="15" height="15" viewBox="0 0 15 15" fil
{id:"optimize",label:"Optimize", icon:<svg width="15" height="15" viewBox="0 0 15 15" fil
{id:"connect", label:"Verbinden",icon:<svg width="15" height="15" viewBox="0 0 15 15" fil
];
if(page==="landing") return <><style>{CSS}</style><Landing onEnter={()=>setPage("onboard
if(page==="login") return <><style>{CSS}</style><LoginScreen onLogin={handleLogin} onR
if(page==="onboarding") return <><style>{CSS}</style><Onboarding onDone={handleRegisterSucc
return(
<>
<style>{CSS}</style>
{showProfile&&<ProfileModal isPremium={isPremium} onClose={()=>setShowProfile(false)} o
<nav className="nav">
<div className="nav-inner">
<div className="logo" onClick={()=>setPage("landing")}>YBU <span className="logo-ac
<div style={{display:"flex",gap:8,alignItems:"center"}}>
<div style={{display:"flex",alignItems:"center",gap:5,padding:"4px 9px",borderRad
<button className="nav-btn" style={{color:isPremium?"var(--gold)":"rgba(255,255,2
<div className="av" onClick={()=>setShowProfile(true)}>{currentAccount?.name?curr
</div>
</div>
</nav>
<aside className="sidebar">
<div className="sb-sec">Portfolio</div>
{navItems.map(item=><div key={item.id} className={`sb-item ${tab===item.id?"active":"
<div className="sb-sec">Konto</div>
<div className="sb-item" onClick={()=>setPage("landing")}><svg width="13" height="13"
</aside>
<nav className="mob-nav">
<div className="mob-nav-inner">
{navItems.map(item=><div key={item.id} className={`mob-item ${tab===item.id?"active
</div>
</nav>
<main className="main" style={{position:"relative"}}>
<div style={{position:"fixed",top:"30%",right:"10%",width:400,height:400,background:"ra
<div style={{position:"fixed",bottom:"20%",left:"15%",width:300,height:300,background:"
{tab==="overview" &&<Overview engine={engine}/>}
{tab==="live" &&<LiveFeed isPremium={isPremium}/>}
{tab==="chart" &&<ChartAnalysis isPremium={isPremium} onTogglePremium={()=>setIsP
{tab==="optimize" &&<Optimization isPremium={isPremium} portfolio={engine.portfolio}
{tab==="connect" &&<ConnectView engine={engine} isPremium={isPremium}/>}
<div style={{marginTop:36,paddingTop:12,borderTop:"1px solid var(--border)",fontSize:
KEINE ANLAGEBERATUNG. YBU FINANCE BIETET AUSSCHLIESSLICH DATENANALYSEN. KURSDATE
</div>
</main>
</>
}
);
