// ═══════════════════════════════════════════════════════════════════════
//  BTCUSDT INSTITUTIONAL AI TRADING PLATFORM  v2  — PATCH APPLIED
//  Fixes: no repaint · stable AI brain · persistent trades · ATR trailing
//         stop (Chandelier Exit) · split price/analysis streams · no lag
//         · fixed-width numbers · scroll-safe CSS · locked signal cards
// ═══════════════════════════════════════════════════════════════════════

import {
  useState, useEffect, useRef, useMemo, useCallback, memo, useReducer
} from "react";
import {
  AreaChart, Area, BarChart, Bar,
  XAxis, YAxis, ResponsiveContainer, Tooltip, Cell, ReferenceLine
} from "recharts";

/* ─── CONFIG ──────────────────────────────────────────────────────────── */
const SYM         = "BTCUSDT";
const FAPI        = "https://fapi.binance.com/fapi/v1";
const WS_BASE     = "wss://fstream.binance.com/stream";
const SHOW_THRESH = 0.60;
const ALERT_THRESH= 0.75;
const KELLY_K     = 0.25;
const MAX_POS     = 0.02;
const MC_PATHS    = 300;
const CW          = { m:0.25, s:0.20, o:0.20, h:0.15, q:0.20 };
// Brain / regime refresh cadence (ms) — prevents flicker
const BRAIN_TTL   = 30_000;   // 30 s
const SIGNAL_TTL  = 60_000;   // 60 s — recompute only on new closed candle
const REGIME_TTL  = 45_000;   // 45 s
// ATR Chandelier multiplier
const CHANDELIER_M = 3.0;
// Trade Management Agent hard rules
const TMA_CONF_MIN   = 0.75;   // hard deny below this — no exceptions
const TMA_MIN_RR     = 1.5;    // minimum R:R to accept any trade
const TMA_MAX_OPEN   = 2;      // max concurrent open trades
const TMA_LOSS_PAUSE = 3;      // consecutive losses before pause (trades)
const TMA_WR_FLOOR   = 0.40;   // win rate floor → switch to PROTECTIVE mode
const TMA_WR_STRICT  = 0.30;   // below this → PAUSED mode
const TMA_HOUR_LIMIT = 5;      // max trades per hour (overtrading guard)
const TMA_BE_TRIGGER = 0.50;   // move SL to breakeven after this fraction of TP1
const TMA_PARTIAL_AT = 0.80;   // recommend partial close at this fraction of TP1
const TMA_FUNDING_L  = 0.03;   // funding% above this → warn longs
const TMA_FUNDING_S  =-0.01;   // funding% below this → warn shorts

/* ─── TELEGRAM CONFIG ────────────────────────────────────────────────── */
const TG_TOKEN  = "8737180442:AAGE8lXKX7gFbdxRiMhIdQy7UmbAbRIxuU4";
const TG_CHAT   = "8737180442";
const TG_API    = `https://api.telegram.org/bot${TG_TOKEN}/sendMessage`;
async function tgSend(text){
  try{
    await fetch(TG_API,{method:"POST",headers:{"Content-Type":"application/json"},
      body:JSON.stringify({chat_id:TG_CHAT,text,parse_mode:"Markdown"})});
  }catch{}
}

/* ─── DEFAULT SETTINGS ───────────────────────────────────────────────── */
const SETTINGS_KEY = "btc_platform_settings_v1";
const DEFAULT_SETTINGS = {
  equity        : 10000,    // virtual + paper equity ($)
  riskPct       : 1.0,      // max risk per trade (%)
  confThresh    : 0.75,     // signal show threshold (matches ALERT_THRESH)
  rrMin         : 1.5,      // min R:R
  chandelierM   : 3.0,      // Chandelier ATR multiplier
  tgEnabled     : true,     // Telegram notifications on/off
  tgSignals     : true,     // notify on new signals
  tgTrades      : true,     // notify on trade open/close
  tgAlerts      : true,     // notify on TMA alerts
  paperMode     : false,    // paper trading mode
  mcPaths       : 300,      // Monte Carlo paths
};
function loadSettings(){
  try{const s=localStorage.getItem(SETTINGS_KEY);return s?{...DEFAULT_SETTINGS,...JSON.parse(s)}:DEFAULT_SETTINGS;}
  catch{return DEFAULT_SETTINGS;}
}
function saveSettings(s){try{localStorage.setItem(SETTINGS_KEY,JSON.stringify(s));}catch{}}

/* ─── PAPER TRADING ENGINE ───────────────────────────────────────────── */
function paperReducer(state,action){
  const now=Date.now();
  switch(action.type){
    case "RESET":
      return{equity:action.equity||10000,startEquity:action.equity||10000,
        trades:[],log:[],peak:action.equity||10000,maxDD:0};
    case "OPEN":{
      const{sig,riskPct}=action;
      const risk=Math.abs(sig.entry-sig.sl);
      const posSize=state.equity*(riskPct/100)/risk;
      const trade={id:now,card:sig.id,dir:sig.dir,entry:sig.entry,sl:sig.sl,
        tp:sig.tp,atr:sig.atr,col:sig.col,openedAt:now,status:"OPEN",
        posSize,riskAmt:state.equity*(riskPct/100),currentPrice:sig.entry,
        pnl:0,pnlPct:0,tpsHit:0,trailStop:sig.sl,conf:sig.finalConf,regime:sig.regime};
      return{...state,trades:[...state.trades,trade],
        log:[...state.log,{ts:now,ev:`OPEN ${sig.id} ${sig.dir} @ $${sig.entry?.toLocaleString()}`}]};
    }
    case "TICK":{
      const p=action.price;
      let eq=state.startEquity;
      const updated=state.trades.map(t=>{
        if(t.status!=="OPEN") return t;
        const pnl=(p-t.entry)*(t.dir==="LONG"?1:-1)*t.posSize;
        const pnlPct=pnl/state.startEquity*100;
        let tpsHit=t.tpsHit;
        t.tp.forEach((tp,i)=>{if(tpsHit<=i){if(t.dir==="LONG"&&p>=tp)tpsHit=i+1;if(t.dir==="SHORT"&&p<=tp)tpsHit=i+1;}});
        const stopHit=t.dir==="LONG"?p<=t.trailStop:p>=t.trailStop;
        const tpHit=t.dir==="LONG"?p>=t.tp[t.tp.length-1]:p<=t.tp[t.tp.length-1];
        if(stopHit||tpHit){
          const finalPnl=(p-t.entry)*(t.dir==="LONG"?1:-1)*t.posSize;
          return{...t,status:"CLOSED",outcome:stopHit?(t.tpsHit>0?"TRAIL_TP":"SL"):"TP",
            closePrice:p,closedAt:now,pnl:finalPnl,pnlPct:finalPnl/state.startEquity*100};
        }
        return{...t,currentPrice:p,pnl,pnlPct,tpsHit};
      });
      // Running equity = start + sum of closed PnL
      const closedPnl=updated.filter(t=>t.status==="CLOSED").reduce((s,t)=>s+t.pnl,0);
      eq=state.startEquity+closedPnl;
      const peak=Math.max(state.peak,eq);
      const maxDD=Math.max(state.maxDD,peak-eq);
      return{...state,trades:updated,equity:parseFloat(eq.toFixed(2)),peak,maxDD};
    }
    case "CLOSE_ALL":{
      const p=action.price;
      const updated=state.trades.map(t=>{
        if(t.status!=="OPEN") return t;
        const finalPnl=(p-t.entry)*(t.dir==="LONG"?1:-1)*t.posSize;
        return{...t,status:"CLOSED",outcome:"MANUAL",closePrice:p,closedAt:now,
          pnl:finalPnl,pnlPct:finalPnl/state.startEquity*100};
      });
      const closedPnl=updated.reduce((s,t)=>s+t.pnl,0);
      return{...state,trades:updated,equity:parseFloat((state.startEquity+closedPnl).toFixed(2))};
    }
    default: return state;
  }
}

/* ─── BACKTESTER ─────────────────────────────────────────────────────── */
// Runs signal logic over historical closed klines — walk-forward, no lookahead
function runBacktest(klines, timeframe, equity=10000, riskPct=1.0, ofStub={score:0.5,pressure:"BALANCED"}){
  if(!klines||klines.length<80) return null;
  const results=[];
  let eq=equity;
  // Walk forward: use [0..i-1] to generate signal for bar i
  const windowSize=Math.min(200,klines.length-2);
  for(let i=windowSize;i<klines.length-1;i++){
    const window=klines.slice(0,i); // closed candles up to but not including i
    const cl=window.map(r=>parseFloat(r[4]));
    const hi=window.map(r=>parseFloat(r[2]));
    const lo=window.map(r=>parseFloat(r[3]));
    const n=cl.length-1,p=cl[n];
    // Quick signal check (EMA + RSI only for backtest speed)
    const e21=TA.ema(cl,21),e50=TA.ema(cl,50);
    const rsiArr=TA.rsi(cl,14);
    const atrArr=TA.atr(hi,lo,cl,14);
    const rN=rsiArr[n],e21N=e21[n],e50N=e50[n],aN=atrArr[n];
    const bullStack=p>e21N&&e21N>e50N;
    const bearStack=p<e21N&&e21N<e50N;
    const macdD=TA.macd(cl);
    const macdBull=macdD.hist[n]>0&&macdD.hist[n-1]<=0;
    const macdBear=macdD.hist[n]<0&&macdD.hist[n-1]>=0;
    let dir=null,score=0;
    if(bullStack&&macdBull&&rN>45&&rN<70){dir="LONG";score=0.68+Math.random()*0.15;}
    else if(bearStack&&macdBear&&rN<55&&rN>30){dir="SHORT";score=0.68+Math.random()*0.15;}
    if(!dir||score<0.65) continue;
    const sl=dir==="LONG"?p-aN*1.8:p+aN*1.8;
    const tp=dir==="LONG"?p+aN*2.5:p-aN*2.5;
    const rr=Math.abs(tp-p)/Math.abs(p-sl);
    if(rr<1.5) continue;
    // Simulate forward: check next candle for outcome
    const nextBar=klines[i];
    const nextHi=parseFloat(nextBar[2]),nextLo=parseFloat(nextBar[3]),nextCl=parseFloat(nextBar[4]);
    let outcome="TIMEOUT",exitPrice=nextCl,pnlPct=0;
    if(dir==="LONG"){
      if(nextLo<=sl){outcome="SL";exitPrice=sl;}
      else if(nextHi>=tp){outcome="TP";exitPrice=tp;}
    } else {
      if(nextHi>=sl){outcome="SL";exitPrice=sl;}
      else if(nextLo<=tp){outcome="TP";exitPrice=tp;}
    }
    const risk=eq*(riskPct/100);
    const riskPts=Math.abs(p-sl);
    const posSize=riskPts>0?risk/riskPts:0;
    const tradePnl=(exitPrice-p)*(dir==="LONG"?1:-1)*posSize;
    eq=Math.max(0,eq+tradePnl);
    const pnl=tradePnl/equity*100;
    results.push({i,ts:parseInt(klines[i][0]),entry:p,sl,tp,dir,outcome,exitPrice,
      pnl:parseFloat(pnl.toFixed(3)),equity:parseFloat(eq.toFixed(2)),score});
  }
  if(!results.length) return null;
  // Stats
  const wins=results.filter(r=>r.outcome==="TP");
  const losses=results.filter(r=>r.outcome==="SL");
  const wr=wins.length/results.length;
  const avgWin=wins.length?wins.reduce((s,r)=>s+r.pnl,0)/wins.length:0;
  const avgLoss=losses.length?Math.abs(losses.reduce((s,r)=>s+r.pnl,0)/losses.length):1;
  const pf=avgLoss>0?(wr*avgWin)/((1-wr)*avgLoss):0;
  let peak=equity,maxDD=0;
  results.forEach(r=>{peak=Math.max(peak,r.equity);maxDD=Math.max(maxDD,peak-r.equity);});
  const finalEq=results[results.length-1]?.equity||equity;
  return{results,wr,pf:pf.toFixed(2),maxDD:maxDD.toFixed(0),total:results.length,
    wins:wins.length,losses:losses.length,finalEq:finalEq.toFixed(0),
    returnPct:((finalEq-equity)/equity*100).toFixed(2),avgWin:avgWin.toFixed(3),avgLoss:avgLoss.toFixed(3)};
}

/* ─── THEME ──────────────────────────────────────────────────────────── */
const C = {
  bg:"#020812", card:"rgba(5,11,25,0.97)",
  border:"rgba(0,185,255,0.09)", borderHi:"rgba(0,185,255,0.26)",
  cyan:"#00BDFF", gold:"#F5A623", green:"#00E676",
  red:"#FF1744",  orange:"#FF8F00", purple:"#8B5CF6",
  text:"#B4C8E4", bright:"#DBF0FF", muted:"#162035", dim:"#3D5270",
};

/* ─── REST HELPERS ───────────────────────────────────────────────────── */
const apiFetch = (path, q="") =>
  fetch(`${FAPI}/${path}?symbol=${SYM}${q?"&"+q:""}`).then(r=>r.json());
const REST = {
  klines : (iv,n=200) => apiFetch("klines",      `interval=${iv}&limit=${n}`),
  ticker : ()         => apiFetch("ticker/24hr"),
  premium: ()         => apiFetch("premiumIndex"),
  oi     : ()         => apiFetch("openInterest"),
  depth  : (n=20)     => apiFetch("depth",        `limit=${n}`),
};

/* ─── TECHNICAL ANALYSIS ─────────────────────────────────────────────── */
const TA = {
  ema(px,p){
    const k=2/(p+1); let e=px[0];
    return px.map((v,i)=>(e=i?v*k+e*(1-k):v));
  },
  rsi(px,p=14){
    if(px.length<p+2) return px.map(()=>50);
    let g=0,l=0;
    for(let i=1;i<=p;i++){const d=px[i]-px[i-1];d>0?g+=d:l-=d;} g/=p;l/=p;
    const r=Array(p).fill(50);
    r.push(l?100-100/(1+g/l):100);
    for(let i=p+1;i<px.length;i++){
      const d=px[i]-px[i-1],gn=d>0?d:0,ln=d<0?-d:0;
      g=(g*(p-1)+gn)/p; l=(l*(p-1)+ln)/p;
      r.push(l?100-100/(1+g/l):100);
    }
    return r;
  },
  macd(px,f=12,s=26,sg=9){
    const ef=this.ema(px,f),es=this.ema(px,s);
    const ml=px.map((_,i)=>ef[i]-es[i]);
    const sl=this.ema(ml,sg);
    return{ml,sl,hist:ml.map((m,i)=>m-sl[i])};
  },
  atr(hi,lo,cl,p=14){
    const tr=cl.map((c,i)=>i?Math.max(hi[i]-lo[i],Math.abs(hi[i]-cl[i-1]),Math.abs(lo[i]-cl[i-1])):hi[0]-lo[0]);
    return this.ema(tr,p);
  },
  bb(px,p=20,m=2){
    return px.map((_,i)=>{
      if(i<p-1) return{u:null,mid:null,l:null,bw:null};
      const sl=px.slice(i-p+1,i+1),mean=sl.reduce((s,v)=>s+v)/p;
      const std=Math.sqrt(sl.reduce((s,v)=>s+(v-mean)**2)/p);
      return{u:mean+m*std,mid:mean,l:mean-m*std,bw:std?m*2*std/mean:0};
    });
  },
  vwap(hi,lo,cl,vol){
    let tv=0,v=0;
    return cl.map((c,i)=>{tv+=(hi[i]+lo[i]+cl[i])/3*vol[i];v+=vol[i];return v?tv/v:c;});
  },
  swings(hi,lo,lb=5){
    const sh=[],sl=[];
    for(let i=lb;i<hi.length-lb;i++){
      if(hi.slice(i-lb,i).every(h=>h<hi[i])&&hi.slice(i+1,i+lb+1).every(h=>h<hi[i])) sh.push({i,p:hi[i]});
      if(lo.slice(i-lb,i).every(l=>l>lo[i])&&lo.slice(i+1,i+lb+1).every(l=>l>lo[i])) sl.push({i,p:lo[i]});
    }
    return{sh,sl};
  },
  hv(cl,p=20){
    if(cl.length<p+1) return 0;
    const rets=cl.slice(1).map((c,i)=>Math.log(c/cl[i]));
    const rec=rets.slice(-p),mu=rec.reduce((s,v)=>s+v)/p;
    return Math.sqrt(rec.reduce((s,v)=>s+(v-mu)**2)/(p-1)*365*24*60);
  },
  // Chandelier Exit — best trailing stop for trend trades
  // Long: max(high,n) - atr*m   Short: min(low,n) + atr*m
  chandelier(hi,lo,cl,p=22,m=CHANDELIER_M){
    const atrArr=this.atr(hi,lo,cl,p);
    return cl.map((_,i)=>{
      if(i<p) return{long:null,short:null};
      const maxH=Math.max(...hi.slice(i-p+1,i+1));
      const minL=Math.min(...lo.slice(i-p+1,i+1));
      return{long:maxH-atrArr[i]*m, short:minL+atrArr[i]*m};
    });
  },
  fibonacci(hi,lo){
    const d=hi-lo;
    return{r236:lo+d*0.236,r382:lo+d*0.382,r500:lo+d*0.500,r618:lo+d*0.618,r786:lo+d*0.786};
  },
};

/* ─── CLOSED-CANDLE GUARD ────────────────────────────────────────────── */
// Strips the last (forming) candle. All analysis runs on closed candles only.
function closedOnly(klines){
  if(!klines||klines.length<3) return klines;
  return klines.slice(0,-1); // last entry is the live forming candle — drop it
}

/* ─── REGIME DETECTOR ────────────────────────────────────────────────── */
function detectRegime(klines){
  const k=closedOnly(klines);
  if(!k||k.length<55) return{regime:"LOADING",dir:"NEUTRAL",conf:0};
  const cl=k.map(r=>parseFloat(r[4]));
  const hi=k.map(r=>parseFloat(r[2]));
  const lo=k.map(r=>parseFloat(r[3]));
  const vol=k.map(r=>parseFloat(r[5]));
  const n=cl.length-1, p=cl[n];
  const e9=TA.ema(cl,9),e21=TA.ema(cl,21),e50=TA.ema(cl,50);
  const e200=TA.ema(cl,Math.min(200,cl.length-1));
  const rsiArr=TA.rsi(cl,14); const rsiN=rsiArr[n];
  const atrArr=TA.atr(hi,lo,cl,14); const atrN=atrArr[n];
  const bbArr=TA.bb(cl,20); const bbN=bbArr[n];
  const e9n=e9[n],e21n=e21[n],e50n=e50[n],e200n=e200[n];
  const bw=bbN?.bw||0;
  const avgBW=bbArr.slice(-20).filter(b=>b.bw!=null).reduce((s,b)=>s+b.bw,0)/20||0.01;
  const slope=(e21n-e21[Math.max(0,n-5)])/e21[Math.max(0,n-5)]*100;
  const avgVol5=vol.slice(-5).reduce((s,v)=>s+v)/5;
  const avgVol20=vol.slice(-20).reduce((s,v)=>s+v)/20;
  const highVol=avgVol5>avgVol20*1.35;
  const atrPct=atrN/p;
  let regime="RANGE_BOUND",dir="NEUTRAL",conf=0.52;
  if(p>e9n&&e9n>e21n&&e21n>e50n&&p>e200n&&slope>0.08&&highVol){regime="STRONG_BULL";dir="BULLISH";conf=0.87;}
  else if(p>e9n&&e9n>e21n&&e21n>e50n&&p>e200n&&slope>0.02)     {regime="MODERATE_BULL";dir="BULLISH";conf=0.73;}
  else if(p<e9n&&e9n<e21n&&e21n<e50n&&p<e200n&&slope<-0.08&&highVol){regime="STRONG_BEAR";dir="BEARISH";conf=0.87;}
  else if(p<e9n&&e9n<e21n&&e21n<e50n&&p<e200n&&slope<-0.02)    {regime="MODERATE_BEAR";dir="BEARISH";conf=0.73;}
  else if((atrPct>0.012||bw>avgBW*1.6)&&!(p>e9n&&e9n>e21n))   {regime="HIGH_VOLATILITY";dir="NEUTRAL";conf=0.65;}
  else if(atrPct<0.004||bw<avgBW*0.65)                          {regime="COMPRESSION";dir="NEUTRAL";conf=0.77;}
  return{regime,dir,conf,e9:e9n,e21:e21n,e50:e50n,e200:e200n,
         rsi:rsiN,atr:atrN,atrPct,bbWidth:bw,slope,above200:p>e200n};
}

/* ─── ORDER FLOW ─────────────────────────────────────────────────────── */
function calcOrderFlow(depth){
  if(!depth?.bids?.length) return{score:0.5,pressure:"NEUTRAL",bidDepth:0,askDepth:0,bidRatio:0.5,askRatio:0.5};
  const bd=depth.bids.slice(0,15).reduce((s,[p,q])=>s+parseFloat(p)*parseFloat(q),0);
  const ad=depth.asks.slice(0,15).reduce((s,[p,q])=>s+parseFloat(p)*parseFloat(q),0);
  const tot=bd+ad; const bidR=tot?bd/tot:0.5;
  const bigBid=depth.bids.slice(0,5).find(([,q])=>parseFloat(q)>8);
  const bigAsk=depth.asks.slice(0,5).find(([,q])=>parseFloat(q)>8);
  return{score:bidR,bidRatio:bidR,askRatio:1-bidR,bidDepth:bd,askDepth:ad,
    pressure:bidR>0.62?"BUY_DOMINANT":bidR>0.56?"BUY_LEANING":bidR<0.38?"SELL_DOMINANT":bidR<0.44?"SELL_LEANING":"BALANCED",
    bigBidWall:bigBid?parseFloat(bigBid[1]):0,bigAskWall:bigAsk?parseFloat(bigAsk[1]):0};
}

/* ─── MONTE CARLO (real returns) ─────────────────────────────────────── */
function runMC(cl,entry,sl,tp1){
  if(cl.length<30) return{probTP:0.5,probSL:0.5,ev:0,p10:sl,p50:entry,p90:tp1,mcData:[]};
  const rets=cl.slice(-80).slice(1).map((c,i)=>Math.log(c/cl.slice(-80)[i]));
  const mu=rets.reduce((s,v)=>s+v)/rets.length;
  const sd=Math.sqrt(rets.reduce((s,v)=>s+(v-mu)**2)/(rets.length-1));
  let tpH=0,slH=0; const finals=[];
  for(let p=0;p<MC_PATHS;p++){
    let pr=entry,hit=null;
    for(let s=0;s<80&&!hit;s++){
      const u1=Math.random()||1e-10,u2=Math.random();
      pr*=Math.exp(mu+sd*Math.sqrt(-2*Math.log(u1))*Math.cos(2*Math.PI*u2));
      if(pr>=tp1){hit="TP";tpH++;}
      else if(pr<=sl){hit="SL";slH++;}
    }
    if(!hit){pr>entry?tpH+=0.5:slH+=0.5;}
    finals.push(Math.round(pr));
  }
  finals.sort((a,b)=>a-b);
  const probTP=tpH/MC_PATHS, probSL=slH/MC_PATHS;
  const ev=probTP*(tp1-entry)-probSL*(entry-sl);
  const mcData=Array.from({length:20},(_,i)=>{
    const t=i/19;
    return{i,
      p10:Math.round(entry+(finals[Math.floor(MC_PATHS*0.10)]-entry)*t),
      p50:Math.round(entry+(finals[Math.floor(MC_PATHS*0.50)]-entry)*t),
      p90:Math.round(entry+(finals[Math.floor(MC_PATHS*0.90)]-entry)*t)};
  });
  return{probTP,probSL,ev,p10:finals[Math.floor(MC_PATHS*0.10)],
    p50:finals[Math.floor(MC_PATHS*0.50)],p90:finals[Math.floor(MC_PATHS*0.90)],mcData};
}

/* ─── SIGNAL BUILDERS (closed candles only) ──────────────────────────── */
function buildSignals(kmap, of, regime){
  const results={snp:null,int:null,trd:null};

  // ── SNIPER (1m closed) ──
  const k1=closedOnly(kmap.k1m);
  if(k1?.length>=55){
    const cl=k1.map(r=>parseFloat(r[4])),hi=k1.map(r=>parseFloat(r[2]));
    const lo=k1.map(r=>parseFloat(r[3])),vol=k1.map(r=>parseFloat(r[5]));
    const n=cl.length-1,p=cl[n];
    const e9=TA.ema(cl,9),e21=TA.ema(cl,21);
    const rsi=TA.rsi(cl,14),macdD=TA.macd(cl,5,13,5);
    const atrArr=TA.atr(hi,lo,cl,14),bbArr=TA.bb(cl,20);
    const rN=rsi[n],e9N=e9[n],e21N=e21[n],aN=atrArr[n],bbN=bbArr[n];
    const hN=macdD.hist[n],hP=macdD.hist[n-1];
    const avgVol=vol.slice(-20).reduce((s,v)=>s+v)/20;
    const bullEMA=p>e9N&&e9N>e21N,bearEMA=p<e9N&&e9N<e21N;
    const macdBull=hN>0&&hP<=0,macdBear=hN<0&&hP>=0;
    const bbBL=bbN&&p<=bbN.l*1.001,bbBH=bbN&&p>=bbN.u*0.999;
    const rsiOS=rN<36,rsiOB=rN>64,rsiBnc=rN<45&&rN>rsi[n-1]&&rN>rsi[n-2];
    const rsiFd=rN>55&&rN<rsi[n-1]&&rN<rsi[n-2];
    const vSpike=vol[n]>avgVol*1.6;
    let lS=0,sS=0;
    if(rsiOS||rsiBnc)lS+=0.28; if(rsiOB||rsiFd)sS+=0.28;
    if(bullEMA)lS+=0.20; if(bearEMA)sS+=0.20;
    if(macdBull)lS+=0.18; if(macdBear)sS+=0.18;
    if(of.score>0.56)lS+=0.18; if(of.score<0.44)sS+=0.18;
    if(vSpike){lS+=0.08;sS+=0.08;}
    if(bbBL)lS+=0.12; if(bbBH)sS+=0.12;
    if(regime.dir==="BULLISH")lS+=0.08; if(regime.dir==="BEARISH")sS+=0.08;
    const dir=lS>sS&&lS>=0.55?"LONG":sS>lS&&sS>=0.55?"SHORT":null;
    if(dir){
      const conf=Math.min(0.95,dir==="LONG"?lS:sS);
      const sl=dir==="LONG"?Math.round(p-aN*1.3):Math.round(p+aN*1.3);
      results.snp={id:"SNP",card:"SNIPER SCALPER",col:C.cyan,dir,entry:p,sl,
        tp:[dir==="LONG"?Math.round(p+aN*1.8):Math.round(p-aN*1.8),
            dir==="LONG"?Math.round(p+aN*2.8):Math.round(p-aN*2.8)],
        conf,rr:1.8/1.3,pw:Math.min(0.82,0.50+conf*0.20),atr:aN,
        regime:regime.regime,expire:480,
        strat:bbBL?"BB Lower Bounce":rsiOS?"RSI Oversold":macdBull?"MACD Cross":"Momentum",
        shap:[{f:"RSI/Momentum",v:rsiOS?0.28:rsiBnc?0.18:0.08},{f:"EMA Structure",v:bullEMA?0.20:0.05},{f:"Order Flow",v:Math.abs(of.score-0.5)*0.36}]};
    }
  }

  // ── INTRADAY (15m closed) ──
  const k15=closedOnly(kmap.k15m);
  if(k15?.length>=100){
    const cl=k15.map(r=>parseFloat(r[4])),hi=k15.map(r=>parseFloat(r[2]));
    const lo=k15.map(r=>parseFloat(r[3])),vol=k15.map(r=>parseFloat(r[5]));
    const n=cl.length-1,p=cl[n];
    const e21=TA.ema(cl,21),e50=TA.ema(cl,50),e200=TA.ema(cl,Math.min(200,cl.length-1));
    const rsi=TA.rsi(cl,14),macdD=TA.macd(cl),atrArr=TA.atr(hi,lo,cl,14);
    const vwap=TA.vwap(hi,lo,cl,vol);
    const rN=rsi[n],e21N=e21[n],e50N=e50[n],aN=atrArr[n];
    const hN=macdD.hist[n],hP=macdD.hist[n-1],mlN=macdD.ml[n],mlP=macdD.ml[n-1];
    const vN=vwap[n];
    const k1h=closedOnly(kmap.k1h);
    let htfBull=false,htfBear=false;
    if(k1h?.length>50){
      const c1h=k1h.map(r=>parseFloat(r[4])),nh=c1h.length-1;
      const e21h=TA.ema(c1h,21),e50h=TA.ema(c1h,50);
      htfBull=c1h[nh]>e21h[nh]&&e21h[nh]>e50h[nh];
      htfBear=c1h[nh]<e21h[nh]&&e21h[nh]<e50h[nh];
    }
    const bullStack=p>e21N&&e21N>e50N,bearStack=p<e21N&&e21N<e50N;
    const pbEMAL=Math.abs(p-e21N)/e21N<0.004&&p>e50N;
    const pbEMAS=Math.abs(p-e21N)/e21N<0.004&&p<e50N;
    const macdBull=hN>0&&(hP<=0||mlN>mlP),macdBear=hN<0&&(hP>=0||mlN<mlP);
    let lS=0,sS=0;
    if(bullStack)lS+=0.25; if(bearStack)sS+=0.25;
    if(htfBull)lS+=0.20; if(htfBear)sS+=0.20;
    if(macdBull)lS+=0.18; if(macdBear)sS+=0.18;
    if(rN>42&&rN<68)lS+=0.12; if(rN>32&&rN<58)sS+=0.12;
    if(pbEMAL)lS+=0.12; if(pbEMAS)sS+=0.12;
    if(of.score>0.55)lS+=0.15; if(of.score<0.45)sS+=0.15;
    if(p>e200[n])lS+=0.08; else sS+=0.08;
    const dir=lS>sS&&lS>=0.55?"LONG":sS>lS&&sS>=0.55?"SHORT":null;
    if(dir){
      const conf=Math.min(0.93,dir==="LONG"?lS:sS);
      const risk=aN*1.8;
      const sl=dir==="LONG"?Math.round(p-risk):Math.round(p+risk);
      results.int={id:"INT",card:"INTRADAY HUNTER",col:C.gold,dir,entry:p,sl,
        tp:[dir==="LONG"?Math.round(p+risk*1.5):Math.round(p-risk*1.5),
            dir==="LONG"?Math.round(p+risk*2.5):Math.round(p-risk*2.5),
            dir==="LONG"?Math.round(p+risk*3.8):Math.round(p-risk*3.8)],
        conf,rr:1.5,pw:Math.min(0.84,0.52+conf*0.18),atr:aN,
        regime:regime.regime,expire:2700,
        strat:pbEMAL||pbEMAS?"EMA21 Pullback":htfBull||htfBear?"HTF Trend":"Structure Play",
        shap:[{f:"EMA Alignment",v:bullStack||bearStack?0.25:0.05},{f:"HTF Trend",v:htfBull||htfBear?0.20:0.03},{f:"MACD",v:macdBull||macdBear?0.18:0.04}]};
    }
  }

  // ── TREND (4h closed) ──
  const k4=closedOnly(kmap.k4h);
  if(k4?.length>=80){
    const cl=k4.map(r=>parseFloat(r[4])),hi=k4.map(r=>parseFloat(r[2]));
    const lo=k4.map(r=>parseFloat(r[3]));
    const n=cl.length-1,p=cl[n];
    const e21=TA.ema(cl,21),e50=TA.ema(cl,50),e200=TA.ema(cl,Math.min(200,cl.length-1));
    const rsi=TA.rsi(cl,14),macdD=TA.macd(cl),atrArr=TA.atr(hi,lo,cl,14);
    const {sh,sl:swSL}=TA.swings(hi,lo,4);
    const rN=rsi[n],e21N=e21[n],e50N=e50[n],e200N=e200[n],aN=atrArr[n];
    const hN=macdD.hist[n],mlN=macdD.ml[n],slN=macdD.sl[n];
    const bosL=sh.length>=2&&p>sh[sh.length-2].p;
    const bosS=swSL.length>=2&&p<swSL[swSL.length-2].p;
    const k1d=closedOnly(kmap.k1d);
    let dailyBull=false,dailyBear=false;
    if(k1d?.length>55){
      const c1d=k1d.map(r=>parseFloat(r[4])),nd=c1d.length-1;
      const e50d=TA.ema(c1d,50),e200d=TA.ema(c1d,Math.min(200,c1d.length-1));
      dailyBull=c1d[nd]>e50d[nd]&&e50d[nd]>e200d[nd];
      dailyBear=c1d[nd]<e50d[nd]&&e50d[nd]<e200d[nd];
    }
    const fullBull=p>e21N&&e21N>e50N&&e50N>e200N;
    const fullBear=p<e21N&&e21N<e50N&&e50N<e200N;
    const macdBull=mlN>slN&&hN>0,macdBear=mlN<slN&&hN<0;
    let lS=0,sS=0;
    if(fullBull)lS+=0.28; if(fullBear)sS+=0.28;
    if(dailyBull)lS+=0.22; if(dailyBear)sS+=0.22;
    if(macdBull)lS+=0.18; if(macdBear)sS+=0.18;
    if(bosL)lS+=0.16; if(bosS)sS+=0.16;
    if(rN>50&&rN<72)lS+=0.10; if(rN<50&&rN>28)sS+=0.10;
    if(of.score>0.55)lS+=0.08; if(of.score<0.45)sS+=0.08;
    const dir=lS>sS&&lS>=0.55?"LONG":sS>lS&&sS>=0.55?"SHORT":null;
    if(dir){
      const conf=Math.min(0.94,dir==="LONG"?lS:sS);
      const risk=aN*2.2;
      const sl=dir==="LONG"?Math.round(p-risk):Math.round(p+risk);
      results.trd={id:"TRD",card:"TREND RIDER",col:C.purple,dir,entry:p,sl,
        tp:[dir==="LONG"?Math.round(p+risk*1.5):Math.round(p-risk*1.5),
            dir==="LONG"?Math.round(p+risk*2.8):Math.round(p-risk*2.8),
            dir==="LONG"?Math.round(p+risk*4.5):Math.round(p-risk*4.5)],
        conf,rr:1.5,pw:Math.min(0.82,0.48+conf*0.22),atr:aN,
        regime:regime.regime,expire:14400,
        strat:bosL?"Break of Structure":bosS?"BOS Short":fullBull?"EMA Stack Long":"EMA Stack Short",
        shap:[{f:"EMA Full Stack",v:fullBull||fullBear?0.28:0.05},{f:"Daily Trend",v:dailyBull||dailyBear?0.22:0.04},{f:"MACD+BOS",v:macdBull||macdBear?0.18:0.06}]};
    }
  }
  return results;
}

/* ─── RISK CALC ─────────────────────────────────────────────────────── */
function calcRisk(sig,eq=10000){
  if(!sig) return null;
  const{entry,sl,tp,pw,conf,atr}=sig;
  const risk=Math.abs(entry-sl),rew=Math.abs(tp[0]-entry);
  const rr=rew/risk, fees=entry*0.0004;
  const ev=pw*rew-(1-pw)*risk-fees;
  const kelly=Math.max(0,pw-(1-pw)/rr);
  const posPct=Math.min(kelly*KELLY_K,MAX_POS);
  return{ev,rr,kelly:kelly.toFixed(4),posPct,positionSize:eq*posPct,
    slippage:Math.round(entry*0.0002),fillProb:(0.82+conf*0.12).toFixed(2),fees:fees.toFixed(2)};
}

/* ─── CONFIDENCE AGGREGATOR ─────────────────────────────────────────── */
function aggregateConf(sig,mc,of,sw){
  if(!sig) return 0;
  const sk=sig.id==="SNP"?"snp":sig.id==="INT"?"int":"trd";
  const m=sig.conf,s=mc?mc.probTP:0.5,
        o=sig.dir==="LONG"?of.score:1-of.score,
        h=0.62,q=sw[sk]||0.33;
  return Math.min(0.99,CW.m*m+CW.s*s+CW.o*o+CW.h*h+CW.q*q);
}

/* ─── STRATEGY WEIGHTS ───────────────────────────────────────────────── */
function stratWeights(r){
  const k=r?.regime||"";
  if(k==="STRONG_BULL"||k==="STRONG_BEAR")          return{snp:0.18,int:0.35,trd:0.47};
  if(k==="MODERATE_BULL"||k==="MODERATE_BEAR")       return{snp:0.22,int:0.52,trd:0.26};
  if(k==="RANGE_BOUND")                              return{snp:0.52,int:0.38,trd:0.10};
  if(k==="HIGH_VOLATILITY")                          return{snp:0.46,int:0.44,trd:0.10};
  if(k==="COMPRESSION")                             return{snp:0.28,int:0.38,trd:0.34};
  return{snp:0.33,int:0.34,trd:0.33};
}


/* ═══════════════════════════════════════════════════════════════════════
   TRADE MANAGEMENT AGENT (TMA)
   Hard gates: conf<75%, RR<1.5, overtrading, scalp-lock, direction clash
   Directives: move-to-BE, partial close, early exit, funding gate
   Audit: agent logic conflict detection
   Modes: NORMAL → PROTECTIVE → RESTRICTED → PAUSED
═══════════════════════════════════════════════════════════════════════ */
function runTMA(signals,trades,regime,of,fundRate){
  const now=Date.now();
  const openT=trades.filter(t=>t.status==="OPEN");
  const closedT=trades.filter(t=>t.status==="CLOSED");
  const wins=closedT.filter(t=>t.outcome==="TP"||t.outcome==="TRAIL_TP");
  const losses=closedT.filter(t=>t.outcome==="SL");
  const sessionWR=closedT.length?wins.length/closedT.length:null;
  const recentT=closedT.filter(t=>now-t.closedAt<3600000);
  const tradesLastHour=recentT.length+openT.length;
  let streak=0;
  for(let i=closedT.length-1;i>=0;i--){if(closedT[i].outcome==="SL")streak++;else break;}

  // Global mode
  let globalMode="NORMAL",pauseUntil=0;
  const globalReasons=[];
  if(streak>=TMA_LOSS_PAUSE){
    globalMode="PAUSED";
    pauseUntil=(closedT[closedT.length-1]?.closedAt||now)+(30*60000);
    globalReasons.push(`Circuit breaker: ${streak} consecutive losses → 30 min cooldown`);
  } else if(sessionWR!==null&&sessionWR<TMA_WR_STRICT){
    globalMode="PAUSED";globalReasons.push(`Win rate critical (${(sessionWR*100).toFixed(0)}%) — suspended`);
  } else if(sessionWR!==null&&sessionWR<TMA_WR_FLOOR){
    globalMode="PROTECTIVE";globalReasons.push(`Win rate low (${(sessionWR*100).toFixed(0)}%) — 80%+ conf only`);
  } else if(tradesLastHour>=TMA_HOUR_LIMIT){
    globalMode="RESTRICTED";globalReasons.push(`Overtrading: ${tradesLastHour} trades/hour — reducing activity`);
  }
  const effMin=globalMode==="PROTECTIVE"?0.80:TMA_CONF_MIN;
  const isPaused=globalMode==="PAUSED"||(pauseUntil>0&&now<pauseUntil);

  function evalSig(sig,id){
    const v={id,verdict:"APPROVED",reasons:[],adjustments:[]};
    if(!sig) return{...v,verdict:"NO_SIGNAL",reasons:["No qualified signal on closed candles"]};
    if(isPaused){return{...v,verdict:"BLOCKED",reasons:["TMA paused — "+globalMode+(pauseUntil>now?" · "+Math.round((pauseUntil-now)/60000)+"m left":"")]};}
    if(sig.finalConf<effMin) v.reasons.push(`Conf ${(sig.finalConf*100).toFixed(1)}% < ${(effMin*100).toFixed(0)}% min`);
    if(sig.rr<TMA_MIN_RR)    v.reasons.push(`R:R ${sig.rr.toFixed(2)}x < ${TMA_MIN_RR}x min`);
    if(sig.mc&&sig.mc.ev<=0) v.reasons.push(`Negative EV ($${Math.round(sig.mc.ev)})`);
    if(sig.mc&&sig.mc.probTP<0.50) v.reasons.push(`MC win prob ${(sig.mc.probTP*100).toFixed(0)}% < 50%`);
    if(id==="SNP"&&openT.find(t=>t.id==="SNP")) v.reasons.push("Scalp lock: SNP already open");
    if(openT.length>=TMA_MAX_OPEN) v.reasons.push(`Max open trades (${TMA_MAX_OPEN}) reached`);
    const trdT=openT.find(t=>t.id==="TRD");
    if(id==="SNP"&&trdT&&trdT.dir!==sig.dir) v.reasons.push(`Direction clash: scalp ${sig.dir} vs TRD ${trdT.dir}`);
    if(v.reasons.length>0){v.verdict="DENIED";return v;}
    // Approved — add advisory adjustments
    if(sig.dir==="LONG"&&fundRate>TMA_FUNDING_L) v.adjustments.push(`Funding ${fundRate.toFixed(3)}% — longs carry cost, reduce size 25%`);
    if(sig.dir==="SHORT"&&fundRate<TMA_FUNDING_S) v.adjustments.push(`Funding ${fundRate.toFixed(3)}% — shorts carry cost, reduce size 25%`);
    if(id==="SNP"&&(regime.regime==="STRONG_BULL"||regime.regime==="STRONG_BEAR")) v.adjustments.push("Strong trend active — scalp tight, max 1× ATR stop");
    if(id==="TRD"&&regime.regime==="RANGE_BOUND") v.adjustments.push("Range regime — TRD lower reliability, halve size");
    if(id==="TRD"&&regime.regime==="HIGH_VOLATILITY") v.adjustments.push("High vol — widen TRD stop by 1.5× ATR");
    const flowOK=(sig.dir==="LONG"&&of.score>0.50)||(sig.dir==="SHORT"&&of.score<0.50);
    if(!flowOK) v.adjustments.push(`OB ${of.pressure} opposes direction — consider waiting for flow alignment`);
    if(sig.mc?.p50) v.adjustments.push(`MC median target $${sig.mc.p50?.toLocaleString()} · ${Math.abs((sig.mc.p50-sig.entry)/sig.entry*100).toFixed(2)}% move`);
    return v;
  }

  // Active trade directives
  const directives=[];
  openT.forEach(t=>{
    const p=t.currentPrice||t.entry;
    const tp1=t.tp[0],tp2=t.tp[1];
    const toTP1=Math.abs(tp1-t.entry);
    const prog=toTP1>0?Math.abs(p-t.entry)/toTP1:0;
    const pnlPct=t.dir==="LONG"?(p-t.entry)/t.entry*100:(t.entry-p)/t.entry*100;
    if(prog>=TMA_BE_TRIGGER&&!t.beApplied&&pnlPct>0)
      directives.push({id:t.id,action:"MOVE_BE",label:"Move SL → Breakeven",
        desc:`${(prog*100).toFixed(0)}% to TP1 · lock entry $${t.entry.toLocaleString()}`,priority:"HIGH",col:C.gold});
    if(prog>=TMA_PARTIAL_AT&&!t.partialClosed&&pnlPct>0)
      directives.push({id:t.id,action:"PARTIAL_CLOSE",label:"Take 50% Partial",
        desc:`Bank half at ${(pnlPct).toFixed(2)}% profit, run remainder`,priority:"HIGH",col:C.green});
    const regimeMismatch=(t.dir==="LONG"&&regime.dir==="BEARISH")||(t.dir==="SHORT"&&regime.dir==="BULLISH");
    if(regimeMismatch&&pnlPct>0.2)
      directives.push({id:t.id,action:"CONSIDER_EXIT",label:"Regime Flip — Consider Exit",
        desc:`Regime now ${regime.dir} · PnL +${pnlPct.toFixed(2)}% · consider locking in`,priority:"MEDIUM",col:C.orange});
    if(t.tpsHit>=1&&tp2&&!t.partialClosed)
      directives.push({id:t.id,action:"INFO",label:"TP1 Hit — Manage Runner",
        desc:`Trail stop active · targeting TP2 $${tp2?.toLocaleString()}`,priority:"INFO",col:C.cyan});
    if(t.dir==="LONG"&&fundRate>TMA_FUNDING_L&&(now-t.openedAt)>3600000)
      directives.push({id:t.id,action:"INFO",label:"Funding Cost",
        desc:`Long open ${Math.round((now-t.openedAt)/3600000)}h · ${fundRate.toFixed(3)}% funding eroding PnL`,priority:"LOW",col:C.dim});
  });

  // Agent audit
  const audit=[];
  if(signals.snp&&regime.dir!=="NEUTRAL"&&signals.snp.dir==="LONG"&&regime.dir==="BEARISH")
    audit.push({agent:"SNP vs Regime",sev:"WARN",msg:"LONG scalp against BEARISH regime — false positive risk"});
  if(signals.snp&&regime.dir!=="NEUTRAL"&&signals.snp.dir==="SHORT"&&regime.dir==="BULLISH")
    audit.push({agent:"SNP vs Regime",sev:"WARN",msg:"SHORT scalp against BULLISH regime — false positive risk"});
  if(signals.int&&signals.trd&&signals.int.dir!==signals.trd.dir)
    audit.push({agent:"INT vs TRD",sev:"WARN",msg:`INT ${signals.int?.dir} conflicts with TRD ${signals.trd?.dir} — HTF takes priority`});
  if(of.score>0.62&&regime.dir==="BEARISH")
    audit.push({agent:"OB vs Regime",sev:"INFO",msg:"Buy-dominant OB in BEARISH regime — short-covering, not genuine demand"});
  if(of.score<0.38&&regime.dir==="BULLISH")
    audit.push({agent:"OB vs Regime",sev:"INFO",msg:"Sell-dominant OB in BULLISH regime — distribution zone possible"});
  if(signals.snp&&signals.int&&signals.trd&&signals.snp.dir===signals.int.dir&&signals.int.dir===signals.trd.dir)
    audit.push({agent:"Full Alignment",sev:"OK",msg:`All cards agree: ${signals.snp.dir} — highest-conviction setup`});
  if(regime.conf<0.55)
    audit.push({agent:"Regime Engine",sev:"WARN",msg:`Regime confidence ${(regime.conf*100).toFixed(0)}% — structure unclear, prefer no-trade`});
  if(audit.length===0)
    audit.push({agent:"All Agents",sev:"OK",msg:"No logic conflicts detected — agents operating normally"});

  // Recommendations
  const recs=[];
  if(globalMode!=="NORMAL") recs.push({icon:"⚠",col:C.orange,text:globalReasons[0]});
  if(isPaused&&pauseUntil>now) recs.push({icon:"⛔",col:C.red,text:`Paused · resumes in ${Math.round((pauseUntil-now)/60000)} min`});
  if(sessionWR!==null&&sessionWR>0.65&&globalMode==="NORMAL")
    recs.push({icon:"↑",col:C.green,text:`Win rate ${(sessionWR*100).toFixed(0)}% — system performing well, standard sizing`});
  if(streak===2) recs.push({icon:"⚠",col:C.orange,text:"Two consecutive losses — review last trades before next entry"});
  const bestApproved=[{v:evalSig(signals.snp,"SNP"),s:signals.snp},{v:evalSig(signals.int,"INT"),s:signals.int},{v:evalSig(signals.trd,"TRD"),s:signals.trd}]
    .filter(x=>x.v.verdict==="APPROVED"&&x.s);
  if(bestApproved.length>0){
    const best=bestApproved.sort((a,b)=>(b.s?.finalConf||0)-(a.s?.finalConf||0))[0];
    recs.push({icon:"✓",col:C.green,text:`Best setup: ${best.v.id} ${best.s?.dir} @ $${best.s?.entry?.toLocaleString()} — ${(best.s?.finalConf*100).toFixed(0)}% conf`});
  } else if(openT.length===0){
    recs.push({icon:"◎",col:C.dim,text:"No valid setups — await closed-candle alignment"});
  }
  if(regime.regime==="COMPRESSION") recs.push({icon:"◈",col:C.cyan,text:"Compression: prepare for breakout, don't trade inside range"});
  if(Math.abs(fundRate)>0.05) recs.push({icon:"⬡",col:C.gold,text:`Extreme funding ${fundRate.toFixed(3)}% — ${fundRate>0?"long squeeze risk":"short squeeze risk"}`});

  return{
    snp:evalSig(signals.snp,"SNP"), int:evalSig(signals.int,"INT"), trd:evalSig(signals.trd,"TRD"),
    directives,audit,recs,globalMode,globalReasons,isPaused,pauseUntil,
    sessionStats:{wr:sessionWR,wins:wins.length,losses:losses.length,
      total:closedT.length,streak,tradesLastHour,openCount:openT.length}
  };
}


/* ─── ATR CHANDELIER TRAILING STOP ENGINE ────────────────────────────── */
// Chandelier Exit: best trailing stop for trend following
// Long trail = highest(close, 22) - ATR(22)*3  (only moves UP, never DOWN)
// Short trail = lowest(close, 22) + ATR(22)*3  (only moves DOWN, never UP)
function updateTrailingStop(trade, currentPrice, klines){
  const k=closedOnly(klines);
  if(!k?.length||!trade) return trade;
  const cl=k.map(r=>parseFloat(r[4]));
  const hi=k.map(r=>parseFloat(r[2]));
  const lo=k.map(r=>parseFloat(r[3]));
  const chandArr=TA.chandelier(hi,lo,cl,22,CHANDELIER_M);
  const latest=chandArr[chandArr.length-1];
  if(!latest) return trade;

  if(trade.dir==="LONG"){
    // Chandelier long: only ratchet UP
    const newTS=Math.round(latest.long);
    const trailStop=Math.max(trade.trailStop||trade.sl, newTS);
    return{...trade,trailStop};
  } else {
    // Chandelier short: only ratchet DOWN
    const newTS=Math.round(latest.short);
    const trailStop=Math.min(trade.trailStop||trade.sl, newTS);
    return{...trade,trailStop};
  }
}

/* ─── TRADE REDUCER ──────────────────────────────────────────────────── */
function tradesReducer(state, action){
  switch(action.type){
    case "OPEN": {
      const sig=action.sig;
      if(state.find(t=>t.id===sig.id&&t.status==="OPEN")) return state; // only 1 per card
      const trade={
        id:sig.id, card:sig.card, dir:sig.dir, entry:sig.entry,
        sl:sig.sl, trailStop:sig.sl, tp:sig.tp, atr:sig.atr,
        col:sig.col, conf:sig.finalConf||sig.conf,
        openedAt:Date.now(), status:"OPEN",
        maxFav:0, maxAdv:0, tpsHit:0,
      };
      return[...state.filter(t=>t.id!==sig.id),trade].slice(-10);
    }
    case "UPDATE_TRAIL": {
      return state.map(t=>{
        if(t.status!=="OPEN"||t.id!==action.id) return t;
        const p=action.price;
        const pnlPct=t.dir==="LONG"?(p-t.entry)/t.entry*100:(t.entry-p)/t.entry*100;
        const maxFav=Math.max(t.maxFav,pnlPct);
        const maxAdv=Math.max(t.maxAdv,-pnlPct);
        // Chandelier trail stop
        const ts=action.newTrailStop||t.trailStop;
        const trailStop=t.dir==="LONG"?Math.max(t.trailStop,ts):Math.min(t.trailStop,ts);
        // Check TP levels hit
        let tpsHit=t.tpsHit;
        t.tp.forEach((tp,i)=>{
          if(tpsHit<=i){
            if(t.dir==="LONG"&&p>=tp) tpsHit=i+1;
            if(t.dir==="SHORT"&&p<=tp) tpsHit=i+1;
          }
        });
        return{...t,currentPrice:p,trailStop,maxFav,maxAdv,tpsHit,
          pnlPct:parseFloat(pnlPct.toFixed(3)),
          pnlUsd:parseFloat(((p-t.entry)*(t.dir==="LONG"?1:-1)).toFixed(0))};
      });
    }
    case "CLOSE": {
      return state.map(t=>t.id===action.id&&t.status==="OPEN"
        ?{...t,status:"CLOSED",outcome:action.outcome,closedAt:Date.now(),closePrice:action.price}
        :t);
    }
    case "CHECK_STOPS": {
      const p=action.price;
      return state.map(t=>{
        if(t.status!=="OPEN") return t;
        // Trail stop hit
        const stopHit=t.dir==="LONG"?p<=t.trailStop:p>=t.trailStop;
        if(stopHit) return{...t,status:"CLOSED",outcome:t.tpsHit>0?"TRAIL_TP":"SL",closedAt:Date.now(),closePrice:p};
        // Final TP hit
        const allTPHit=t.dir==="LONG"?p>=t.tp[t.tp.length-1]:p<=t.tp[t.tp.length-1];
        if(allTPHit) return{...t,status:"CLOSED",outcome:"TP",closedAt:Date.now(),closePrice:p};
        return t;
      });
    }
    // TMA directive: move stop loss to breakeven
    case "MOVE_BE": {
      return state.map(t=>{
        if(t.status!=="OPEN"||t.id!==action.id) return t;
        // Ratchet trail stop to entry — only if profitable
        const newTS=t.entry;
        const trailStop=t.dir==="LONG"?Math.max(t.trailStop,newTS):Math.min(t.trailStop,newTS);
        return{...t,trailStop,beApplied:true};
      });
    }
    // TMA directive: record partial close taken
    case "PARTIAL_CLOSE": {
      return state.map(t=>t.id===action.id&&t.status==="OPEN"?{...t,partialClosed:true}:t);
    }
    default: return state;
  }
}

/* ─── DATA HOOK: SPLIT STREAMS ───────────────────────────────────────── */
// price stream = fast (every tick, display only)
// candle store = slow (update only on CLOSED candle → k.x===true)
// depth = 500ms throttled
function useMarketData(){
  // ── fast display state (price, depth display, funding)
  const [display,setDisplay]=useState({
    price:0,change24h:0,high24h:0,low24h:0,vol24h:0,
    fundRate:0,oi:0,markPrice:0,connected:false,loading:true,error:null
  });
  // ── closed candle store (ref → no re-render on tick, only on closed candle)
  const kRef=useRef({k1m:null,k5m:null,k15m:null,k1h:null,k4h:null,k1d:null});
  // ── candle version counter — increments only on closed candle
  const [candleVer,setCandleVer]=useState(0);
  // ── depth snapshot (throttled display)
  const [depth,setDepth]=useState({bids:[],asks:[]});
  const depthBuf=useRef({bids:[],asks:[]});
  const depthTimer=useRef(null);
  const wsRef=useRef(null);
  const reconnTimer=useRef(null);

  const loadRest=useCallback(async()=>{
    try{
      const[ticker,prem,oi,dp]=await Promise.all([REST.ticker(),REST.premium(),REST.oi(),REST.depth(20)]);
      const[k1m,k5m,k15m,k1h,k4h,k1d]=await Promise.all([
        REST.klines("1m",300),REST.klines("5m",200),REST.klines("15m",200),
        REST.klines("1h",200),REST.klines("4h",200),REST.klines("1d",100)
      ]);
      kRef.current={k1m,k5m,k15m,k1h,k4h,k1d};
      setDisplay(d=>({...d,
        price:parseFloat(ticker.lastPrice),change24h:parseFloat(ticker.priceChangePercent),
        high24h:parseFloat(ticker.highPrice),low24h:parseFloat(ticker.lowPrice),
        vol24h:parseFloat(ticker.quoteVolume)/1e9,
        fundRate:parseFloat(prem.lastFundingRate)*100,
        oi:parseFloat(oi.openInterest)*parseFloat(ticker.lastPrice)/1e9,
        markPrice:parseFloat(prem.markPrice),loading:false
      }));
      setDepth(dp);
      setCandleVer(v=>v+1);
    }catch(e){setDisplay(d=>({...d,error:e.message,loading:false}));}
  },[]);

  const connectWS=useCallback(()=>{
    if(wsRef.current) wsRef.current.close();
    const ws=new WebSocket(`${WS_BASE}?streams=btcusdt@kline_1m/btcusdt@kline_5m/btcusdt@depth20@500ms/btcusdt@markPrice@1s`);
    wsRef.current=ws;
    ws.onopen=()=>setDisplay(d=>({...d,connected:true}));
    ws.onclose=()=>{setDisplay(d=>({...d,connected:false}));reconnTimer.current=setTimeout(connectWS,3000);};
    ws.onerror=()=>setDisplay(d=>({...d,connected:false}));
    ws.onmessage=(evt)=>{
      try{
        const{stream,data}=JSON.parse(evt.data);
        if(!stream) return;

        // Price tick — only update price display, never trigger analysis
        if(stream.includes("kline_1m")){
          const k=data.k;
          setDisplay(d=>({...d,price:parseFloat(k.c),markPrice:parseFloat(k.c)}));
          // Closed candle → update store → bump version (triggers analysis)
          if(k.x){
            const nc=[k.t,k.o,k.h,k.l,k.c,k.v,k.T,"","",k.V,"",""];
            kRef.current.k1m=[...(kRef.current.k1m||[]).slice(-299),nc];
            setCandleVer(v=>v+1);  // analysis re-runs
          }
        }
        if(stream.includes("kline_5m")&&data.k?.x){
          const k=data.k;
          const nc=[k.t,k.o,k.h,k.l,k.c,k.v,k.T,"","",k.V,"",""];
          kRef.current.k5m=[...(kRef.current.k5m||[]).slice(-199),nc];
        }
        // Depth — buffer and flush at most every 600ms to avoid layout thrash
        if(stream.includes("depth20")){
          depthBuf.current={bids:data.b||[],asks:data.a||[]};
          if(!depthTimer.current){
            depthTimer.current=setTimeout(()=>{
              setDepth({...depthBuf.current});
              depthTimer.current=null;
            },600);
          }
        }
        // Funding (1s) — only update funding, not price
        if(stream.includes("markPrice")){
          setDisplay(d=>({...d,fundRate:parseFloat(data.r||0)*100}));
        }
      }catch{}
    };
  },[]);

  // Heavy refresh — 15m, 1h, 4h, 1d — every 60s
  useEffect(()=>{
    const heavy=async()=>{
      try{
        const[k15m,k1h,k4h,k1d,oi,dp]=await Promise.all([
          REST.klines("15m",200),REST.klines("1h",200),
          REST.klines("4h",200),REST.klines("1d",100),
          REST.oi(),REST.depth(20)
        ]);
        kRef.current={...kRef.current,k15m,k1h,k4h,k1d};
        setDisplay(d=>({...d,oi:parseFloat(oi.openInterest)*d.price/1e9}));
        setCandleVer(v=>v+1);
      }catch{}
    };
    const iv=setInterval(heavy,60000);
    return()=>clearInterval(iv);
  },[]);

  useEffect(()=>{
    loadRest(); connectWS();
    return()=>{
      if(wsRef.current) wsRef.current.close();
      if(reconnTimer.current) clearTimeout(reconnTimer.current);
      if(depthTimer.current) clearTimeout(depthTimer.current);
    };
  },[]);

  return{display,kRef,candleVer,depth};
}

/* ─── STABLE ANALYSIS HOOK ───────────────────────────────────────────── */
// Re-runs only when candleVer changes (closed candle) or regime TTL expires
function useAnalysis(kRef, candleVer, of){
  const [regime,setRegime]=useState({regime:"LOADING",dir:"NEUTRAL",conf:0});
  const [rawSignals,setRawSignals]=useState({snp:null,int:null,trd:null});
  const regimeTs=useRef(0);
  const signalTs=useRef(0);
  const sw=useMemo(()=>stratWeights(regime),[regime.regime]);

  useEffect(()=>{
    const now=Date.now();
    // Regime — update if TTL expired
    if(now-regimeTs.current>REGIME_TTL||regime.regime==="LOADING"){
      const r=detectRegime(kRef.current.k1m);
      setRegime(r);
      regimeTs.current=now;
    }
    // Signals — update only on closed candle (candleVer change) and if TTL expired
    if(now-signalTs.current>SIGNAL_TTL||signalTs.current===0){
      const sigs=buildSignals(kRef.current,of,regime);
      setRawSignals(sigs);
      signalTs.current=now;
    }
  },[candleVer]); // only closed candles trigger this

  // Enrich with MC + risk — stable useMemo
  const signals=useMemo(()=>{
    const enrich=(sig,klines)=>{
      if(!sig) return null;
      const cl=closedOnly(klines)?.map(r=>parseFloat(r[4]));
      const mc=cl?.length>30?runMC(cl,sig.entry,sig.sl,sig.tp[0]):null;
      const risk=calcRisk(sig);
      const finalConf=aggregateConf(sig,mc,of,sw);
      if(finalConf<SHOW_THRESH) return null;
      return{...sig,mc,risk,finalConf};
    };
    return{
      snp:enrich(rawSignals.snp,kRef.current.k1m),
      int:enrich(rawSignals.int,kRef.current.k15m),
      trd:enrich(rawSignals.trd,kRef.current.k4h),
    };
  },[rawSignals,sw]); // does NOT re-run on price tick

  return{regime,signals,sw};
}

/* ─── UI PRIMITIVES ──────────────────────────────────────────────────── */
// Fixed widths on all number displays prevent layout shift
const Box=memo(({style,...p})=><div style={{background:C.card,border:`1px solid ${C.border}`,
  borderRadius:10,padding:"13px 15px",contain:"layout style",...style}}{...p}/>);
const Num=({v,col,sz=18,w})=><div style={{fontFamily:"'Space Mono',monospace",fontWeight:700,
  fontSize:sz,color:col||C.bright,lineHeight:1.1,
  minWidth:w||"auto",fontVariantNumeric:"tabular-nums",letterSpacing:"-0.01em"}}>{v}</div>;
const Label=({ch,col,style})=><div style={{fontSize:9,letterSpacing:"0.12em",color:col||C.dim,
  textTransform:"uppercase",marginBottom:3,fontFamily:"'Space Mono',monospace",...style}}>{ch}</div>;
const Badge=({ch,col,sm})=><span style={{fontSize:sm?8:10,fontWeight:700,
  fontFamily:"'Space Mono',monospace",letterSpacing:"0.08em",color:col,
  background:`${col}18`,border:`1px solid ${col}30`,borderRadius:3,
  padding:sm?"1px 5px":"2px 7px",whiteSpace:"nowrap"}}>{ch}</span>;
const Dot=({col,pulse})=><span style={{width:7,height:7,borderRadius:"50%",
  background:col,display:"inline-block",flexShrink:0,
  boxShadow:pulse?`0 0 7px ${col}`:"none",
  animation:pulse?"blink 2s ease-in-out infinite":"none"}}/>;
const HR=()=><div style={{height:1,background:C.border,margin:"8px 0"}}/>;
const SecTitle=({ch,sub})=><div style={{padding:"18px 0 9px",contain:"layout"}}>
  <div style={{display:"flex",alignItems:"center",gap:7}}>
    <div style={{width:3,height:13,background:C.cyan,borderRadius:2,flexShrink:0}}/>
    <span style={{fontFamily:"'Rajdhani',sans-serif",fontWeight:700,fontSize:13,
      letterSpacing:"0.18em",color:C.cyan,textTransform:"uppercase"}}>{ch}</span>
  </div>
  {sub&&<div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace",
    paddingLeft:10,marginTop:2}}>{sub}</div>}
  <div style={{height:1,background:C.border,marginTop:7}}/>
</div>;
const ChartTip=({active,payload})=>{
  if(!active||!payload?.length) return null;
  return <div style={{background:"rgba(3,7,20,0.98)",border:`1px solid ${C.border}`,
    borderRadius:5,padding:"5px 10px",fontSize:9,fontFamily:"'Space Mono',monospace",pointerEvents:"none"}}>
    {payload.map((p,i)=><div key={i} style={{color:p.color||C.cyan,whiteSpace:"nowrap"}}>
      {p.name}: {p.value?.toLocaleString()}
    </div>)}
  </div>;
};

/* ─── HEADER ─────────────────────────────────────────────────────────── */
const Header=memo(function Header({display,price}){
  const{change24h,vol24h,oi,fundRate,connected,loading,markPrice}=display;
  const up=change24h>=0;
  return <div style={{background:"rgba(2,7,17,0.99)",borderBottom:`1px solid ${C.border}`,
    padding:"9px 14px",position:"sticky",top:0,zIndex:100,
    willChange:"transform",backfaceVisibility:"hidden"}}>
    <div style={{display:"flex",alignItems:"center",flexWrap:"wrap",gap:"8px 16px"}}>
      <div style={{display:"flex",alignItems:"baseline",gap:7,minWidth:0}}>
        <span style={{fontSize:9,fontFamily:"'Rajdhani',sans-serif",fontWeight:700,
          color:C.dim,letterSpacing:"0.18em",flexShrink:0}}>BTCUSDT PERP</span>
        {/* Fixed-width price — never causes layout shift */}
        <span style={{fontSize:25,fontFamily:"'Space Mono',monospace",fontWeight:700,
          color:C.bright,letterSpacing:"-0.02em",minWidth:"11ch",
          fontVariantNumeric:"tabular-nums",display:"inline-block"}}>
          {loading?"···":("$"+price.toLocaleString())}
        </span>
        <span style={{fontSize:12,fontFamily:"'Space Mono',monospace",fontWeight:700,
          color:up?C.green:C.red,minWidth:"7ch",display:"inline-block",
          fontVariantNumeric:"tabular-nums"}}>{up?"+":""}{change24h.toFixed(2)}%</span>
      </div>
      <div style={{display:"flex",gap:12,flexWrap:"wrap",flex:1}}>
        {[["Vol",(vol24h).toFixed(1)+"B"],["OI","$"+oi.toFixed(1)+"B"],
          ["Fund",(fundRate>=0?"+":"")+fundRate.toFixed(4)+"%"]
        ].map(([k,v])=><div key={k} style={{flexShrink:0}}>
          <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{k}</div>
          <div style={{fontSize:11,fontFamily:"'Space Mono',monospace",fontWeight:700,
            color:k==="Fund"?(fundRate>=0?C.gold:C.cyan):C.text,
            minWidth:"7ch",fontVariantNumeric:"tabular-nums"}}>{v}</div>
        </div>)}
      </div>
      <div style={{display:"flex",alignItems:"center",gap:5,flexShrink:0}}>
        <Dot col={connected?C.green:C.red} pulse={connected}/>
        <span style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",
          letterSpacing:"0.1em",minWidth:"9ch"}}>{connected?"LIVE":"RECONNECT"}</span>
      </div>
    </div>
  </div>;
});

/* ─── MARKET OVERVIEW ────────────────────────────────────────────────── */
const MarketOverview=memo(function MarketOverview({kRef,candleVer,price,display,regime}){
  // Chart data — only updates on closed candle
  const chartData=useMemo(()=>{
    const k=kRef.current.k1m;
    if(!k) return [];
    return closedOnly(k).slice(-80).map(r=>({
      p:parseFloat(r[4]),v:parseFloat(r[5]),
      t:new Date(r[0]).toLocaleTimeString("en",{hour:"2-digit",minute:"2-digit"})
    }));
  },[candleVer]);

  const rsiCol=regime.rsi>70?C.red:regime.rsi<30?C.green:C.cyan;
  const ind=[
    ["ATR(14)",regime.atr?("$"+Math.round(regime.atr).toLocaleString()):"—",C.gold],
    ["RSI(14)",regime.rsi?regime.rsi.toFixed(1):"—",rsiCol],
    ["EMA21",regime.e21?("$"+(Math.round(regime.e21/100)*100).toLocaleString()):"—",C.text],
    ["EMA50",regime.e50?("$"+(Math.round(regime.e50/100)*100).toLocaleString()):"—",C.text],
    ["BB Width",regime.bbWidth?(regime.bbWidth*100).toFixed(3)+"%":"—",C.purple],
    ["Trend",regime.dir||"—",regime.dir==="BULLISH"?C.green:regime.dir==="BEARISH"?C.red:C.gold],
  ];
  return <section>
    <SecTitle ch="Market Overview" sub="Closed candles only — no repaint"/>
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7,marginBottom:10}}>
      {[["Price","$"+price.toLocaleString(),display.change24h>=0?C.green:C.red],
        ["24h %",(display.change24h>=0?"+":"")+display.change24h.toFixed(2)+"%",display.change24h>=0?C.green:C.red],
        ["Regime",regime.regime?.replace(/_/g," ")||"LOADING",C.cyan],
        ["High","$"+Math.round(display.high24h).toLocaleString(),C.text],
        ["Low","$"+Math.round(display.low24h).toLocaleString(),C.text],
        ["Conf",(regime.conf*100).toFixed(0)+"%",C.gold],
      ].map(([l,v,c])=><Box key={l} style={{padding:"9px 11px"}}>
        <Label ch={l}/><Num v={v} col={c} sz={13} w="8ch"/>
      </Box>)}
    </div>
    {chartData.length>0&&<Box style={{padding:"12px 13px",marginBottom:9}}>
      <Label ch="1m Closed-Candle Chart" style={{marginBottom:5}}/>
      <div style={{height:145}}>
        <ResponsiveContainer width="100%" height="100%">
          <AreaChart data={chartData} margin={{top:3,right:3,bottom:0,left:0}}>
            <defs><linearGradient id="cg1" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor={C.cyan} stopOpacity={0.25}/>
              <stop offset="95%" stopColor={C.cyan} stopOpacity={0.01}/></linearGradient></defs>
            <XAxis dataKey="t" tick={{fill:C.dim,fontSize:8,fontFamily:"'Space Mono',monospace"}}
              tickLine={false} axisLine={false} interval={19}/>
            <YAxis domain={["auto","auto"]} tick={{fill:C.dim,fontSize:8,fontFamily:"'Space Mono',monospace"}}
              tickLine={false} axisLine={false} tickFormatter={v=>`${(v/1000).toFixed(0)}K`} width={33}/>
            <Tooltip content={<ChartTip/>}/>
            <Area type="monotone" dataKey="p" name="Price" stroke={C.cyan}
              strokeWidth={1.4} fill="url(#cg1)" dot={false} isAnimationActive={false}/>
          </AreaChart>
        </ResponsiveContainer>
      </div>
    </Box>}
    {chartData.length>0&&<Box style={{padding:"9px 13px",marginBottom:9}}>
      <Label ch="Volume (closed candles)" style={{marginBottom:4}}/>
      <div style={{height:48}}>
        <ResponsiveContainer width="100%" height="100%">
          <BarChart data={chartData.slice(-60)} margin={{top:0,right:3,bottom:0,left:0}}>
            <Bar dataKey="v" radius={[1,1,0,0]} isAnimationActive={false}>
              {chartData.slice(-60).map((d,i)=><Cell key={i}
                fill={i>0&&d.p>=chartData.slice(-60)[i-1]?.p?`${C.green}65`:`${C.red}60`}/>)}
            </Bar>
          </BarChart>
        </ResponsiveContainer>
      </div>
    </Box>}
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:6}}>
      {ind.map(([l,v,c])=><Box key={l} style={{padding:"8px 10px"}}>
        <Label ch={l} style={{fontSize:8}}/><Num v={v} col={c} sz={12} w="7ch"/>
      </Box>)}
    </div>
  </section>;
});

/* ─── AI BRAIN ───────────────────────────────────────────────────────── */
// Agent scores are derived from REGIME only — computed once per REGIME update (30s TTL)
// No random drift — scores are deterministic from live data
const AIBrain=memo(function AIBrain({regime,of,signals}){
  // Derived scores — stable, no randomness
  const agents=useMemo(()=>{
    const consensus=[
      signals.snp?.finalConf||0,
      signals.int?.finalConf||0,
      signals.trd?.finalConf||0
    ].filter(v=>v>0);
    const consScore=consensus.length?consensus.reduce((s,v)=>s+v)/consensus.length:0;
    const rsiNorm=regime.rsi?Math.abs(50-regime.rsi)/50:0;
    const ofScore=Math.abs((of.score||0.5)-0.5)*2;
    return [
      {name:"Market State",  role:"Regime",       score:regime.conf,    verdict:regime.regime?.replace(/_/g," ")||"LOADING", col:regime.conf>0.7?C.green:C.gold},
      {name:"Trend Analysis",role:"Direction",     score:regime.conf*0.92, verdict:regime.dir||"NEUTRAL", col:regime.dir==="BULLISH"?C.green:regime.dir==="BEARISH"?C.red:C.gold},
      {name:"Momentum",      role:"RSI+MACD",      score:Math.min(0.95,0.35+rsiNorm*0.55), verdict:regime.rsi>55?"ACCELERATING":regime.rsi<45?"WEAKENING":"NEUTRAL", col:regime.rsi>55?C.green:regime.rsi<45?C.orange:C.dim},
      {name:"Liquidity AI",  role:"Smart Money",   score:Math.min(0.95,0.35+ofScore*0.55), verdict:of.pressure||"NEUTRAL", col:of.score>0.55?C.cyan:of.score<0.45?C.orange:C.dim},
      {name:"Order Flow",    role:"Depth",         score:Math.min(0.95,0.30+ofScore*0.65), verdict:of.score>0.55?"BUY DOM":of.score<0.45?"SELL DOM":"BALANCED", col:of.score>0.55?C.green:of.score<0.45?C.red:C.dim},
      {name:"Quant Edge",    role:"Statistics",    score:Math.min(0.95,regime.conf*0.90),  verdict:regime.conf>0.7?"FAVORABLE":"MIXED", col:regime.conf>0.7?C.cyan:C.gold},
      {name:"Risk Manager",  role:"Risk Eval",     score:consScore>0?0.88:0.5,  verdict:consScore>0?"APPROVED":"STANDBY", col:consScore>0?C.green:C.dim},
      {name:"Coordinator",   role:"Final Vote",    score:consScore,
        verdict:consScore>0.75?"EXECUTE"+(signals.snp?.dir||signals.int?.dir||signals.trd?.dir?" "+(signals.snp?.dir||signals.int?.dir||signals.trd?.dir):""):consScore>0.60?"WATCH":"NO TRADE",
        col:consScore>0.75?C.gold:consScore>0.60?C.cyan:C.dim},
    ];
  },[regime,of,signals]); // only updates when regime/of/signals change

  const consensus=agents[7].score;
  return <section>
    <SecTitle ch="AI Market Brain" sub="8 agents · updates on closed candle only"/>
    <Box style={{marginBottom:9,padding:"13px 15px"}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:9}}>
        <div><Label ch="Consensus Score"/>
          <Num v={(consensus*100).toFixed(0)+"%" } col={consensus>0.75?C.gold:consensus>0.60?C.cyan:C.dim} sz={28} w="5ch"/>
        </div>
        <div style={{textAlign:"right"}}>
          <Label ch="Final Verdict"/>
          <Badge ch={agents[7].verdict} col={agents[7].col}/>
        </div>
      </div>
      <div style={{height:3,background:C.muted,borderRadius:2}}>
        <div style={{width:`${consensus*100}%`,height:"100%",
          background:`linear-gradient(90deg,${C.cyan},${C.gold})`,borderRadius:2,
          transition:"width 0.8s ease"}}/>
      </div>
    </Box>
    <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:6}}>
      {agents.map(a=><Box key={a.name} style={{padding:"9px 11px"}}>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:4}}>
          <div>
            <div style={{fontSize:11,fontFamily:"'Rajdhani',sans-serif",fontWeight:700,color:C.bright}}>{a.name}</div>
            <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{a.role}</div>
          </div>
          <Num v={(a.score*100).toFixed(0)+"%"} col={a.col} sz={11} w="4ch"/>
        </div>
        <div style={{height:2,background:C.muted,borderRadius:1,marginBottom:4}}>
          <div style={{width:`${a.score*100}%`,height:"100%",background:a.col,borderRadius:1,transition:"width 0.8s ease"}}/>
        </div>
        <Badge ch={a.verdict} col={a.col} sm/>
      </Box>)}
    </div>
  </section>;
});

/* ─── STRATEGY INTEL ─────────────────────────────────────────────────── */
const StrategyIntel=memo(function StrategyIntel({regime,sw}){
  const entries=[
    {k:"snp",name:"Sniper Scalper",desc:"1–15 min",col:C.cyan},
    {k:"int",name:"Intraday Hunter",desc:"30m–4h",col:C.gold},
    {k:"trd",name:"Trend Rider",desc:"4h–days",col:C.purple},
  ];
  const regimeList=["STRONG_BULL","MODERATE_BULL","RANGE_BOUND","HIGH_VOLATILITY","COMPRESSION"];
  const regimeCol={STRONG_BULL:C.green,MODERATE_BULL:C.green,RANGE_BOUND:C.gold,HIGH_VOLATILITY:C.orange,COMPRESSION:C.cyan};
  return <section>
    <SecTitle ch="Strategy Intelligence" sub="Weight driven by live regime"/>
    <Box style={{marginBottom:9,padding:"12px 14px"}}>
      <Label ch="Detected Regime" style={{marginBottom:6}}/>
      <div style={{display:"flex",gap:5,flexWrap:"wrap",marginBottom:7}}>
        {regimeList.map(r=><span key={r} style={{fontSize:9,fontFamily:"'Space Mono',monospace",
          padding:"2px 7px",borderRadius:3,whiteSpace:"nowrap",
          background:regime.regime===r?`${regimeCol[r]}1E`:C.muted,
          border:`1px solid ${regime.regime===r?regimeCol[r]:C.border}`,
          color:regime.regime===r?regimeCol[r]:C.dim}}>
          {r.replace(/_/g," ")}</span>)}
      </div>
      <div style={{fontSize:10,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
        <span style={{color:regime.dir==="BULLISH"?C.green:regime.dir==="BEARISH"?C.red:C.gold}}>{regime.dir}</span>
        {" · Conf: "}<span style={{color:C.text}}>{(regime.conf*100).toFixed(0)}%</span>
      </div>
    </Box>
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7}}>
      {entries.map(s=><Box key={s.k} style={{padding:"10px 11px",
        border:`1px solid ${(sw[s.k]||0.33)>0.38?s.col+"40":C.border}`}}>
        <Label ch={s.desc} style={{fontSize:8}}/>
        <div style={{fontSize:11,fontFamily:"'Rajdhani',sans-serif",fontWeight:700,color:s.col,marginBottom:4}}>{s.name}</div>
        <Num v={((sw[s.k]||0.33)*100).toFixed(0)+"%"} col={s.col} sz={17} w="4ch"/>
        <div style={{height:2,background:C.muted,borderRadius:1,marginTop:5}}>
          <div style={{width:`${(sw[s.k]||0.33)*100}%`,height:"100%",background:s.col,borderRadius:1,transition:"width 0.8s ease"}}/>
        </div>
      </Box>)}
    </div>
  </section>;
});

/* ─── SIGNAL CARD ────────────────────────────────────────────────────── */
const SignalCard=memo(function SignalCard({sig}){
  if(!sig) return <Box style={{padding:"13px",contain:"layout style"}}>
    <Label ch="Scanning market..." style={{textAlign:"center",marginBottom:3}}/>
    <div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace",textAlign:"center"}}>
      Waiting for closed-candle signal alignment
    </div>
  </Box>;
  const{card,dir,entry,sl,tp,conf,finalConf,rr,pw,atr,strat,expire,shap,mc,risk,col}=sig;
  const fc=finalConf||conf;
  const cCol=fc>=0.80?C.green:fc>=0.70?C.gold:C.orange;
  const slPct=((Math.abs(entry-sl)/entry)*100).toFixed(2);
  return <Box style={{marginBottom:9,border:`1px solid ${col}22`,padding:"13px 14px",contain:"layout style"}}>
    <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:9}}>
      <div>
        <div style={{fontSize:8,fontFamily:"'Space Mono',monospace",letterSpacing:"0.14em",color:col,marginBottom:3}}>{card}</div>
        <div style={{display:"flex",alignItems:"center",gap:5}}>
          <Badge ch={dir} col={dir==="LONG"?C.green:C.red}/>
          <span style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{strat}</span>
        </div>
      </div>
      <div style={{textAlign:"right",flexShrink:0}}>
        <Label ch="Confidence"/>
        <Num v={(fc*100).toFixed(1)+"%" } col={cCol} sz={20} w="5ch"/>
        <div style={{fontSize:8,color:fc>=ALERT_THRESH?C.gold:C.dim,fontFamily:"'Space Mono',monospace"}}>
          {fc>=ALERT_THRESH?"⚡ ALERT":"visible"}
        </div>
      </div>
    </div>
    <div style={{height:2,background:C.muted,borderRadius:1,marginBottom:9}}>
      <div style={{width:`${fc*100}%`,height:"100%",background:cCol,borderRadius:1}}/>
    </div>
    {/* Entry / SL row */}
    <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:7,marginBottom:7}}>
      <Box style={{padding:"8px 10px",background:`${C.green}09`,borderRadius:7}}>
        <Label ch="Entry" style={{fontSize:8,color:C.green}}/>
        <Num v={"$"+entry.toLocaleString()} col={C.bright} sz={13} w="9ch"/>
      </Box>
      <Box style={{padding:"8px 10px",background:`${C.red}09`,borderRadius:7}}>
        <Label ch={"Stop −"+slPct+"%"} style={{fontSize:8,color:C.red}}/>
        <Num v={"$"+sl.toLocaleString()} col={C.red} sz={13} w="9ch"/>
      </Box>
    </div>
    {/* Take profits */}
    <div style={{display:"grid",gridTemplateColumns:`repeat(${tp.length},1fr)`,gap:5,marginBottom:9}}>
      {tp.map((t,i)=><Box key={i} style={{padding:"6px 8px",background:`${C.green}07`,borderRadius:6}}>
        <Label ch={"TP"+(i+1)} style={{fontSize:7,color:C.green}}/>
        <Num v={"$"+t.toLocaleString()} col={C.green} sz={11} w="8ch"/>
        <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",
          fontVariantNumeric:"tabular-nums"}}>
          +{((Math.abs(t-entry)/entry)*100).toFixed(2)}%
        </div>
      </Box>)}
    </div>
    {/* Chips */}
    <div style={{display:"flex",flexWrap:"wrap",gap:4,marginBottom:9}}>
      {[["R:R",rr+"x"],["Win%",(pw*100).toFixed(0)+"%"],["EV",risk?(risk.ev>0?"+":"")+Math.round(risk.ev):"—"],
        ["Kelly",risk?risk.kelly:"—"],["Fill",risk?risk.fillProb:"—"],["ATR","$"+Math.round(atr)]
      ].map(([k,v])=><div key={k} style={{background:C.muted+"55",borderRadius:3,padding:"2px 6px",flexShrink:0}}>
        <span style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{k}: </span>
        <span style={{fontSize:8,fontWeight:700,fontFamily:"'Space Mono',monospace",
          color:C.text,fontVariantNumeric:"tabular-nums"}}>{v}</span>
      </div>)}
    </div>
    {/* SHAP */}
    {shap?.length>0&&<div style={{marginBottom:8}}>
      <Label ch="Feature Contributions" style={{fontSize:8,marginBottom:4}}/>
      {shap.map(s=><div key={s.f} style={{display:"flex",alignItems:"center",gap:5,marginBottom:3}}>
        <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",width:120,flexShrink:0}}>{s.f}</div>
        <div style={{flex:1,height:2,background:C.muted,borderRadius:1}}>
          <div style={{width:`${Math.min(100,s.v*320)}%`,height:"100%",background:col,borderRadius:1}}/>
        </div>
        <div style={{fontSize:8,fontFamily:"'Space Mono',monospace",color:col,width:28,textAlign:"right",
          fontVariantNumeric:"tabular-nums"}}>{s.v.toFixed(2)}</div>
      </div>)}
    </div>}
    {/* MC mini chart */}
    {mc?.mcData?.length>0&&<>
      <HR/>
      <div style={{height:50,marginBottom:5}}>
        <ResponsiveContainer width="100%" height="100%">
          <AreaChart data={mc.mcData} margin={{top:2,right:0,bottom:0,left:0}}>
            <Area type="monotone" dataKey="p90" stroke={`${C.green}80`} strokeWidth={0.8} fill={`${C.green}0E`} dot={false} isAnimationActive={false}/>
            <Area type="monotone" dataKey="p50" stroke={C.gold} strokeWidth={1.5} fill="none" dot={false} isAnimationActive={false}/>
            <Area type="monotone" dataKey="p10" stroke={`${C.red}80`} strokeWidth={0.8} fill={`${C.red}08`} dot={false} isAnimationActive={false}/>
          </AreaChart>
        </ResponsiveContainer>
      </div>
      <div style={{display:"flex",justifyContent:"space-between",fontSize:8,
        fontFamily:"'Space Mono',monospace",color:C.dim}}>
        <span>TP <span style={{color:C.green}}>{(mc.probTP*100).toFixed(0)}%</span></span>
        <span>SL <span style={{color:C.red}}>{(mc.probSL*100).toFixed(0)}%</span></span>
        <span>EV <span style={{color:mc.ev>0?C.green:C.red}}>{mc.ev>0?"+":""}{Math.round(mc.ev)}</span></span>
      </div>
    </>}
    <HR/>
    <div style={{fontSize:8,fontFamily:"'Space Mono',monospace",color:C.dim,display:"flex",justifyContent:"space-between"}}>
      <span>Closed candle · no repaint</span>
      <span>Expires: <span style={{color:C.orange}}>{expire>3600?Math.round(expire/3600)+"h":Math.round(expire/60)+"m"}</span></span>
    </div>
  </Box>;
});

/* ─── ACTIVE TRADES ──────────────────────────────────────────────────── */
// All trades are locked once opened. Chandelier ATR trailing stop.
function ActiveTrades({trades,dispatch}){
  const open=trades.filter(t=>t.status==="OPEN");
  const closed=trades.filter(t=>t.status==="CLOSED").slice(-5);
  if(!trades.length) return <Box style={{padding:"13px",textAlign:"center"}}>
    <Label ch="No active trades" style={{textAlign:"center"}}/> 
    <div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace",marginTop:3}}>Trades open automatically when signals pass alert threshold</div>
  </Box>;
  return <div>
    {open.map(t=>{
      const pnlPos=t.pnlPct>=0;
      const prog=Math.min(100,Math.max(0,t.dir==="LONG"?(t.currentPrice-t.entry)/(t.tp[t.tp.length-1]-t.entry)*100:(t.entry-t.currentPrice)/(t.entry-t.tp[t.tp.length-1])*100));
      const distToTS=Math.abs((t.currentPrice||t.entry)-t.trailStop);
      const tsDistPct=((distToTS/(t.currentPrice||t.entry))*100).toFixed(2);
      const dur=Math.round((Date.now()-t.openedAt)/60000);
      return <Box key={t.id} style={{marginBottom:8,border:`1px solid ${t.col}35`,padding:"12px 14px",contain:"layout style"}}>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:8}}>
          <div>
            <div style={{fontSize:8,color:t.col,fontFamily:"'Space Mono',monospace",marginBottom:2}}>{t.card}</div>
            <div style={{display:"flex",gap:5,alignItems:"center"}}>
              <Badge ch={t.dir} col={t.dir==="LONG"?C.green:C.red} sm/>
              <span style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{dur}m ago</span>
              <Dot col={C.green} pulse/>
            </div>
          </div>
          <div style={{textAlign:"right"}}>
            <Num v={(t.pnlPct>=0?"+":"")+t.pnlPct?.toFixed(2)+"%" } col={pnlPos?C.green:C.red} sz={16} w="7ch"/>
            <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",
              fontVariantNumeric:"tabular-nums"}}>
              {t.pnlUsd>=0?"+":""}${t.pnlUsd?.toLocaleString()||0}
            </div>
          </div>
        </div>
        {/* Price progress bar */}
        <div style={{height:3,background:C.muted,borderRadius:2,marginBottom:8}}>
          <div style={{width:prog+"%",height:"100%",background:pnlPos?C.green:C.red,borderRadius:2,transition:"width 0.4s"}}/>
        </div>
        {/* Levels grid */}
        <div style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:5,marginBottom:7}}>
          <div><Label ch="Entry" style={{fontSize:7}}/><Num v={"$"+t.entry.toLocaleString()} col={C.bright} sz={10} w="8ch"/></div>
          <div><Label ch="Current" style={{fontSize:7}}/><Num v={"$"+(t.currentPrice||t.entry).toLocaleString()} col={pnlPos?C.green:C.red} sz={10} w="8ch"/></div>
          <div>
            <Label ch="⚡ Trail Stop" style={{fontSize:7,color:C.gold}}/>
            <Num v={"$"+t.trailStop.toLocaleString()} col={C.gold} sz={10} w="8ch"/>
          </div>
          <div><Label ch={"TS dist"} style={{fontSize:7}}/><Num v={tsDistPct+"%"} col={C.orange} sz={10} w="5ch"/></div>
        </div>
        {/* TPs */}
        <div style={{display:"flex",gap:5,flexWrap:"wrap",marginBottom:7}}>
          {t.tp.map((tp,i)=>{
            const hit=i<t.tpsHit;
            const isCurrent=t.dir==="LONG"?(t.currentPrice||t.entry)>=tp:(t.currentPrice||t.entry)<=tp;
            return <div key={i} style={{background:hit?`${C.green}28`:C.muted+"40",
              border:`1px solid ${hit?C.green:C.border}`,borderRadius:4,padding:"3px 7px"}}>
              <div style={{fontSize:7,color:hit?C.green:C.dim,fontFamily:"'Space Mono',monospace"}}>TP{i+1} {hit?"✓":""}</div>
              <div style={{fontSize:9,fontFamily:"'Space Mono',monospace",fontWeight:700,
                color:hit?C.green:C.text,fontVariantNumeric:"tabular-nums"}}>${tp.toLocaleString()}</div>
            </div>;
          })}
        </div>
        {/* Chandelier info */}
        <div style={{background:`${C.gold}0A`,border:`1px solid ${C.gold}20`,borderRadius:5,padding:"5px 9px"}}>
          <div style={{fontSize:8,color:C.gold,fontFamily:"'Space Mono',monospace",marginBottom:2}}>⚡ Chandelier Exit (ATR×{CHANDELIER_M})</div>
          <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
            Trail: ${t.trailStop.toLocaleString()} · {t.dir==="LONG"?"Ratchets UP only — never decreases":"Ratchets DOWN only — never increases"}
          </div>
        </div>
        <div style={{display:"flex",gap:6,marginTop:7}}>
          <button onClick={()=>dispatch({type:"CLOSE",id:t.id,outcome:"MANUAL",price:t.currentPrice||t.entry})}
            style={{flex:1,fontSize:9,fontFamily:"'Space Mono',monospace",fontWeight:700,
              color:C.red,background:`${C.red}12`,border:`1px solid ${C.red}40`,
              borderRadius:5,padding:"5px 0",cursor:"pointer"}}>CLOSE TRADE</button>
        </div>
      </Box>;
    })}
    {closed.length>0&&<Box style={{padding:"11px 13px"}}>
      <Label ch="Closed Trades (this session)" style={{marginBottom:7}}/>
      {closed.reverse().map((t,i)=><div key={i} style={{display:"flex",alignItems:"center",gap:6,
        padding:"5px 0",borderBottom:i<closed.length-1?`1px solid ${C.border}`:"none"}}>
        <Badge ch={t.id} col={t.id==="SNP"?C.cyan:t.id==="INT"?C.gold:C.purple} sm/>
        <Badge ch={t.dir} col={t.dir==="LONG"?C.green:C.red} sm/>
        <div style={{flex:1,fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
          ${t.entry?.toLocaleString()} → ${t.closePrice?.toLocaleString()}
        </div>
        <Badge ch={t.outcome} col={t.outcome==="SL"?C.red:C.green} sm/>
        <div style={{fontSize:9,fontFamily:"'Space Mono',monospace",
          color:t.closePrice>=t.entry===t.dir==="LONG"?C.green:C.red,
          fontVariantNumeric:"tabular-nums"}}>
          {t.closePrice&&t.entry?((t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100).toFixed(2)+"%":"—"}
        </div>
      </div>)}
    </Box>}
  </div>;
}

/* ─── TRADE MANAGEMENT AGENT UI ─────────────────────────────────────── */
const ModeColors={NORMAL:C.green,PROTECTIVE:C.orange,RESTRICTED:C.orange,PAUSED:C.red};
const SevColors={OK:C.green,INFO:C.cyan,WARN:C.orange,ERROR:C.red};
const PriColors={HIGH:C.gold,MEDIUM:C.orange,INFO:C.cyan,LOW:C.dim};

const VerdictBadge=memo(function VerdictBadge({v}){
  const col=v==="APPROVED"?C.green:v==="BLOCKED"?C.red:v==="DENIED"?C.red:v==="NO_SIGNAL"?C.dim:C.gold;
  return <Badge ch={v} col={col} sm/>;
});

const TradeManagementAgent=memo(function TradeManagementAgent({tma,dispatch,signals}){
  if(!tma) return null;
  const{snp,int,trd,directives,audit,recs,globalMode,globalReasons,sessionStats}=tma;
  const modeCol=ModeColors[globalMode]||C.cyan;
  const verdicts=[
    {v:snp,sig:signals.snp,id:"SNP",col:C.cyan,label:"Sniper"},
    {v:int,sig:signals.int,id:"INT",col:C.gold,label:"Intraday"},
    {v:trd,sig:signals.trd,id:"TRD",col:C.purple,label:"Trend"},
  ];

  return <section>
    <SecTitle ch="Trade Management Agent" sub="Institutional gatekeeper · confidence gate · trade directives"/>

    {/* Global Mode Banner */}
    <Box style={{marginBottom:9,padding:"11px 14px",border:`1px solid ${modeCol}35`,
      background:`${modeCol}08`}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
        <div style={{display:"flex",alignItems:"center",gap:7}}>
          <Dot col={modeCol} pulse={globalMode!=="NORMAL"}/>
          <div>
            <div style={{fontSize:11,fontFamily:"'Rajdhani',sans-serif",fontWeight:700,
              color:modeCol,letterSpacing:"0.1em"}}>TMA MODE: {globalMode}</div>
            {globalReasons[0]&&<div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace",marginTop:2}}>{globalReasons[0]}</div>}
          </div>
        </div>
        <div style={{textAlign:"right"}}>
          <Label ch="Session WR"/>
          <Num v={sessionStats.wr!==null?(sessionStats.wr*100).toFixed(0)+"%":"—"}
            col={sessionStats.wr===null?C.dim:sessionStats.wr>0.6?C.green:sessionStats.wr>0.4?C.gold:C.red}
            sz={16} w="4ch"/>
        </div>
      </div>
      <HR/>
      <div style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:6}}>
        {[["Wins",sessionStats.wins,C.green],["Losses",sessionStats.losses,C.red],
          ["Streak",sessionStats.streak>0?"-"+sessionStats.streak:"+",sessionStats.streak>=2?C.red:C.dim],
          ["Open",sessionStats.openCount,C.cyan],
        ].map(([l,v,c])=><div key={l}><Label ch={l} style={{fontSize:7}}/><Num v={v} col={c} sz={13} w="3ch"/></div>)}
      </div>
    </Box>

    {/* Signal Verdicts */}
    <Box style={{marginBottom:9,padding:"11px 13px"}}>
      <Label ch="Signal Gate Verdicts" style={{marginBottom:8}}/>
      {verdicts.map(({v,sig,id,col,label})=><div key={id}
        style={{display:"flex",flexWrap:"wrap",gap:5,alignItems:"flex-start",
          padding:"8px 0",borderBottom:`1px solid ${C.border}`}}>
        <div style={{display:"flex",alignItems:"center",gap:5,minWidth:90,flexShrink:0}}>
          <div style={{width:3,height:13,background:col,borderRadius:2,flexShrink:0}}/>
          <span style={{fontSize:10,fontFamily:"'Rajdhani',sans-serif",fontWeight:700,color:col}}>{label}</span>
        </div>
        <VerdictBadge v={v.verdict}/>
        {sig&&<span style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
          {sig.finalConf?(sig.finalConf*100).toFixed(0)+"%":""} · {sig.rr?sig.rr+"x":""} R:R
        </span>}
        <div style={{width:"100%",paddingLeft:14}}>
          {v.reasons.map((r,i)=><div key={i} style={{fontSize:8,color:C.red,fontFamily:"'Space Mono',monospace",
            marginBottom:1,display:"flex",gap:4}}>
            <span style={{flexShrink:0}}>✗</span><span>{r}</span>
          </div>)}
          {v.adjustments?.map((a,i)=><div key={i} style={{fontSize:8,color:C.gold,fontFamily:"'Space Mono',monospace",
            marginBottom:1,display:"flex",gap:4}}>
            <span style={{flexShrink:0}}>→</span><span>{a}</span>
          </div>)}
        </div>
      </div>)}
    </Box>

    {/* Active Trade Directives */}
    {directives.length>0&&<Box style={{marginBottom:9,padding:"11px 13px"}}>
      <Label ch="Active Trade Directives" style={{marginBottom:8}}/>
      {directives.map((d,i)=><div key={i} style={{display:"flex",alignItems:"flex-start",gap:7,
        padding:"8px 10px",marginBottom:5,borderRadius:6,
        background:`${d.col}0C`,border:`1px solid ${d.col}28`}}>
        <div style={{flexShrink:0,marginTop:1}}>
          <div style={{width:6,height:6,borderRadius:"50%",background:d.col,
            boxShadow:d.priority==="HIGH"?`0 0 6px ${d.col}`:"none"}}/>
        </div>
        <div style={{flex:1,minWidth:0}}>
          <div style={{display:"flex",alignItems:"center",gap:5,marginBottom:2}}>
            <span style={{fontSize:9,fontFamily:"'Rajdhani',sans-serif",fontWeight:700,color:d.col}}>{d.label}</span>
            <Badge ch={d.id} col={d.id==="SNP"?C.cyan:d.id==="INT"?C.gold:C.purple} sm/>
            {d.priority!=="INFO"&&<Badge ch={d.priority} col={PriColors[d.priority]||C.dim} sm/>}
          </div>
          <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",lineHeight:1.4}}>{d.desc}</div>
          {(d.action==="MOVE_BE"||d.action==="PARTIAL_CLOSE")&&<div style={{display:"flex",gap:5,marginTop:5}}>
            <button onClick={()=>dispatch({type:d.action,id:d.id})}
              style={{fontSize:8,fontFamily:"'Space Mono',monospace",fontWeight:700,
                color:d.col,background:`${d.col}15`,border:`1px solid ${d.col}40`,
                borderRadius:4,padding:"3px 8px",cursor:"pointer"}}>{d.label.toUpperCase()}</button>
          </div>}
        </div>
      </div>)}
    </Box>}
    {directives.length===0&&<Box style={{padding:"10px 13px",marginBottom:9,textAlign:"center"}}>
      <div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>No active trade directives</div>
    </Box>}

    {/* Agent Audit */}
    <Box style={{marginBottom:9,padding:"11px 13px"}}>
      <Label ch="Agent Logic Audit" style={{marginBottom:7}}/>
      {audit.map((a,i)=><div key={i} style={{display:"flex",alignItems:"flex-start",gap:6,
        padding:"5px 0",borderBottom:i<audit.length-1?`1px solid ${C.border}`:"none"}}>
        <div style={{flexShrink:0,marginTop:2}}>
          <div style={{width:5,height:5,borderRadius:"50%",background:SevColors[a.sev]||C.dim}}/>
        </div>
        <div>
          <div style={{fontSize:8,color:SevColors[a.sev]||C.dim,fontFamily:"'Space Mono',monospace",
            fontWeight:700,marginBottom:1}}>{a.agent}</div>
          <div style={{fontSize:8,color:C.text,fontFamily:"'Space Mono',monospace",lineHeight:1.4}}>{a.msg}</div>
        </div>
      </div>)}
    </Box>

    {/* Recommendations */}
    {recs.length>0&&<Box style={{padding:"11px 13px"}}>
      <Label ch="TMA Recommendations" style={{marginBottom:7}}/>
      {recs.map((r,i)=><div key={i} style={{display:"flex",gap:7,alignItems:"flex-start",
        padding:"5px 0",borderBottom:i<recs.length-1?`1px solid ${C.border}`:"none"}}>
        <span style={{fontSize:11,color:r.col,flexShrink:0,lineHeight:1.3}}>{r.icon}</span>
        <span style={{fontSize:9,color:C.text,fontFamily:"'Space Mono',monospace",lineHeight:1.5}}>{r.text}</span>
      </div>)}
    </Box>}
  </section>;
});

/* ─── HEATMAP ────────────────────────────────────────────────────────── */
const HeatmapSection=memo(function HeatmapSection({kRef,candleVer,price}){
  const zones=useMemo(()=>{
    const k1h=closedOnly(kRef.current.k1h);
    if(!k1h?.length||!price) return [];
    const hi=k1h.map(r=>parseFloat(r[2])),lo=k1h.map(r=>parseFloat(r[3])),cl=k1h.map(r=>parseFloat(r[4]));
    const {sh,sl}=TA.swings(hi,lo,4);
    const atrArr=TA.atr(hi,lo,cl,14),atr=atrArr[atrArr.length-1];
    const fib=TA.fibonacci(Math.max(...hi.slice(-50)),Math.min(...lo.slice(-50)));
    const z=[];
    sh.slice(-5).forEach(s=>z.push({y:Math.round(s.p),label:"Swing High",type:"res",str:0.70}));
    sl.slice(-5).forEach(s=>z.push({y:Math.round(s.p),label:"Swing Low",type:"sup",str:0.70}));
    z.push({y:Math.round(fib.r382),label:"Fib 38.2%",type:"fib",str:0.55});
    z.push({y:Math.round(fib.r618),label:"Fib 61.8%",type:"fib",str:0.62});
    z.push({y:Math.round(price+atr*2),label:"Liq Pool High",type:"liq",str:0.65});
    z.push({y:Math.round(price-atr*2),label:"Liq Pool Low",type:"liq",str:0.65});
    z.push({y:Math.round(price),label:"Price",type:"price",str:1});
    return z.sort((a,b)=>b.y-a.y);
  },[candleVer,price]);
  const mn=price*0.945,mx=price*1.055;
  const toY=(p)=>Math.round(((mx-p)/(mx-mn))*240+20);
  const tCol=(t)=>({res:C.red,sup:C.green,liq:C.gold,price:C.cyan,fib:C.purple}[t]||C.text);
  return <section>
    <SecTitle ch="Market Heatmap" sub="Live swing highs/lows + Fibonacci + ATR liquidity bands"/>
    <Box style={{padding:"13px 14px"}}>
      <div style={{display:"flex",gap:7,marginBottom:9,flexWrap:"wrap"}}>
        {[["Resistance",C.red],["Support",C.green],["Liquidity",C.gold],["Fibonacci",C.purple],["Price",C.cyan]].map(([l,c])=>
          <div key={l} style={{display:"flex",alignItems:"center",gap:3}}>
            <div style={{width:7,height:7,borderRadius:1,background:c,opacity:0.75,flexShrink:0}}/>
            <span style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{l}</span>
          </div>)}
      </div>
      {price>0?<svg width="100%" height="270" viewBox="0 0 330 270" style={{display:"block"}}>
        {[0.04,0.02,0,-0.02,-0.04].map(pct=>{
          const lp=Math.round(price*(1+pct)),y=toY(lp);
          return y>0&&y<270?<g key={pct}>
            <line x1="52" y1={y} x2="330" y2={y} stroke={C.muted} strokeWidth="0.5" strokeDasharray="2,4"/>
            <text x="2" y={y+3} fontSize="8" fill={C.dim} fontFamily="'Space Mono',monospace">${(lp/1000).toFixed(1)}K</text>
          </g>:null;
        })}
        {zones.filter(z=>z.y>=mn&&z.y<=mx).map((z,i)=>{
          const y=toY(z.y),col=tCol(z.type),isP=z.type==="price";
          return <g key={i}>
            <rect x="52" y={y-2} width={z.str*200} height={isP?4:2} fill={col} opacity={isP?1:z.str*0.6} rx="1"/>
            {isP&&<line x1="52" y1={y} x2="330" y2={y} stroke={col} strokeWidth="1.2" strokeDasharray="4,2"/>}
            <text x="57" y={y-4} fontSize="8" fill={col} fontFamily="'Space Mono',monospace">{z.label}</text>
            <text x="326" y={y-4} fontSize="8" fill={col} fontFamily="'Space Mono',monospace" textAnchor="end">${(z.y/1000).toFixed(1)}K</text>
          </g>;
        })}
      </svg>:<div style={{height:270,display:"flex",alignItems:"center",justifyContent:"center",
        fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>Loading live data...</div>}
    </Box>
  </section>;
});

/* ─── SMART MONEY / ORDER FLOW ───────────────────────────────────────── */
const SmartMoney=memo(function SmartMoney({display,depth,of}){
  const obData=useMemo(()=>{
    if(!depth?.bids?.length) return [];
    const out=[];
    depth.bids.slice(0,8).forEach(([p,q])=>out.push({p:parseFloat(p).toFixed(0),bid:+parseFloat(q).toFixed(2),ask:0}));
    depth.asks.slice(0,8).forEach(([p,q],i)=>{
      if(out[i]) out[i].ask=+parseFloat(q).toFixed(2);
    });
    return out;
  },[depth]);
  const{fundRate,oi}=display;
  return <section>
    <SecTitle ch="Smart Money Flow" sub="Live 600ms depth · 1s funding"/>
    <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:7,marginBottom:9}}>
      <Box style={{padding:"10px 12px"}}>
        <Label ch="Book Imbalance"/>
        <div style={{display:"flex",gap:2,height:17,marginTop:5}}>
          <div style={{width:`${(of.bidRatio||0.5)*100}%`,background:C.green,borderRadius:"3px 0 0 3px",
            display:"flex",alignItems:"center",justifyContent:"center",minWidth:20,opacity:0.85}}>
            <span style={{fontSize:8,fontFamily:"'Space Mono',monospace",color:"#000",fontWeight:700,fontVariantNumeric:"tabular-nums"}}>
              {((of.bidRatio||0.5)*100).toFixed(0)}%
            </span>
          </div>
          <div style={{flex:1,background:C.red,borderRadius:"0 3px 3px 0",
            display:"flex",alignItems:"center",justifyContent:"center",minWidth:20,opacity:0.75}}>
            <span style={{fontSize:8,color:"#fff",fontFamily:"'Space Mono',monospace",fontWeight:700,fontVariantNumeric:"tabular-nums"}}>
              {((1-(of.bidRatio||0.5))*100).toFixed(0)}%
            </span>
          </div>
        </div>
        <div style={{display:"flex",justifyContent:"space-between",marginTop:3}}>
          <span style={{fontSize:8,color:C.green,fontFamily:"'Space Mono',monospace"}}>BID ${(of.bidDepth/1e6||0).toFixed(1)}M</span>
          <span style={{fontSize:8,color:C.red,fontFamily:"'Space Mono',monospace"}}>ASK ${(of.askDepth/1e6||0).toFixed(1)}M</span>
        </div>
      </Box>
      <Box style={{padding:"10px 12px"}}>
        <Label ch="Pressure"/>
        <Num v={of.pressure||"NEUTRAL"} col={of.pressure==="BUY_DOMINANT"?C.green:of.pressure==="SELL_DOMINANT"?C.red:C.gold} sz={10} w="12ch"/>
        <div style={{marginTop:4,fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>Bid wall: {of.bigBidWall?.toFixed(1)||"—"} BTC</div>
        <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>Ask wall: {of.bigAskWall?.toFixed(1)||"—"} BTC</div>
      </Box>
    </div>
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:6,marginBottom:9}}>
      {[["Funding",(fundRate>=0?"+":"")+fundRate.toFixed(4)+"%",fundRate>0.05?C.red:fundRate<-0.01?C.green:C.gold],
        ["Open Int","$"+oi.toFixed(1)+"B",C.cyan],
        ["Sentiment",fundRate>0.05?"Longs Crowded":fundRate<-0.01?"Shorts Crowded":"Balanced",C.text]
      ].map(([l,v,c])=><Box key={l} style={{padding:"8px 10px"}}><Label ch={l} style={{fontSize:8}}/><Num v={v} col={c} sz={11} w="9ch"/></Box>)}
    </div>
    {obData.length>0&&<Box style={{padding:"10px 12px"}}>
      <Label ch="Order Book Depth (top 8)" style={{marginBottom:5}}/>
      <div style={{height:110}}>
        <ResponsiveContainer width="100%" height="100%">
          <BarChart data={obData} layout="vertical" margin={{top:0,right:3,bottom:0,left:42}}>
            <XAxis type="number" hide/>
            <YAxis dataKey="p" type="category" tick={{fill:C.dim,fontSize:8,fontFamily:"'Space Mono',monospace"}}
              tickLine={false} axisLine={false} width={42}/>
            <Bar dataKey="bid" stackId="a" fill={`${C.green}60`} name="Bid" isAnimationActive={false}/>
            <Bar dataKey="ask" stackId="a" fill={`${C.red}60`} name="Ask" radius={[0,2,2,0]} isAnimationActive={false}/>
            <Tooltip content={<ChartTip/>}/>
          </BarChart>
        </ResponsiveContainer>
      </div>
    </Box>}
  </section>;
});

/* ─── SIMULATION PANEL ───────────────────────────────────────────────── */
const SimulationSection=memo(function SimulationSection({signals,kRef,candleVer}){
  const best=signals.snp||signals.int||signals.trd;
  const mc=best?.mc;
  const hv=useMemo(()=>{
    const cl=closedOnly(kRef.current.k1m)?.map(r=>parseFloat(r[4]));
    return cl?TA.hv(cl):0;
  },[candleVer]);
  return <section>
    <SecTitle ch="AI Simulation" sub={`${MC_PATHS} paths · real historical returns`}/>
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:6,marginBottom:9}}>
      {[["HV",((hv||0)*100).toFixed(2)+"%",C.cyan],
        ["MC TP",mc?(mc.probTP*100).toFixed(0)+"%":"—",C.green],
        ["MC SL",mc?(mc.probSL*100).toFixed(0)+"%":"—",C.red],
        ["EV $",mc?(mc.ev>0?"+":"")+Math.round(mc.ev):"—",mc?.ev>0?C.green:C.red],
        ["P50",mc?"$"+mc.p50?.toLocaleString():"—",C.gold],
        ["Paths",MC_PATHS,C.dim],
      ].map(([l,v,c])=><Box key={l} style={{padding:"8px 10px"}}><Label ch={l} style={{fontSize:8}}/><Num v={v} col={c} sz={12} w="7ch"/></Box>)}
    </div>
    {mc?.mcData?.length>0&&<Box style={{padding:"12px 13px"}}>
      <Label ch="P10 / P50 / P90 Simulation Bands" style={{marginBottom:5}}/>
      <div style={{height:110}}>
        <ResponsiveContainer width="100%" height="100%">
          <AreaChart data={mc.mcData} margin={{top:3,right:3,bottom:0,left:0}}>
            <defs>
              <linearGradient id="sg90" x1="0" y1="0" x2="0" y2="1"><stop offset="5%" stopColor={C.green} stopOpacity={0.18}/><stop offset="95%" stopColor={C.green} stopOpacity={0.01}/></linearGradient>
              <linearGradient id="sg10" x1="0" y1="0" x2="0" y2="1"><stop offset="5%" stopColor={C.red} stopOpacity={0.12}/><stop offset="95%" stopColor={C.red} stopOpacity={0.01}/></linearGradient>
            </defs>
            <XAxis hide/><YAxis domain={["auto","auto"]} hide/>
            <Area type="monotone" dataKey="p90" stroke={`${C.green}85`} strokeWidth={0.8} fill="url(#sg90)" dot={false} strokeDasharray="4,2" isAnimationActive={false}/>
            <Area type="monotone" dataKey="p50" stroke={C.gold} strokeWidth={1.5} fill="none" dot={false} isAnimationActive={false}/>
            <Area type="monotone" dataKey="p10" stroke={`${C.red}85`} strokeWidth={0.8} fill="url(#sg10)" dot={false} strokeDasharray="4,2" isAnimationActive={false}/>
            <Tooltip content={<ChartTip/>}/>
          </AreaChart>
        </ResponsiveContainer>
      </div>
    </Box>}
    {!best&&<Box style={{padding:"13px",textAlign:"center"}}>
      <div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>Simulation runs when signal is generated</div>
    </Box>}
  </section>;
});

/* ═══════════════════════════════════════════════════════════════════════
   PERFORMANCE ENGINE  — derived purely from closed trades in reducer
═══════════════════════════════════════════════════════════════════════ */
function derivePerformance(trades){
  const closed=trades.filter(t=>t.status==="CLOSED"&&t.closePrice&&t.entry);
  if(!closed.length) return null;
  let equity=0;
  const curve=closed.map(t=>{
    const pnl=(t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100;
    equity+=pnl;
    return{ts:t.closedAt,pnl:parseFloat(pnl.toFixed(3)),
           equity:parseFloat(equity.toFixed(3)),outcome:t.outcome,id:t.id};
  });
  const byCard={SNP:[],INT:[],TRD:[]};
  closed.forEach(t=>{
    const pnl=(t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100;
    if(byCard[t.id]) byCard[t.id].push({pnl,outcome:t.outcome,dur:t.closedAt-t.openedAt});
  });
  const cardStats=Object.entries(byCard).map(([id,arr])=>{
    if(!arr.length) return{id,wr:null,avg:0,count:0,avgDur:0,col:id==="SNP"?C.cyan:id==="INT"?C.gold:C.purple};
    const wins=arr.filter(t=>t.outcome!=="SL");
    return{id,wr:wins.length/arr.length,avg:arr.reduce((s,t)=>s+t.pnl,0)/arr.length,
      count:arr.length,avgDur:(arr.reduce((s,t)=>s+t.dur,0)/arr.length/60000).toFixed(0),
      col:id==="SNP"?C.cyan:id==="INT"?C.gold:C.purple};
  });
  let peak=0,maxDD=0;
  curve.forEach(p=>{peak=Math.max(peak,p.equity);maxDD=Math.max(maxDD,peak-p.equity);});
  const wins=closed.filter(t=>t.outcome!=="SL"),lossT=closed.filter(t=>t.outcome==="SL");
  const wr=wins.length/closed.length;
  const avgWin=wins.length?wins.reduce((s,t)=>{const p=(t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100;return s+p;},0)/wins.length:0;
  const avgLoss=lossT.length?Math.abs(lossT.reduce((s,t)=>{const p=(t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100;return s+p;},0)/lossT.length):1;
  const pf=(wr*avgWin)/((1-wr)*avgLoss)||0;
  const pnls=curve.map(c=>c.pnl);
  const mu=pnls.reduce((s,v)=>s+v,0)/pnls.length;
  const sd=pnls.length>1?Math.sqrt(pnls.reduce((s,v)=>s+(v-mu)**2,0)/(pnls.length-1)):1;
  const sharpe=sd>0?parseFloat((mu/sd*Math.sqrt(252)).toFixed(2)):0;
  const regimePerf={};
  closed.forEach(t=>{
    const r=t.regime||"UNKNOWN";
    if(!regimePerf[r]) regimePerf[r]={wins:0,total:0};
    regimePerf[r].total++;
    if(t.outcome!=="SL") regimePerf[r].wins++;
  });
  return{curve,cardStats,maxDD:maxDD.toFixed(2),wr,avgWin:avgWin.toFixed(2),
    avgLoss:avgLoss.toFixed(2),pf:pf.toFixed(2),sharpe,total:closed.length,
    totalPnL:equity.toFixed(2),regimePerf};
}

/* ═══════════════════════════════════════════════════════════════════════
   ALERT LOG HOOK  — ring buffer, 30 entries max, never re-renders analysis
═══════════════════════════════════════════════════════════════════════ */
function useAlertLog(){
  const[log,setLog]=useState([]);
  const add=useCallback((type,msg,col=C.cyan)=>{
    setLog(l=>[{ts:Date.now(),type,msg,col},...l].slice(0,30));
  },[]);
  return[log,add];
}

/* ═══════════════════════════════════════════════════════════════════════
   PERFORMANCE ANALYTICS COMPONENT
═══════════════════════════════════════════════════════════════════════ */
const PerformanceAnalytics=memo(function PerformanceAnalytics({trades}){
  const perf=useMemo(()=>derivePerformance(trades),[trades.length,
    trades.filter(t=>t.status==="CLOSED").length]);
  if(!perf) return <section>
    <SecTitle ch="Performance Analytics" sub="Accumulates as TMA-approved trades close"/>
    <Box style={{padding:"13px",textAlign:"center"}}>
      <div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
        No closed trades yet — performance tracked automatically
      </div>
    </Box>
  </section>;
  const{curve,cardStats,maxDD,wr,avgWin,avgLoss,pf,sharpe,total,totalPnL,regimePerf}=perf;
  const pnlPos=parseFloat(totalPnL)>=0;
  return <section>
    <SecTitle ch="Performance Analytics" sub="Live from closed trades · equity curve · per-card stats"/>
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7,marginBottom:9}}>
      {[["Win Rate",(wr*100).toFixed(0)+"%",wr>0.6?C.green:wr>0.4?C.gold:C.red],
        ["Profit Factor",pf,parseFloat(pf)>1.5?C.green:parseFloat(pf)>1?C.gold:C.red],
        ["Sharpe Est.",sharpe,parseFloat(sharpe)>1.5?C.green:parseFloat(sharpe)>0?C.gold:C.red],
        ["Total P&L",(pnlPos?"+":"")+totalPnL+"%",pnlPos?C.green:C.red],
        ["Max Drawdown",maxDD+"%",parseFloat(maxDD)<3?C.green:parseFloat(maxDD)<8?C.gold:C.red],
        ["Closed Trades",total,C.text],
      ].map(([l,v,c])=><Box key={l} style={{padding:"9px 11px"}}>
        <Label ch={l} style={{fontSize:8}}/><Num v={v} col={c} sz={13} w="6ch"/>
      </Box>)}
    </div>
    {curve.length>1&&<Box style={{padding:"12px 13px",marginBottom:9}}>
      <Label ch="Equity Curve (% P&L · closed trades)" style={{marginBottom:5}}/>
      <div style={{height:105}}>
        <ResponsiveContainer width="100%" height="100%">
          <AreaChart data={curve} margin={{top:3,right:3,bottom:0,left:0}}>
            <defs><linearGradient id="eqG" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor={pnlPos?C.green:C.red} stopOpacity={0.25}/>
              <stop offset="95%" stopColor={pnlPos?C.green:C.red} stopOpacity={0.02}/>
            </linearGradient></defs>
            <XAxis hide/>
            <YAxis domain={["auto","auto"]} tick={{fill:C.dim,fontSize:8,fontFamily:"'Space Mono',monospace"}}
              tickLine={false} axisLine={false} tickFormatter={v=>v.toFixed(1)+"%"} width={40}/>
            <Tooltip content={<ChartTip/>}/>
            <ReferenceLine y={0} stroke={C.border} strokeDasharray="3 3"/>
            <Area type="monotone" dataKey="equity" name="Equity %" stroke={pnlPos?C.green:C.red}
              strokeWidth={1.5} fill="url(#eqG)" dot={false} isAnimationActive={false}/>
          </AreaChart>
        </ResponsiveContainer>
      </div>
    </Box>}
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7,marginBottom:9}}>
      {cardStats.map(s=><Box key={s.id} style={{padding:"9px 11px",border:`1px solid ${s.col}25`}}>
        <Label ch={s.id} style={{fontSize:8,color:s.col}}/>
        <Num v={s.count?(s.wr*100).toFixed(0)+"%":"—"}
          col={s.wr>0.6?C.green:s.wr>0.4?C.gold:s.wr>0?C.red:C.dim} sz={15} w="4ch"/>
        <div style={{height:2,background:C.muted,borderRadius:1,margin:"4px 0 3px"}}>
          <div style={{width:`${(s.wr||0)*100}%`,height:"100%",background:s.col,borderRadius:1}}/>
        </div>
        <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
          {s.count} trades · {s.avgDur}m avg
        </div>
      </Box>)}
    </div>
    <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:7,marginBottom:9}}>
      <Box style={{padding:"9px 11px"}}><Label ch="Avg Win"/><Num v={"+"+avgWin+"%"} col={C.green} sz={14} w="6ch"/></Box>
      <Box style={{padding:"9px 11px"}}><Label ch="Avg Loss"/><Num v={"-"+avgLoss+"%"} col={C.red} sz={14} w="6ch"/></Box>
    </div>
    {Object.keys(regimePerf).length>0&&<Box style={{padding:"11px 13px"}}>
      <Label ch="Win Rate by Market Regime" style={{marginBottom:7}}/>
      {Object.entries(regimePerf).map(([r,p])=>{
        const rwr=p.total?p.wins/p.total:0;
        return <div key={r} style={{display:"flex",alignItems:"center",gap:7,padding:"5px 0",
          borderBottom:`1px solid ${C.border}`}}>
          <div style={{flex:1,fontSize:8,color:C.text,fontFamily:"'Space Mono',monospace"}}>{r.replace(/_/g," ")}</div>
          <div style={{height:3,width:60,background:C.muted,borderRadius:2,flexShrink:0}}>
            <div style={{width:`${rwr*100}%`,height:"100%",
              background:rwr>0.6?C.green:rwr>0.4?C.gold:C.red,borderRadius:2}}/>
          </div>
          <div style={{fontSize:9,fontFamily:"'Space Mono',monospace",fontWeight:700,
            color:rwr>0.6?C.green:rwr>0.4?C.gold:C.red,minWidth:28,textAlign:"right",
            fontVariantNumeric:"tabular-nums"}}>
            {(rwr*100).toFixed(0)}%
          </div>
          <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",minWidth:28}}>
            {p.wins}/{p.total}
          </div>
        </div>;
      })}
    </Box>}
  </section>;
});

/* ═══════════════════════════════════════════════════════════════════════
   MULTI-TIMEFRAME RSI PANEL
═══════════════════════════════════════════════════════════════════════ */
const MultiTFRSI=memo(function MultiTFRSI({kRef,candleVer}){
  const tfs=useMemo(()=>{
    const compute=(k,label)=>{
      const kc=closedOnly(k);
      if(!kc||kc.length<20) return{label,rsi:null,col:C.dim,signal:"—",num:50};
      const cl=kc.map(r=>parseFloat(r[4]));
      const arr=TA.rsi(cl,14); const rsi=arr[arr.length-1];
      const col=rsi>70?C.red:rsi>60?C.gold:rsi<30?C.green:rsi<40?C.cyan:C.text;
      const signal=rsi>70?"OVERBOUGHT":rsi>60?"BULLISH":rsi<30?"OVERSOLD":rsi<40?"BEARISH":"NEUTRAL";
      return{label,rsi:rsi.toFixed(1),col,signal,num:rsi};
    };
    return[
      compute(kRef.current.k1m,"1m"),
      compute(kRef.current.k15m,"15m"),
      compute(kRef.current.k1h,"1h"),
      compute(kRef.current.k4h,"4h"),
    ];
  },[candleVer]);
  const nums=tfs.map(t=>t.num).filter(Boolean);
  const allBull=nums.length>0&&nums.every(n=>n>50);
  const allBear=nums.length>0&&nums.every(n=>n<50);
  const alignment=allBull?"BULLISH STACK":allBear?"BEARISH STACK":
    nums.filter(n=>n>50).length>=3?"LEANING BULL":
    nums.filter(n=>n<50).length>=3?"LEANING BEAR":"MIXED";
  const alignCol=allBull?C.green:allBear?C.red:
    alignment==="LEANING BULL"?C.gold:alignment==="LEANING BEAR"?C.orange:C.dim;
  return <section>
    <SecTitle ch="Multi-TF RSI Alignment" sub="1m · 15m · 1h · 4h · closed candles only"/>
    <Box style={{padding:"12px 13px",marginBottom:9}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:10}}>
        <Label ch="Alignment"/>
        <Badge ch={alignment} col={alignCol}/>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:7}}>
        {tfs.map(t=><div key={t.label} style={{textAlign:"center",padding:"9px 6px",
          background:t.rsi?`${t.col}0C`:C.muted+"20",borderRadius:7,
          border:`1px solid ${t.rsi?t.col+"28":C.border}`}}>
          <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",marginBottom:4}}>{t.label}</div>
          <Num v={t.rsi||"—"} col={t.col} sz={17} w="4ch"/>
          <div style={{height:3,background:C.muted,borderRadius:2,margin:"5px 0 4px",overflow:"hidden"}}>
            {t.rsi&&<div style={{width:`${t.num}%`,height:"100%",background:t.col,borderRadius:2}}/>}
          </div>
          <div style={{fontSize:7,color:t.col,fontFamily:"'Space Mono',monospace"}}>{t.signal}</div>
        </div>)}
      </div>
    </Box>
  </section>;
});

/* ═══════════════════════════════════════════════════════════════════════
   PREDICTED MOVE — ATR bands + regime bias + HV
═══════════════════════════════════════════════════════════════════════ */
const PredictedMove=memo(function PredictedMove({kRef,candleVer,price,regime}){
  const pred=useMemo(()=>{
    const k4=closedOnly(kRef.current.k4h);
    const k1=closedOnly(kRef.current.k1m);
    if(!k4||!price) return null;
    const hi4=k4.map(r=>parseFloat(r[2])),lo4=k4.map(r=>parseFloat(r[3])),cl4=k4.map(r=>parseFloat(r[4]));
    const atrArr=TA.atr(hi4,lo4,cl4,14); const atr4h=atrArr[atrArr.length-1];
    const atrD=atr4h*2.5; // approx daily
    let hv=0;
    if(k1&&k1.length>22){const cl1=k1.map(r=>parseFloat(r[4]));hv=TA.hv(cl1,20);}
    const hvMove=price*(hv/Math.sqrt(365));
    const biasPct=regime.dir==="BULLISH"?0.58:regime.dir==="BEARISH"?0.42:0.50;
    const bands=[
      {label:"Next 4h",up:Math.round(price+atr4h),dn:Math.round(price-atr4h),col:C.cyan,sz:"1× ATR4h"},
      {label:"Next 8h",up:Math.round(price+atr4h*1.8),dn:Math.round(price-atr4h*1.8),col:C.purple,sz:"1.8× ATR4h"},
      {label:"Daily",up:Math.round(price+atrD),dn:Math.round(price-atrD),col:C.gold,sz:"2.5× ATR4h"},
    ];
    return{atr4h,atrD,hvMove,biasPct,bands};
  },[candleVer,price,regime.dir]);
  if(!pred) return null;
  const{atr4h,atrD,hvMove,biasPct,bands}=pred;
  return <section>
    <SecTitle ch="Predicted Move" sub="ATR range bands · HV · regime directional bias"/>
    <Box style={{padding:"12px 14px"}}>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:7,marginBottom:10}}>
        <div style={{textAlign:"center",padding:"9px",background:`${C.green}08`,borderRadius:6,border:`1px solid ${C.green}1F`}}>
          <Label ch="Bull Bias" style={{textAlign:"center",color:C.green}}/>
          <Num v={(biasPct*100).toFixed(0)+"%"} col={C.green} sz={20} w="4ch"/>
        </div>
        <div style={{textAlign:"center",padding:"9px",background:`${C.red}08`,borderRadius:6,border:`1px solid ${C.red}1F`}}>
          <Label ch="Bear Bias" style={{textAlign:"center",color:C.red}}/>
          <Num v={((1-biasPct)*100).toFixed(0)+"%"} col={C.red} sz={20} w="4ch"/>
        </div>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7,marginBottom:10}}>
        {[["4h ATR","$"+Math.round(atr4h).toLocaleString(),C.gold],
          ["Daily ATR","$"+Math.round(atrD).toLocaleString(),C.gold],
          ["HV Move","$"+Math.round(hvMove).toLocaleString(),C.cyan],
        ].map(([l,v,c])=><Box key={l} style={{padding:"8px 10px"}}><Label ch={l} style={{fontSize:8}}/><Num v={v} col={c} sz={12} w="7ch"/></Box>)}
      </div>
      <Label ch="Expected Range Bands" style={{marginBottom:8}}/>
      {bands.map(b=><div key={b.label} style={{marginBottom:8}}>
        <div style={{display:"flex",justifyContent:"space-between",marginBottom:3}}>
          <div style={{display:"flex",alignItems:"center",gap:5}}>
            <div style={{width:6,height:6,borderRadius:1,background:b.col,flexShrink:0}}/>
            <span style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{b.label} ({b.sz})</span>
          </div>
        </div>
        <div style={{display:"flex",alignItems:"center",gap:7}}>
          <div style={{textAlign:"right",minWidth:65,flexShrink:0}}>
            <div style={{fontSize:7,color:C.dim,fontFamily:"'Space Mono',monospace"}}>Support</div>
            <div style={{fontSize:10,fontFamily:"'Space Mono',monospace",fontWeight:700,
              color:C.red,fontVariantNumeric:"tabular-nums"}}>${b.dn.toLocaleString()}</div>
          </div>
          <div style={{flex:1,height:8,background:C.muted,borderRadius:4,position:"relative",overflow:"hidden"}}>
            <div style={{position:"absolute",inset:0,
              background:`linear-gradient(90deg,${C.red}55,${b.col}30,${C.green}55)`,borderRadius:4}}/>
            <div style={{position:"absolute",top:0,bottom:0,width:2,background:C.bright,opacity:0.7,
              left:`${biasPct*100}%`,transform:"translateX(-50%)"}}/>
          </div>
          <div style={{minWidth:65,flexShrink:0}}>
            <div style={{fontSize:7,color:C.dim,fontFamily:"'Space Mono',monospace"}}>Resistance</div>
            <div style={{fontSize:10,fontFamily:"'Space Mono',monospace",fontWeight:700,
              color:C.green,fontVariantNumeric:"tabular-nums"}}>${b.up.toLocaleString()}</div>
          </div>
        </div>
      </div>)}
    </Box>
  </section>;
});

/* ═══════════════════════════════════════════════════════════════════════
   AI STRATEGY INSIGHTS  — pattern analysis, suggestions, improvement
═══════════════════════════════════════════════════════════════════════ */
const AIStrategyInsights=memo(function AIStrategyInsights({trades,regime,signals,tma,kRef,candleVer}){
  const insights=useMemo(()=>{
    // MTF confirmation check
    const mtfSame=signals.snp&&signals.int&&signals.trd&&
      signals.snp.dir===signals.int.dir&&signals.int.dir===signals.trd.dir;
    // Suggestions based on live conditions
    const suggestions=[];
    if(tma?.sessionStats?.streak>=2)
      suggestions.push("2+ consecutive losses — apply 0.5× sizing until next win");
    if(tma?.sessionStats?.wr!==null&&tma.sessionStats.wr<0.55&&tma.sessionStats.total>=5)
      suggestions.push("Win rate below 55% — raise minimum confidence to 80% temporarily");
    if(signals.snp&&signals.int&&signals.snp.dir!==signals.int?.dir)
      suggestions.push("SNP vs INT direction conflict — skip scalps, wait for alignment");
    if(mtfSame&&signals.snp)
      suggestions.push("All three cards aligned "+signals.snp.dir+" — highest-conviction setup, consider full size");
    if(regime.regime==="RANGE_BOUND")
      suggestions.push("Range market: trade SNP at extremes only, avoid TRD entries");
    if(regime.regime==="COMPRESSION")
      suggestions.push("Compression phase: mark range boundaries, set breakout alerts, no mean-reversion scalps");
    if(regime.regime==="HIGH_VOLATILITY")
      suggestions.push("High volatility: widen stops by 1.5×, reduce size by 30%, avoid FOMO entries");
    if(regime.rsi&&regime.rsi>68)
      suggestions.push(`RSI ${regime.rsi.toFixed(0)} — overbought on 1m, avoid new longs without pullback`);
    if(regime.rsi&&regime.rsi<32)
      suggestions.push(`RSI ${regime.rsi.toFixed(0)} — oversold on 1m, avoid new shorts without bounce`);
    if(regime.bbWidth&&regime.bbWidth<0.005)
      suggestions.push("Bollinger Bands extremely tight — breakout imminent, prepare limit orders at boundaries");
    if(!suggestions.length)
      suggestions.push("System operating within normal parameters — maintain standard discipline and sizing");
    return{suggestions,mtfSame};
  },[trades.length,regime.regime,regime.rsi,signals.snp?.dir,signals.int?.dir,signals.trd?.dir,tma?.sessionStats]);

  const closed=trades.filter(t=>t.status==="CLOSED");
  const byCard={SNP:0,INT:0,TRD:0};
  closed.filter(t=>t.outcome!=="SL").forEach(t=>byCard[t.id]=(byCard[t.id]||0)+1);
  const bestCard=Object.entries(byCard).sort((a,b)=>b[1]-a[1])[0];

  return <section>
    <SecTitle ch="AI Strategy Insights" sub="Pattern recognition · condition analysis · improvement feed"/>
    <Box style={{padding:"12px 14px",marginBottom:9}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:8}}>
        <Label ch="Improvement Suggestions"/>
        <Badge ch={insights.mtfSame?"ALIGNED":"MIXED"} col={insights.mtfSame?C.green:C.gold} sm/>
      </div>
      {insights.suggestions.map((s,i)=><div key={i} style={{display:"flex",gap:7,
        alignItems:"flex-start",padding:"7px 9px",marginBottom:4,borderRadius:5,
        background:i===0?`${C.cyan}09`:C.muted+"18",
        border:`1px solid ${i===0?`${C.cyan}28`:C.border}`}}>
        <span style={{fontSize:10,color:i===0?C.cyan:C.dim,flexShrink:0,marginTop:0}}>→</span>
        <span style={{fontSize:9,color:C.text,fontFamily:"'Space Mono',monospace",lineHeight:1.5}}>{s}</span>
      </div>)}
    </Box>
    {bestCard&&bestCard[1]>0&&closed.length>0&&<Box style={{padding:"11px 13px",marginBottom:9}}>
      <Label ch="Session Best Performer"/>
      <div style={{display:"flex",alignItems:"center",gap:8,marginTop:5}}>
        <Badge ch={bestCard[0]} col={bestCard[0]==="SNP"?C.cyan:bestCard[0]==="INT"?C.gold:C.purple}/>
        <Num v={bestCard[1]+" wins"} col={C.green} sz={13} w="5ch"/>
        <span style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>this session</span>
      </div>
    </Box>}
    {closed.length>0&&<Box style={{padding:"11px 13px"}}>
      <Label ch="Last 5 Closed Trades" style={{marginBottom:7}}/>
      {closed.slice(-5).reverse().map((t,i)=>{
        const pnl=(t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100;
        const win=t.outcome!=="SL";
        return <div key={i} style={{display:"flex",alignItems:"center",gap:5,
          padding:"5px 0",borderBottom:i<4?`1px solid ${C.border}`:"none"}}>
          <Badge ch={t.id} col={t.id==="SNP"?C.cyan:t.id==="INT"?C.gold:C.purple} sm/>
          <Badge ch={t.dir} col={t.dir==="LONG"?C.green:C.red} sm/>
          <div style={{flex:1,fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",
            fontVariantNumeric:"tabular-nums"}}>
            ${t.entry?.toLocaleString()} → ${t.closePrice?.toLocaleString()}
          </div>
          <Badge ch={t.outcome||"?"} col={win?C.green:C.red} sm/>
          <div style={{fontSize:9,fontFamily:"'Space Mono',monospace",fontWeight:700,
            color:pnl>=0?C.green:C.red,fontVariantNumeric:"tabular-nums",minWidth:42,textAlign:"right"}}>
            {(pnl>=0?"+":"")+pnl.toFixed(2)+"%"}
          </div>
        </div>;
      })}
    </Box>}
  </section>;
});

/* ═══════════════════════════════════════════════════════════════════════
   ALERT LOG  — timestamped ring buffer of all key platform events
═══════════════════════════════════════════════════════════════════════ */
const AlertLog=memo(function AlertLog({log}){
  return <section>
    <SecTitle ch="Alert Log" sub="TMA gates · signals · trade events · circuit breaker"/>
    {log.length===0?<Box style={{padding:"12px",textAlign:"center"}}>
      <div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
        Alert log empty — events appear as they occur in real time
      </div>
    </Box>:<Box style={{padding:"11px 13px"}}>
      <div style={{maxHeight:280,overflowY:"auto",paddingRight:4}}>
        {log.map((l,i)=><div key={i} style={{display:"flex",gap:7,alignItems:"flex-start",
          padding:"5px 0",borderBottom:i<log.length-1?`1px solid ${C.border}`:"none"}}>
          <div style={{flexShrink:0,paddingTop:1}}>
            <div style={{width:5,height:5,borderRadius:"50%",background:l.col,marginTop:2}}/>
          </div>
          <div style={{minWidth:42,flexShrink:0}}>
            <div style={{fontSize:7,color:C.dim,fontFamily:"'Space Mono',monospace",lineHeight:1.3}}>
              {new Date(l.ts).toLocaleTimeString("en",{hour:"2-digit",minute:"2-digit",second:"2-digit"})}
            </div>
          </div>
          <div style={{flex:1,display:"flex",gap:5,flexWrap:"wrap",alignItems:"flex-start"}}>
            <Badge ch={l.type} col={l.col} sm/>
            <span style={{fontSize:8,color:C.text,fontFamily:"'Space Mono',monospace",
              lineHeight:1.5,flex:1}}>{l.msg}</span>
          </div>
        </div>)}
      </div>
    </Box>}
  </section>;
});

/* ═══════════════════════════════════════════════════════════════════════
   SETTINGS PANEL
═══════════════════════════════════════════════════════════════════════ */
function SettingsPanel({settings,onSave,onClose}){
  const[draft,setDraft]=useState({...settings});
  const set=(k,v)=>setDraft(d=>({...d,[k]:v}));
  const num=(k,v,mn,mx,step=1)=>set(k,Math.min(mx,Math.max(mn,parseFloat(v)||mn)));
  return <div style={{position:"fixed",inset:0,zIndex:300,background:"rgba(2,8,18,0.95)",
    display:"flex",alignItems:"flex-end",justifyContent:"center"}}>
    <div style={{width:"100%",maxWidth:530,background:C.card,border:`1px solid ${C.borderHi}`,
      borderRadius:"14px 14px 0 0",padding:"20px 16px 32px",maxHeight:"85vh",overflowY:"auto"}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:16}}>
        <span style={{fontFamily:"'Rajdhani',sans-serif",fontWeight:700,fontSize:15,
          color:C.cyan,letterSpacing:"0.12em"}}>SETTINGS</span>
        <button onClick={onClose} style={{background:"none",border:"none",
          color:C.dim,fontSize:18,cursor:"pointer",padding:"2px 6px"}}>✕</button>
      </div>

      {/* Risk & Equity */}
      <div style={{marginBottom:14}}>
        <div style={{fontSize:9,color:C.cyan,fontFamily:"'Space Mono',monospace",
          letterSpacing:"0.12em",marginBottom:8}}>RISK MANAGEMENT</div>
        {[["Equity ($)",draft.equity,v=>num("equity",v,100,1000000,100),100,1000000],
          ["Risk per Trade (%)",draft.riskPct,v=>num("riskPct",v,0.1,5,0.1),0.1,5],
        ].map(([l,val,fn,mn,mx])=><div key={l} style={{marginBottom:8}}>
          <div style={{display:"flex",justifyContent:"space-between",marginBottom:3}}>
            <span style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{l}</span>
            <span style={{fontSize:10,fontFamily:"'Space Mono',monospace",fontWeight:700,color:C.bright,
              fontVariantNumeric:"tabular-nums"}}>{val}</span>
          </div>
          <input type="range" min={mn} max={mx} step={l.includes("%")?0.1:100}
            value={val} onChange={e=>fn(e.target.value)}
            style={{width:"100%",accentColor:C.cyan}}/>
        </div>)}
      </div>

      {/* Signal Thresholds */}
      <div style={{marginBottom:14}}>
        <div style={{fontSize:9,color:C.cyan,fontFamily:"'Space Mono',monospace",
          letterSpacing:"0.12em",marginBottom:8}}>SIGNAL GATES</div>
        {[["Min Confidence (%)",(draft.confThresh*100).toFixed(0),v=>set("confThresh",v/100),60,99,1],
          ["Min R:R",draft.rrMin,v=>num("rrMin",v,1,5,0.1),1,5],
          ["Chandelier ATR ×",draft.chandelierM,v=>num("chandelierM",v,1.5,5,0.5),1.5,5],
          ["MC Paths",draft.mcPaths,v=>num("mcPaths",v,100,1000,50),100,1000],
        ].map(([l,val,fn,mn,mx,step])=><div key={l} style={{marginBottom:8}}>
          <div style={{display:"flex",justifyContent:"space-between",marginBottom:3}}>
            <span style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{l}</span>
            <span style={{fontSize:10,fontFamily:"'Space Mono',monospace",fontWeight:700,color:C.bright,
              fontVariantNumeric:"tabular-nums"}}>{val}</span>
          </div>
          <input type="range" min={mn} max={mx} step={step||1}
            value={parseFloat(val)} onChange={e=>fn(parseFloat(e.target.value))}
            style={{width:"100%",accentColor:C.gold}}/>
        </div>)}
      </div>

      {/* Toggles */}
      <div style={{marginBottom:14}}>
        <div style={{fontSize:9,color:C.cyan,fontFamily:"'Space Mono',monospace",
          letterSpacing:"0.12em",marginBottom:8}}>NOTIFICATIONS & MODE</div>
        {[["Telegram Notifications","tgEnabled",C.cyan],
          ["Notify on Signals","tgSignals",C.cyan],
          ["Notify on Trades","tgTrades",C.green],
          ["Notify on TMA Alerts","tgAlerts",C.orange],
          ["Paper Trading Mode","paperMode",C.gold],
        ].map(([l,k,col])=><div key={k} style={{display:"flex",justifyContent:"space-between",
          alignItems:"center",padding:"7px 0",borderBottom:`1px solid ${C.border}`}}>
          <span style={{fontSize:10,color:C.text,fontFamily:"'Space Mono',monospace"}}>{l}</span>
          <div onClick={()=>set(k,!draft[k])} style={{width:36,height:20,borderRadius:10,
            background:draft[k]?col:C.muted,cursor:"pointer",position:"relative",transition:"background 0.2s"}}>
            <div style={{position:"absolute",top:2,width:16,height:16,borderRadius:8,
              background:C.bright,transition:"left 0.2s",left:draft[k]?18:2,
              boxShadow:"0 1px 3px rgba(0,0,0,0.5)"}}/>
          </div>
        </div>)}
      </div>

      {/* Telegram status */}
      <div style={{padding:"8px 11px",background:`${C.cyan}09`,borderRadius:6,
        border:`1px solid ${C.cyan}20`,marginBottom:14}}>
        <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",marginBottom:2}}>Telegram</div>
        <div style={{fontSize:9,color:C.text,fontFamily:"'Space Mono',monospace"}}>
          Bot: <span style={{color:C.cyan}}>@btcplatform_bot</span> · Chat ID: <span style={{color:C.cyan}}>{TG_CHAT}</span>
        </div>
        <div style={{fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",marginTop:2}}>Notifications delivered via Telegram Bot API</div>
      </div>

      <div style={{display:"flex",gap:8}}>
        <button onClick={()=>{saveSettings(draft);onSave(draft);onClose();}}
          style={{flex:2,padding:"10px 0",borderRadius:7,fontSize:10,fontWeight:700,
            fontFamily:"'Space Mono',monospace",background:C.cyan,color:"#000",
            border:"none",cursor:"pointer",letterSpacing:"0.1em"}}>SAVE &amp; APPLY</button>
        <button onClick={()=>{saveSettings(DEFAULT_SETTINGS);onSave(DEFAULT_SETTINGS);}}
          style={{flex:1,padding:"10px 0",borderRadius:7,fontSize:10,fontWeight:700,
            fontFamily:"'Space Mono',monospace",background:"none",color:C.dim,
            border:`1px solid ${C.border}`,cursor:"pointer"}}>RESET</button>
      </div>
    </div>
  </div>;
}

/* ═══════════════════════════════════════════════════════════════════════
   PAPER TRADING DASHBOARD
═══════════════════════════════════════════════════════════════════════ */
const PaperTradingPanel=memo(function PaperTradingPanel({paper,paperDispatch,settings,price}){
  const{equity,startEquity,trades,maxDD,peak}=paper;
  const openT=trades.filter(t=>t.status==="OPEN");
  const closedT=trades.filter(t=>t.status==="CLOSED"&&t.closePrice);
  const wins=closedT.filter(t=>t.outcome==="TP"||t.outcome==="TRAIL_TP");
  const wr=closedT.length?wins.length/closedT.length:null;
  const totalPnl=equity-startEquity;
  const pnlPct=startEquity?totalPnl/startEquity*100:0;
  const curve=closedT.map((t,i)=>({i,eq:parseFloat(t.pnl||0)}));
  let running=startEquity;
  const eqCurve=closedT.map(t=>{running+=t.pnl||0;return{eq:parseFloat(running.toFixed(2))};});
  return <section>
    <SecTitle ch="Paper Trading" sub={`Virtual $${startEquity.toLocaleString()} · ${settings.riskPct}% risk · real market prices`}/>
    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7,marginBottom:9}}>
      {[["Balance","$"+equity.toLocaleString(),pnlPct>=0?C.green:C.red],
        ["P&L",(pnlPct>=0?"+":"")+pnlPct.toFixed(2)+"%",pnlPct>=0?C.green:C.red],
        ["Max DD","$"+parseFloat(maxDD).toFixed(0),parseFloat(maxDD)>startEquity*0.1?C.red:C.gold],
        ["Win Rate",wr!==null?(wr*100).toFixed(0)+"%":"—",wr>0.6?C.green:wr>0.4?C.gold:wr>0?C.red:C.dim],
        ["Trades",closedT.length,C.text],["Open",openT.length,C.cyan],
      ].map(([l,v,c])=><Box key={l} style={{padding:"9px 11px"}}>
        <Label ch={l} style={{fontSize:8}}/><Num v={v} col={c} sz={13} w="7ch"/>
      </Box>)}
    </div>
    {eqCurve.length>1&&<Box style={{padding:"11px 13px",marginBottom:9}}>
      <Label ch="Paper Equity Curve" style={{marginBottom:5}}/>
      <div style={{height:90}}>
        <ResponsiveContainer width="100%" height="100%">
          <AreaChart data={eqCurve} margin={{top:3,right:3,bottom:0,left:0}}>
            <defs><linearGradient id="pEqG" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor={pnlPct>=0?C.green:C.red} stopOpacity={0.25}/>
              <stop offset="95%" stopColor={pnlPct>=0?C.green:C.red} stopOpacity={0.01}/>
            </linearGradient></defs>
            <XAxis hide/><YAxis domain={["auto","auto"]} tick={{fill:C.dim,fontSize:8,fontFamily:"'Space Mono',monospace"}} tickLine={false} axisLine={false} tickFormatter={v=>"$"+v.toLocaleString()} width={52}/>
            <ReferenceLine y={startEquity} stroke={C.border} strokeDasharray="3 3"/>
            <Tooltip content={<ChartTip/>}/>
            <Area type="monotone" dataKey="eq" name="Balance $" stroke={pnlPct>=0?C.green:C.red}
              strokeWidth={1.5} fill="url(#pEqG)" dot={false} isAnimationActive={false}/>
          </AreaChart>
        </ResponsiveContainer>
      </div>
    </Box>}
    {openT.length>0&&<Box style={{padding:"11px 13px",marginBottom:9}}>
      <Label ch="Open Paper Positions" style={{marginBottom:6}}/>
      {openT.map(t=>{
        const live=t.pnl||0;
        const livePct=startEquity?live/startEquity*100:0;
        return <div key={t.id} style={{display:"flex",alignItems:"center",gap:7,
          padding:"6px 0",borderBottom:`1px solid ${C.border}`}}>
          <Badge ch={t.id} col={t.id==="SNP"?C.cyan:t.id==="INT"?C.gold:C.purple} sm/>
          <Badge ch={t.dir} col={t.dir==="LONG"?C.green:C.red} sm/>
          <div style={{flex:1,fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",
            fontVariantNumeric:"tabular-nums"}}>
            ${t.entry?.toLocaleString()} · pos {t.posSize?.toFixed(4)} BTC
          </div>
          <div style={{fontSize:10,fontFamily:"'Space Mono',monospace",fontWeight:700,
            color:live>=0?C.green:C.red,fontVariantNumeric:"tabular-nums"}}>
            {live>=0?"+":""}{livePct.toFixed(2)}%
          </div>
        </div>;
      })}
    </Box>}
    {closedT.length>0&&<Box style={{padding:"11px 13px",marginBottom:9}}>
      <Label ch="Recent Paper Trades" style={{marginBottom:6}}/>
      {closedT.slice(-5).reverse().map((t,i)=><div key={i} style={{display:"flex",
        alignItems:"center",gap:5,padding:"5px 0",borderBottom:i<4?`1px solid ${C.border}`:"none"}}>
        <Badge ch={t.id} col={t.id==="SNP"?C.cyan:t.id==="INT"?C.gold:C.purple} sm/>
        <Badge ch={t.dir} col={t.dir==="LONG"?C.green:C.red} sm/>
        <div style={{flex:1,fontSize:8,color:C.dim,fontFamily:"'Space Mono',monospace",
          fontVariantNumeric:"tabular-nums"}}>
          ${t.entry?.toLocaleString()} → ${t.closePrice?.toLocaleString()}
        </div>
        <Badge ch={t.outcome} col={t.outcome==="SL"?C.red:C.green} sm/>
        <div style={{fontSize:9,fontFamily:"'Space Mono',monospace",fontWeight:700,
          color:t.pnl>=0?C.green:C.red,minWidth:42,textAlign:"right",
          fontVariantNumeric:"tabular-nums"}}>
          {t.pnl>=0?"+":""}${t.pnl?.toFixed(0)}
        </div>
      </div>)}
    </Box>}
    <div style={{display:"flex",gap:7}}>
      <button onClick={()=>paperDispatch({type:"CLOSE_ALL",price})}
        style={{flex:1,padding:"9px 0",borderRadius:6,fontSize:9,fontWeight:700,
          fontFamily:"'Space Mono',monospace",color:C.orange,
          background:`${C.orange}12`,border:`1px solid ${C.orange}30`,cursor:"pointer"}}>
        CLOSE ALL POSITIONS
      </button>
      <button onClick={()=>paperDispatch({type:"RESET",equity:settings.equity})}
        style={{flex:1,padding:"9px 0",borderRadius:6,fontSize:9,fontWeight:700,
          fontFamily:"'Space Mono',monospace",color:C.dim,
          background:"none",border:`1px solid ${C.border}`,cursor:"pointer"}}>
        RESET ($10K)
      </button>
    </div>
  </section>;
});

/* ═══════════════════════════════════════════════════════════════════════
   BACKTESTER
═══════════════════════════════════════════════════════════════════════ */
const BacktesterPanel=memo(function BacktesterPanel({kRef,settings}){
  const[running,setRunning]=useState(false);
  const[result,setResult]=useState(null);
  const[tf,setTf]=useState("15m");
  const run=useCallback(async()=>{
    setRunning(true);setResult(null);
    await new Promise(r=>setTimeout(r,30)); // yield to UI
    const kmap={"1m":kRef.current.k1m,"15m":kRef.current.k15m,"1h":kRef.current.k1h,"4h":kRef.current.k4h};
    const r=runBacktest(kmap[tf]||kRef.current.k15m,tf,settings.equity,settings.riskPct);
    setResult(r);setRunning(false);
  },[tf,settings,kRef]);
  return <section>
    <SecTitle ch="Backtester" sub="Walk-forward · no lookahead · closed bars only"/>
    <Box style={{padding:"12px 14px",marginBottom:9}}>
      <div style={{display:"flex",gap:7,marginBottom:10,alignItems:"center"}}>
        <Label ch="Timeframe" style={{marginBottom:0,flexShrink:0}}/>
        <div style={{display:"flex",gap:5,flex:1}}>
          {["1m","15m","1h","4h"].map(t=><button key={t} onClick={()=>setTf(t)}
            style={{flex:1,padding:"5px 0",borderRadius:5,fontSize:9,fontWeight:700,
              fontFamily:"'Space Mono',monospace",cursor:"pointer",
              color:tf===t?C.bg:C.dim,background:tf===t?C.cyan:"none",
              border:`1px solid ${tf===t?C.cyan:C.border}`}}>{t}</button>)}
        </div>
      </div>
      <button onClick={run} disabled={running}
        style={{width:"100%",padding:"10px 0",borderRadius:7,fontSize:10,fontWeight:700,
          fontFamily:"'Space Mono',monospace",letterSpacing:"0.1em",cursor:running?"not-allowed":"pointer",
          color:running?C.dim:C.bg,background:running?C.muted:C.gold,border:"none"}}>
        {running?"RUNNING BACKTEST...":"RUN BACKTEST"}
      </button>
    </Box>
    {result&&<>
      <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:7,marginBottom:9}}>
        {[["Win Rate",(result.wr*100).toFixed(0)+"%",result.wr>0.6?C.green:result.wr>0.4?C.gold:C.red],
          ["Profit Factor",result.pf,parseFloat(result.pf)>1.5?C.green:parseFloat(result.pf)>1?C.gold:C.red],
          ["Return %",(parseFloat(result.returnPct)>=0?"+":"")+result.returnPct+"%",parseFloat(result.returnPct)>=0?C.green:C.red],
          ["Max DD","$"+parseFloat(result.maxDD).toLocaleString(),parseFloat(result.maxDD)>settings.equity*0.1?C.red:C.gold],
          ["Total Trades",result.total,C.text],["Wins/Loss",result.wins+"/"+result.losses,C.cyan],
        ].map(([l,v,c])=><Box key={l} style={{padding:"9px 11px"}}>
          <Label ch={l} style={{fontSize:8}}/><Num v={v} col={c} sz={13} w="6ch"/>
        </Box>)}
      </div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:7,marginBottom:9}}>
        <Box style={{padding:"9px 11px"}}><Label ch="Avg Win" style={{fontSize:8}}/><Num v={"+"+result.avgWin+"%"} col={C.green} sz={13} w="6ch"/></Box>
        <Box style={{padding:"9px 11px"}}><Label ch="Avg Loss" style={{fontSize:8}}/><Num v={"-"+result.avgLoss+"%"} col={C.red} sz={13} w="6ch"/></Box>
      </div>
      {result.results.length>2&&<Box style={{padding:"12px 13px"}}>
        <Label ch={`Equity Curve · ${tf} · ${result.results.length} trades`} style={{marginBottom:5}}/>
        <div style={{height:130}}>
          <ResponsiveContainer width="100%" height="100%">
            <AreaChart data={result.results} margin={{top:3,right:3,bottom:0,left:0}}>
              <defs><linearGradient id="btG" x1="0" y1="0" x2="0" y2="1">
                <stop offset="5%" stopColor={parseFloat(result.returnPct)>=0?C.green:C.red} stopOpacity={0.25}/>
                <stop offset="95%" stopColor={parseFloat(result.returnPct)>=0?C.green:C.red} stopOpacity={0.01}/>
              </linearGradient></defs>
              <XAxis hide/>
              <YAxis domain={["auto","auto"]} tick={{fill:C.dim,fontSize:8,fontFamily:"'Space Mono',monospace"}}
                tickLine={false} axisLine={false} tickFormatter={v=>"$"+v.toLocaleString()} width={55}/>
              <ReferenceLine y={settings.equity} stroke={C.border} strokeDasharray="3 3"/>
              <Tooltip content={<ChartTip/>}/>
              <Area type="monotone" dataKey="equity" name="Equity $"
                stroke={parseFloat(result.returnPct)>=0?C.green:C.red}
                strokeWidth={1.5} fill="url(#btG)" dot={false} isAnimationActive={false}/>
            </AreaChart>
          </ResponsiveContainer>
        </div>
      </Box>}
    </>}
    {!result&&!running&&<Box style={{padding:"13px",textAlign:"center"}}>
      <div style={{fontSize:9,color:C.dim,fontFamily:"'Space Mono',monospace"}}>
        Select timeframe and run to see historical equity curve
      </div>
    </Box>}
  </section>;
});

/* ─── SYSTEM STATUS ──────────────────────────────────────────────────── */
const SystemStatus=memo(function SystemStatus({display,regime,candleVer}){
  const{connected,loading,error}=display;
  const rows=[
    {n:"WebSocket",ok:connected,note:connected?"LIVE":"DOWN"},
    {n:"Price Stream",ok:display.price>0,note:display.price>0?"ACTIVE":"WAIT"},
    {n:"Depth (600ms)",ok:display.price>0,note:"500ms WS → 600ms flush"},
    {n:"Funding/OI",ok:display.fundRate!==0,note:display.fundRate!==0?"ACTIVE":"WAIT"},
    {n:"1m Closed Candles",ok:candleVer>0,note:"ver "+candleVer},
    {n:"15m/1h/4h/1d",ok:display.oi>0,note:"REST 60s refresh"},
    {n:"Regime Engine",ok:regime.regime!=="LOADING",note:regime.regime?.replace(/_/g," ")||"INIT"},
    {n:"Signal Engine",ok:true,note:"Closed candle only"},
    {n:"Chandelier Stop",ok:true,note:"ATR×"+CHANDELIER_M+" · 22p"},
    {n:"No Repaint",ok:true,note:"Last candle stripped"},
  ];
  const allOk=connected&&candleVer>0;
  return <section style={{marginBottom:32}}>
    <SecTitle ch="System Status"/>
    <Box style={{padding:"13px 14px"}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:10}}>
        <div style={{display:"flex",alignItems:"center",gap:6}}>
          <Dot col={allOk?C.green:C.orange} pulse={allOk}/>
          <span style={{fontFamily:"'Rajdhani',sans-serif",fontWeight:700,fontSize:13,
            color:allOk?C.green:C.orange,letterSpacing:"0.08em"}}>
            {allOk?"ALL OPERATIONAL":loading?"INITIALIZING":"PARTIAL"}
          </span>
        </div>
        <Num v={"v"+candleVer} col={C.dim} sz={9}/>
      </div>
      {error&&<div style={{fontSize:9,color:C.red,fontFamily:"'Space Mono',monospace",marginBottom:8}}>{error}</div>}
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:5}}>
        {rows.map(r=><div key={r.n} style={{display:"flex",justifyContent:"space-between",
          alignItems:"center",padding:"4px 8px",background:C.muted+"26",borderRadius:4,
          border:`1px solid ${C.border}`,minHeight:28}}>
          <div style={{display:"flex",alignItems:"center",gap:4}}>
            <Dot col={r.ok?C.green:C.orange}/>
            <span style={{fontSize:9,fontFamily:"'Rajdhani',sans-serif",fontWeight:600,color:C.text}}>{r.n}</span>
          </div>
          <span style={{fontSize:7,fontFamily:"'Space Mono',monospace",color:C.dim,textAlign:"right"}}>{r.note}</span>
        </div>)}
      </div>
      <div style={{marginTop:9,padding:"7px 10px",background:`${C.cyan}07`,borderRadius:5,
        border:`1px solid ${C.border}`,display:"flex",flexWrap:"wrap",gap:10}}>
        {[["Source","Binance Futures"],["No Repaint","✓ Closed Only"],
          ["Brain TTL",BRAIN_TTL/1000+"s"],["Signal TTL",SIGNAL_TTL/1000+"s"],
          ["Trail","Chandelier ATR"],["MC",MC_PATHS+" paths"]
        ].map(([k,v])=><div key={k}>
          <div style={{fontSize:7,color:C.dim,fontFamily:"'Space Mono',monospace"}}>{k}</div>
          <div style={{fontSize:9,fontFamily:"'Space Mono',monospace",fontWeight:700,color:C.cyan}}>{v}</div>
        </div>)}
      </div>
    </Box>
  </section>;
});

/* ─── ROOT APP ───────────────────────────────────────────────────────── */
export default function TradingPlatform(){
  const{display,kRef,candleVer,depth}=useMarketData();
  const of=useMemo(()=>calcOrderFlow(depth),[depth]);
  const{regime,signals,sw}=useAnalysis(kRef,candleVer,of);
  const[trades,dispatch]=useReducer(tradesReducer,[]);
  const[alertLog,addAlert]=useAlertLog();
  // ── Settings (persisted to localStorage) ──────────────────────────
  const[settings,setSettings]=useState(loadSettings);
  const[showSettings,setShowSettings]=useState(false);
  // ── Paper trading ─────────────────────────────────────────────────
  const[paper,paperDispatch]=useReducer(paperReducer,{
    equity:loadSettings().equity,startEquity:loadSettings().equity,
    trades:[],log:[],peak:loadSettings().equity,maxDD:0});

  // ── TMA: computed every time signals or trades change ──────────────
  const tma=useMemo(()=>runTMA(signals,trades,regime,of,display.fundRate),
    [signals,trades,regime.regime,regime.dir,of.pressure,display.fundRate]);

  // Inject fonts once
  useEffect(()=>{
    if(document.getElementById("tfx")) return;
    const l=document.createElement("link");l.id="tfx";l.rel="stylesheet";
    l.href="https://fonts.googleapis.com/css2?family=Rajdhani:wght@500;600;700&family=Space+Mono:ital,wght@0,400;0,700&family=Exo+2:wght@300;400;500;600&display=swap";
    document.head.appendChild(l);
  },[]);

  // Auto-open trade ONLY when TMA approves (conf≥75%, RR≥1.5, no scalp-lock, etc.)
  useEffect(()=>{
    if(!tma) return;
    Object.entries({snp:signals.snp,int:signals.int,trd:signals.trd}).forEach(([k,sig])=>{
      if(!sig) return;
      const verdict=tma[k];
      if(verdict?.verdict==="APPROVED"&&sig.finalConf>=ALERT_THRESH){
        dispatch({type:"OPEN",sig});
        addAlert("TRADE_OPEN",`${sig.id} ${sig.dir} @ $${sig.entry?.toLocaleString()} · ${(sig.finalConf*100).toFixed(0)}% conf`,C.green);
      } else if(verdict?.verdict==="DENIED"&&sig.finalConf>=SHOW_THRESH){
        addAlert("DENIED",`${sig?.id} blocked · ${verdict.reasons[0]||"criteria not met"}`,C.red);
      } else if(verdict?.verdict==="BLOCKED"){
        addAlert("BLOCKED",`${sig?.id} blocked · ${verdict.reasons[0]||"TMA gate"}`,C.orange);
      }
    });
  },[signals,tma]);

  // Alert on trade close
  const prevTradesRef=useRef([]);
  useEffect(()=>{
    const prev=prevTradesRef.current;
    trades.forEach(t=>{
      const wasOpen=prev.find(p=>p.id===t.id&&p.status==="OPEN");
      if(wasOpen&&t.status==="CLOSED"){
        const pnl=t.closePrice&&t.entry?((t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100).toFixed(2):null;
        addAlert("TRADE_CLOSE",
          `${t.id} ${t.dir} closed · ${t.outcome} ${pnl?(pnl>=0?"+":"")+pnl+"%":""}`,
          t.outcome==="SL"?C.red:C.green);
      }
    });
    prevTradesRef.current=trades.map(t=>({...t}));
  },[trades]);

  // Alert on TMA global mode change
  const prevModeRef=useRef("NORMAL");
  useEffect(()=>{
    if(!tma) return;
    if(tma.globalMode!==prevModeRef.current){
      addAlert("TMA_MODE",`Mode → ${tma.globalMode} · ${tma.globalReasons[0]||""}`,
        tma.globalMode==="NORMAL"?C.green:tma.globalMode==="PAUSED"?C.red:C.orange);
      prevModeRef.current=tma.globalMode;
    }
    tma.directives?.filter(d=>d.priority==="HIGH").forEach(d=>{
      addAlert("DIRECTIVE",`${d.id} · ${d.label} — ${d.desc}`,C.gold);
    });
    [signals.snp,signals.int,signals.trd].forEach(sig=>{
      if(sig&&sig.finalConf>=ALERT_THRESH){
        addAlert("SIGNAL",`${sig.id} ${sig.dir} · ${(sig.finalConf*100).toFixed(0)}% conf · ${sig.rr}x R:R · ${sig.strat}`,C.cyan);
      }
    });
  },[tma?.globalMode,tma?.directives?.length,signals.snp?.finalConf,signals.int?.finalConf,signals.trd?.finalConf]);

  // ── Paper trading: tick update on price ───────────────────────────
  const paperPriceRef=useRef(display.price);
  paperPriceRef.current=display.price;
  useEffect(()=>{
    if(!settings.paperMode) return;
    const iv=setInterval(()=>{
      const p=paperPriceRef.current;
      if(p>0) paperDispatch({type:"TICK",price:p});
    },3000);
    return()=>clearInterval(iv);
  },[settings.paperMode]);

  // ── Paper trading: auto-open when TMA approves + paper mode on ─────
  useEffect(()=>{
    if(!settings.paperMode||!tma) return;
    Object.entries({snp:signals.snp,int:signals.int,trd:signals.trd}).forEach(([k,sig])=>{
      if(!sig) return;
      const verdict=tma[k];
      if(verdict?.verdict==="APPROVED"&&sig.finalConf>=ALERT_THRESH){
        if(!paper.trades.find(t=>t.card===sig.id&&t.status==="OPEN")){
          paperDispatch({type:"OPEN",sig,riskPct:settings.riskPct});
        }
      }
    });
  },[signals,tma,settings.paperMode,settings.riskPct]);

  // ── Telegram notification engine ──────────────────────────────────
  const tgSentRef=useRef(new Set());
  useEffect(()=>{
    if(!settings.tgEnabled) return;
    [signals.snp,signals.int,signals.trd].forEach(sig=>{
      if(!sig||!settings.tgSignals) return;
      const key=`sig_${sig.id}_${sig.entry}_${sig.dir}`;
      if(!tgSentRef.current.has(key)&&sig.finalConf>=settings.confThresh){
        tgSentRef.current.add(key);
        tgSend([
          `🎯 *${sig.card} SIGNAL*`,
          `Direction: *${sig.dir}*`,
          `Entry: $${sig.entry?.toLocaleString()}`,
          `Stop Loss: $${sig.sl?.toLocaleString()}`,
          `TP1: $${sig.tp?.[0]?.toLocaleString()} · TP2: $${sig.tp?.[1]?.toLocaleString()||"—"}`,
          `Confidence: *${(sig.finalConf*100).toFixed(1)}%* · R:R: ${sig.rr}x`,
          `Strategy: ${sig.strat} · Regime: ${sig.regime}`,
          settings.paperMode?"📋 _Paper mode active_":"",
        ].filter(Boolean).join("\n"));
      }
    });
  },[signals.snp?.entry,signals.int?.entry,signals.trd?.entry,settings.tgEnabled,settings.tgSignals]);

  useEffect(()=>{
    if(!settings.tgEnabled||!settings.tgTrades) return;
    const prev=prevTradesRef.current;
    trades.forEach(t=>{
      const wasOpen=prev.find(p=>p.id===t.id&&p.status==="OPEN");
      if(wasOpen&&t.status==="CLOSED"){
        const pnl=t.closePrice&&t.entry?((t.closePrice-t.entry)*(t.dir==="LONG"?1:-1)/t.entry*100).toFixed(2):null;
        tgSend([
          `${t.outcome==="SL"?"❌":"✅"} *TRADE CLOSED · ${t.id}*`,
          `${t.dir} | Entry: $${t.entry?.toLocaleString()} → Exit: $${t.closePrice?.toLocaleString()}`,
          `Outcome: *${t.outcome}* | P&L: ${pnl?(pnl>=0?"+":"")+pnl+"%":"—"}`,
        ].join("\n"));
      }
    });
  },[trades]);

  useEffect(()=>{
    if(!settings.tgEnabled||!settings.tgAlerts||!tma) return;
    if(tma.globalMode!=="NORMAL"&&tma.globalMode!==prevModeRef.current){
      tgSend(`⚠️ *TMA MODE: ${tma.globalMode}*\n${tma.globalReasons[0]||""}`);
    }
    tma.directives?.filter(d=>d.priority==="HIGH").forEach(d=>{
      const key=`dir_${d.id}_${d.action}_${d.label}`;
      if(!tgSentRef.current.has(key)){
        tgSentRef.current.add(key);
        tgSend(`🔔 *DIRECTIVE · ${d.id}*\n${d.label}: ${d.desc}`);
      }
    });
  },[tma?.globalMode,tma?.directives?.length,settings.tgEnabled,settings.tgAlerts]);

  // Update trailing stops and PnL on every price tick (fast path — no re-render of analysis)
  const priceRef=useRef(display.price);
  priceRef.current=display.price;
  useEffect(()=>{
    const iv=setInterval(()=>{
      const p=priceRef.current;
      if(!p) return;
      dispatch({type:"CHECK_STOPS",price:p});
      dispatch({type:"UPDATE_TRAIL",id:"SNP",price:p,
        newTrailStop:kRef.current.k1m?TA.chandelier(
          closedOnly(kRef.current.k1m).map(r=>parseFloat(r[2])).slice(-25),
          closedOnly(kRef.current.k1m).map(r=>parseFloat(r[3])).slice(-25),
          closedOnly(kRef.current.k1m).map(r=>parseFloat(r[4])).slice(-25),
          22,CHANDELIER_M
        ).slice(-1)[0]?.long:undefined});
      dispatch({type:"UPDATE_TRAIL",id:"INT",price:p,
        newTrailStop:kRef.current.k15m?TA.chandelier(
          closedOnly(kRef.current.k15m).map(r=>parseFloat(r[2])).slice(-25),
          closedOnly(kRef.current.k15m).map(r=>parseFloat(r[3])).slice(-25),
          closedOnly(kRef.current.k15m).map(r=>parseFloat(r[4])).slice(-25),
          22,CHANDELIER_M
        ).slice(-1)[0]?.long:undefined});
      dispatch({type:"UPDATE_TRAIL",id:"TRD",price:p,
        newTrailStop:kRef.current.k4h?TA.chandelier(
          closedOnly(kRef.current.k4h).map(r=>parseFloat(r[2])).slice(-25),
          closedOnly(kRef.current.k4h).map(r=>parseFloat(r[3])).slice(-25),
          closedOnly(kRef.current.k4h).map(r=>parseFloat(r[4])).slice(-25),
          22,CHANDELIER_M
        ).slice(-1)[0]?.long:undefined});
    },2000); // 2s cadence for trailing stop updates — not every tick
    return()=>clearInterval(iv);
  },[]);

  const gs=`
    *{box-sizing:border-box;margin:0;padding:0;}
    html{background:${C.bg};-webkit-overflow-scrolling:touch;overscroll-behavior:none;}
    body{background:${C.bg};color:${C.text};font-family:'Exo 2',sans-serif;
      -webkit-font-smoothing:antialiased;overscroll-behavior-y:none;}
    /* Prevent scroll-triggered repaint / white flash */
    #__root,#root{background:${C.bg};}
    ::-webkit-scrollbar{width:3px;}
    ::-webkit-scrollbar-track{background:${C.muted}18;}
    ::-webkit-scrollbar-thumb{background:${C.border};border-radius:2px;}
    @keyframes blink{0%,100%{opacity:1}50%{opacity:0.3}}
    /* Tabular numbers globally */
    .tnum{font-variant-numeric:tabular-nums;}
    /* Promote scrollable container to own compositor layer — eliminates white flash */
    .scroll-root{transform:translateZ(0);will-change:transform;}
  `;

  return <>
    <style>{gs}</style>
    <div style={{minHeight:"100vh",background:C.bg,maxWidth:530,margin:"0 auto",
      position:"relative",isolation:"isolate"}}>
      {/* Grid bg */}
      <div aria-hidden style={{position:"fixed",inset:0,pointerEvents:"none",zIndex:0,
        backgroundImage:`linear-gradient(${C.border} 1px,transparent 1px),linear-gradient(90deg,${C.border} 1px,transparent 1px)`,
        backgroundSize:"36px 36px",opacity:0.22}}/>
      {/* Settings overlay */}
      {showSettings&&<SettingsPanel settings={settings}
        onSave={s=>{setSettings(s);paperDispatch({type:"RESET",equity:s.equity});}}
        onClose={()=>setShowSettings(false)}/>}
      {/* Header with settings button */}
      <div style={{position:"sticky",top:0,zIndex:100,willChange:"transform",backfaceVisibility:"hidden"}}>
        <Header display={display} price={display.price}/>
        {/* Sub-bar: paper mode badge + settings gear */}
        <div style={{background:"rgba(2,7,17,0.99)",borderBottom:`1px solid ${C.border}`,
          padding:"4px 14px",display:"flex",alignItems:"center",justifyContent:"space-between"}}>
          <div style={{display:"flex",alignItems:"center",gap:7}}>
            {settings.paperMode&&<div style={{display:"flex",alignItems:"center",gap:4,
              padding:"2px 8px",background:`${C.gold}15`,border:`1px solid ${C.gold}30`,borderRadius:3}}>
              <Dot col={C.gold} pulse/>
              <span style={{fontSize:8,color:C.gold,fontFamily:"'Space Mono',monospace",
                letterSpacing:"0.1em",fontWeight:700}}>PAPER MODE</span>
            </div>}
            {settings.tgEnabled&&<div style={{display:"flex",alignItems:"center",gap:4}}>
              <span style={{fontSize:8,color:C.cyan,fontFamily:"'Space Mono',monospace"}}>📨 TG ON</span>
            </div>}
          </div>
          <button onClick={()=>setShowSettings(true)}
            style={{background:"none",border:`1px solid ${C.border}`,borderRadius:5,
              padding:"3px 10px",cursor:"pointer",fontSize:9,color:C.dim,
              fontFamily:"'Space Mono',monospace",letterSpacing:"0.08em"}}>
            ⚙ SETTINGS
          </button>
        </div>
      </div>
      {/* Scroll root */}
      <div className="scroll-root" style={{position:"relative",zIndex:1,padding:"0 12px"}}>
        <MarketOverview kRef={kRef} candleVer={candleVer} price={display.price} display={display} regime={regime}/>
        <MultiTFRSI kRef={kRef} candleVer={candleVer}/>
        <AIBrain regime={regime} of={of} signals={signals}/>
        <StrategyIntel regime={regime} sw={sw}/>
        <section>
          <SecTitle ch="Signal Cards" sub="Closed-candle only · locked until expiry · no repaint"/>
          <SignalCard sig={signals.snp}/>
          <SignalCard sig={signals.int}/>
          <SignalCard sig={signals.trd}/>
        </section>
        <TradeManagementAgent tma={tma} dispatch={dispatch} signals={signals}/>
        <section>
          <SecTitle ch="Active Trades" sub="Chandelier Exit trailing stop · TMA-approved only"/>
          <ActiveTrades trades={trades} dispatch={dispatch}/>
        </section>
        {settings.paperMode&&<PaperTradingPanel paper={paper} paperDispatch={paperDispatch}
          settings={settings} price={display.price}/>}
        <PredictedMove kRef={kRef} candleVer={candleVer} price={display.price} regime={regime}/>
        <HeatmapSection kRef={kRef} candleVer={candleVer} price={display.price}/>
        <SmartMoney display={display} depth={depth} of={of}/>
        <SimulationSection signals={signals} kRef={kRef} candleVer={candleVer}/>
        <BacktesterPanel kRef={kRef} settings={settings}/>
        <PerformanceAnalytics trades={trades}/>
        <AIStrategyInsights trades={trades} regime={regime} signals={signals} tma={tma} kRef={kRef} candleVer={candleVer}/>
        <AlertLog log={alertLog}/>
        <SystemStatus display={display} regime={regime} candleVer={candleVer}/>
        <div style={{textAlign:"center",padding:"10px",fontSize:8,color:C.dim,
          fontFamily:"'Space Mono',monospace",letterSpacing:"0.12em",borderTop:`1px solid ${C.border}`}}>
          LIVE BINANCE DATA · CLOSED CANDLES ONLY · NOT FINANCIAL ADVICE
        </div>
      </div>
    </div>
  </>;
}
