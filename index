import { useState, useEffect, useCallback } from "react";
import {
  PieChart, Pie, Cell, BarChart, Bar,
  XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, Legend
} from "recharts";
import {
  Wallet, TrendingUp, Plus, Eye, Star,
  ArrowUpRight, ArrowDownRight, RefreshCw, X,
  BarChart3, DollarSign, Activity, Shield,
  Loader2, Trash2
} from "lucide-react";

// ─── Theme ───
const C = {
  bg: "#060b18", bg2: "#0c1222", card: "#111827", card2: "#161f33",
  border: "#1e293b",
  green: "#10b981", greenDim: "#065f46",
  red: "#ef4444", redDim: "#7f1d1d",
  blue: "#3b82f6", blueDim: "#1e3a5f",
  amber: "#f59e0b", purple: "#8b5cf6", cyan: "#06b6d4",
  text: "#f1f5f9", textSec: "#94a3b8", muted: "#475569",
  gold: "#fbbf24"
};
const PIE_COLORS = [C.blue, C.green, C.purple, C.amber, C.cyan, C.red, "#ec4899", "#14b8a6"];
const fmt = (v, cur = "USD") => new Intl.NumberFormat("en-US", { style: "currency", currency: cur, maximumFractionDigits: v >= 1 ? 2 : 6 }).format(v);
const fmtPct = (v) => `${v >= 0 ? "+" : ""}${v.toFixed(2)}%`;
const fmtCompact = (v) => new Intl.NumberFormat("en-US", { notation: "compact", maximumFractionDigits: 1 }).format(v);

// ─── Asset Catalog ───
const ASSETS = [
  { id: "btc", symbol: "BTC", name: "Bitcoin", type: "crypto", cgId: "bitcoin" },
  { id: "eth", symbol: "ETH", name: "Ethereum", type: "crypto", cgId: "ethereum" },
  { id: "sol", symbol: "SOL", name: "Solana", type: "crypto", cgId: "solana" },
  { id: "ada", symbol: "ADA", name: "Cardano", type: "crypto", cgId: "cardano" },
  { id: "xrp", symbol: "XRP", name: "XRP", type: "crypto", cgId: "ripple" },
  { id: "doge", symbol: "DOGE", name: "Dogecoin", type: "crypto", cgId: "dogecoin" },
  { id: "avax", symbol: "AVAX", name: "Avalanche", type: "crypto", cgId: "avalanche-2" },
  { id: "dot", symbol: "DOT", name: "Polkadot", type: "crypto", cgId: "polkadot" },
  { id: "matic", symbol: "MATIC", name: "Polygon", type: "crypto", cgId: "matic-network" },
  { id: "link", symbol: "LINK", name: "Chainlink", type: "crypto", cgId: "chainlink" },
  { id: "aapl", symbol: "AAPL", name: "Apple Inc.", type: "stock", sector: "Tech" },
  { id: "msft", symbol: "MSFT", name: "Microsoft", type: "stock", sector: "Tech" },
  { id: "googl", symbol: "GOOGL", name: "Alphabet", type: "stock", sector: "Tech" },
  { id: "amzn", symbol: "AMZN", name: "Amazon", type: "stock", sector: "Tech" },
  { id: "nvda", symbol: "NVDA", name: "NVIDIA", type: "stock", sector: "Tech" },
  { id: "tsla", symbol: "TSLA", name: "Tesla", type: "stock", sector: "Auto" },
  { id: "meta", symbol: "META", name: "Meta", type: "stock", sector: "Tech" },
  { id: "bimbo", symbol: "BIMBOA.MX", name: "Grupo Bimbo", type: "stock", sector: "Consumer" },
  { id: "amx", symbol: "AMXB.MX", name: "América Móvil", type: "stock", sector: "Telecom" },
  { id: "walmex", symbol: "WALMEX.MX", name: "Walmart México", type: "stock", sector: "Retail" },
  { id: "voo", symbol: "VOO", name: "Vanguard S&P 500", type: "etf" },
  { id: "qqq", symbol: "QQQ", name: "Invesco QQQ", type: "etf" },
  { id: "vti", symbol: "VTI", name: "Vanguard Total Mkt", type: "etf" },
  { id: "naftrac", symbol: "NAFTRAC", name: "iShares NAFTRAC", type: "etf" },
];

// ─── Storage ───
const S = {
  async get(k) { try { const r = await window.storage.get(k); return r ? JSON.parse(r.value) : null; } catch { return null; } },
  async set(k, v) { try { await window.storage.set(k, JSON.stringify(v)); } catch(e) { console.error("store err", e); } }
};

// ─── Components ───
const Btn = ({ children, onClick, variant = "primary", disabled, loading, style }) => {
  const base = { padding: "10px 20px", borderRadius: 10, border: "none", cursor: disabled ? "not-allowed" : "pointer", fontWeight: 600, fontSize: 14, display: "inline-flex", alignItems: "center", gap: 8, transition: "all 0.2s", opacity: disabled ? 0.5 : 1, fontFamily: "inherit" };
  const vs = { primary: { background: `linear-gradient(135deg,${C.blue},${C.purple})`, color: "#fff" }, success: { background: C.green, color: "#fff" }, danger: { background: C.red, color: "#fff" }, ghost: { background: "transparent", color: C.textSec, border: `1px solid ${C.border}` } };
  return <button onClick={onClick} disabled={disabled||loading} style={{...base,...vs[variant],...style}}>{loading&&<Loader2 size={16} style={{animation:"spin 1s linear infinite"}}/>}{children}</button>;
};
const Inp = ({ label, ...p }) => (
  <div style={{ marginBottom: 14 }}>
    {label && <label style={{ display: "block", color: C.textSec, fontSize: 11, marginBottom: 5, fontWeight: 700, textTransform: "uppercase", letterSpacing: 1 }}>{label}</label>}
    <input {...p} style={{ width: "100%", padding: "10px 14px", background: C.bg2, border: `1px solid ${C.border}`, borderRadius: 8, color: C.text, fontSize: 14, outline: "none", fontFamily: "inherit", boxSizing: "border-box", ...p.style }} />
  </div>
);
const KPI = ({ icon, label, value, delta, sub, color = C.green }) => (
  <div style={{ background: C.card, border: `1px solid ${C.border}`, borderRadius: 14, padding: "18px 20px", flex: 1, minWidth: 155, position: "relative", overflow: "hidden" }}>
    <div style={{ position: "absolute", top: -20, right: -20, width: 80, height: 80, background: color, opacity: 0.06, borderRadius: "50%" }} />
    <div style={{ display: "flex", alignItems: "center", gap: 6, marginBottom: 8 }}>
      <div style={{ color, opacity: 0.8 }}>{icon}</div>
      <span style={{ color: C.muted, fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: 0.8 }}>{label}</span>
    </div>
    <div style={{ fontSize: 22, fontWeight: 800, color: C.text, fontFamily: "'SF Mono','Fira Code',monospace", letterSpacing: -0.5 }}>{value}</div>
    {delta !== undefined && <div style={{ display: "flex", alignItems: "center", gap: 4, marginTop: 5 }}>{delta>=0?<ArrowUpRight size={13} color={C.green}/>:<ArrowDownRight size={13} color={C.red}/>}<span style={{ color: delta>=0?C.green:C.red, fontSize: 12, fontWeight: 600 }}>{fmtPct(delta)}</span>{sub&&<span style={{color:C.muted,fontSize:11,marginLeft:3}}>{sub}</span>}</div>}
  </div>
);
const Modal = ({ open, onClose, title, children }) => {
  if (!open) return null;
  return <div style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,0.75)", display: "flex", alignItems: "center", justifyContent: "center", zIndex: 1000, backdropFilter: "blur(4px)" }} onClick={onClose}>
    <div style={{ background: C.card, border: `1px solid ${C.border}`, borderRadius: 16, padding: 24, width: "92%", maxWidth: 440, maxHeight: "85vh", overflowY: "auto" }} onClick={e=>e.stopPropagation()}>
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 18 }}>
        <h3 style={{ margin: 0, fontSize: 17, fontWeight: 700, color: C.text }}>{title}</h3>
        <X size={18} color={C.muted} style={{ cursor: "pointer" }} onClick={onClose} />
      </div>
      {children}
    </div>
  </div>;
};

// ═══ MAIN APP ═══
export default function App() {
  const [positions, setPositions] = useState([]);
  const [transactions, setTransactions] = useState([]);
  const [watchlist, setWatchlist] = useState([]);
  const [prices, setPrices] = useState({});
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [tab, setTab] = useState("portfolio");
  const [showTx, setShowTx] = useState(false);
  const [showWatch, setShowWatch] = useState(false);

  const load = useCallback(async () => {
    const [p, t, w] = await Promise.all([S.get("pos"), S.get("txs"), S.get("watch")]);
    if (p) setPositions(p);
    if (t) setTransactions(t);
    if (w) setWatchlist(w);
  }, []);

  const fetchPrices = useCallback(async (silent = false) => {
    if (!silent) setLoading(true); else setRefreshing(true);
    try {
      const ids = ASSETS.filter(a=>a.cgId).map(a=>a.cgId).join(",");
      const r = await fetch(`https://api.coingecko.com/api/v3/simple/price?ids=${ids}&vs_currencies=usd,mxn&include_24hr_change=true&include_market_cap=true`);
      const d = await r.json();
      const pm = {};
      ASSETS.forEach(a => { if (a.cgId && d[a.cgId]) pm[a.id] = { usd: d[a.cgId].usd, mxn: d[a.cgId].mxn, change: d[a.cgId].usd_24h_change, mcap: d[a.cgId].usd_market_cap }; });
      setPrices(pm);
    } catch(e) { console.error(e); }
    finally { setLoading(false); setRefreshing(false); }
  }, []);

  useEffect(() => { load().then(() => fetchPrices()); }, [load, fetchPrices]);

  const savePos = async p => { setPositions(p); await S.set("pos", p); };
  const saveTxs = async t => { setTransactions(t); await S.set("txs", t); };
  const saveWatch = async w => { setWatchlist(w); await S.set("watch", w); };

  // Calcs
  const posV = positions.map(p => {
    const a = ASSETS.find(x=>x.id===p.aid);
    const pr = prices[p.aid]?.usd||0;
    const val = p.qty*pr, cost = p.qty*p.avg, pnl = val-cost;
    return { ...p, a, pr, val, cost, pnl, pnlP: cost>0?(pnl/cost)*100:0, ch24: prices[p.aid]?.change||0 };
  }).sort((a,b)=>b.val-a.val);

  const totVal = posV.reduce((s,p)=>s+p.val,0);
  const totCost = posV.reduce((s,p)=>s+p.cost,0);
  const totPnL = totVal-totCost;
  const totPnLP = totCost>0?(totPnL/totCost)*100:0;

  const typeA = {}; posV.forEach(p=>{ const t=p.a?.type||"other"; typeA[t]=(typeA[t]||0)+p.val; });
  const pieD = Object.entries(typeA).map(([n,v])=>({name:n.toUpperCase(),value:v}));
  const barD = posV.filter(p=>p.val>0).map(p=>({name:p.a?.symbol||"?",value:p.val}));

  // ─── AddTx Modal ───
  const TxModal = () => {
    const [aid, setAid] = useState("");
    const [qty, setQty] = useState("");
    const [pr, setPr] = useState("");
    const [typ, setTyp] = useState("buy");
    const [srch, setSrch] = useState("");
    const fl = ASSETS.filter(a=>a.symbol.toLowerCase().includes(srch.toLowerCase())||a.name.toLowerCase().includes(srch.toLowerCase()));
    const sel = ASSETS.find(a=>a.id===aid);
    const lp = aid?prices[aid]?.usd:null;

    const save = async () => {
      if(!aid||!qty||!pr) return;
      const q=parseFloat(qty), p=parseFloat(pr);
      await saveTxs([{id:Date.now().toString(),aid,typ,qty:q,pr:p,tot:q*p,date:new Date().toISOString()},...transactions]);
      const ex = positions.find(x=>x.aid===aid);
      if (ex) {
        const np = positions.map(x=>{
          if(x.aid!==aid) return x;
          const nq=typ==="buy"?x.qty+q:Math.max(0,x.qty-q);
          return{...x,qty:nq,avg:typ==="buy"?((x.qty*x.avg)+(q*p))/(nq||1):x.avg};
        }).filter(x=>x.qty>0);
        await savePos(np);
      } else if(typ==="buy") { await savePos([...positions,{aid,qty:q,avg:p}]); }
      setShowTx(false);
    };

    return <Modal open={showTx} onClose={()=>setShowTx(false)} title="Nueva Transacción">
      <div style={{display:"flex",gap:8,marginBottom:14}}>
        {["buy","sell"].map(t=><button key={t} onClick={()=>setTyp(t)} style={{flex:1,padding:"10px",borderRadius:8,border:`1px solid ${typ===t?(t==="buy"?C.green:C.red):C.border}`,background:typ===t?(t==="buy"?C.greenDim:C.redDim):"transparent",color:typ===t?(t==="buy"?C.green:C.red):C.muted,fontWeight:700,fontSize:13,cursor:"pointer",textTransform:"uppercase",fontFamily:"inherit"}}>{t==="buy"?"Comprar":"Vender"}</button>)}
      </div>
      <Inp label="Buscar activo" placeholder="BTC, AAPL, VOO..." value={srch} onChange={e=>setSrch(e.target.value)} />
      {srch&&!sel&&<div style={{maxHeight:150,overflowY:"auto",marginBottom:14,border:`1px solid ${C.border}`,borderRadius:8}}>
        {fl.slice(0,8).map(a=><div key={a.id} onClick={()=>{setAid(a.id);setSrch("");if(prices[a.id])setPr(prices[a.id].usd.toString());}} style={{padding:"9px 14px",cursor:"pointer",display:"flex",justifyContent:"space-between",borderBottom:`1px solid ${C.border}08`}}>
          <div><span style={{color:C.text,fontWeight:700,fontSize:13}}>{a.symbol}</span><span style={{color:C.muted,fontSize:12,marginLeft:8}}>{a.name}</span></div>
          <span style={{color:C.textSec,fontSize:10,textTransform:"uppercase",background:C.bg,padding:"2px 7px",borderRadius:4}}>{a.type}</span>
        </div>)}
      </div>}
      {sel&&<div style={{background:C.bg2,borderRadius:8,padding:"10px 14px",marginBottom:14,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
        <div><span style={{color:C.text,fontWeight:700}}>{sel.symbol}</span><span style={{color:C.muted,fontSize:12,marginLeft:8}}>{sel.name}</span></div>
        <div style={{display:"flex",alignItems:"center",gap:8}}>{lp&&<span style={{color:C.green,fontSize:13}}>Live: {fmt(lp)}</span>}<X size={14} color={C.muted} style={{cursor:"pointer"}} onClick={()=>{setAid("");setPr("");}}/></div>
      </div>}
      <div style={{display:"flex",gap:10}}>
        <div style={{flex:1}}><Inp label="Cantidad" type="number" step="any" placeholder="0.00" value={qty} onChange={e=>setQty(e.target.value)}/></div>
        <div style={{flex:1}}><Inp label="Precio USD" type="number" step="any" placeholder="0.00" value={pr} onChange={e=>setPr(e.target.value)}/></div>
      </div>
      {qty&&pr&&<div style={{background:C.bg2,borderRadius:8,padding:"10px",marginBottom:14,textAlign:"center"}}>
        <span style={{color:C.muted,fontSize:11}}>TOTAL</span>
        <div style={{color:C.text,fontSize:20,fontWeight:800,fontFamily:"monospace"}}>{fmt(parseFloat(qty)*parseFloat(pr))}</div>
      </div>}
      <Btn onClick={save} variant={typ==="buy"?"success":"danger"} disabled={!aid||!qty||!pr} style={{width:"100%",justifyContent:"center",padding:"12px"}}>{typ==="buy"?"Confirmar Compra":"Confirmar Venta"}</Btn>
    </Modal>;
  };

  // ─── Watch Modal ───
  const WatchModal = () => {
    const [aid, setAid] = useState("");
    const [tgt, setTgt] = useState("");
    const [srch, setSrch] = useState("");
    const wIds = watchlist.map(w=>w.aid);
    const fl = ASSETS.filter(a=>!wIds.includes(a.id)&&(a.symbol.toLowerCase().includes(srch.toLowerCase())||a.name.toLowerCase().includes(srch.toLowerCase())));
    return <Modal open={showWatch} onClose={()=>setShowWatch(false)} title="Agregar a Watchlist">
      <Inp label="Buscar activo" placeholder="BTC, NVDA..." value={srch} onChange={e=>setSrch(e.target.value)}/>
      <div style={{maxHeight:180,overflowY:"auto",marginBottom:14,border:`1px solid ${C.border}`,borderRadius:8}}>
        {fl.slice(0,10).map(a=><div key={a.id} onClick={()=>setAid(a.id)} style={{padding:"9px 14px",cursor:"pointer",display:"flex",justifyContent:"space-between",background:aid===a.id?C.blueDim:"transparent",borderBottom:`1px solid ${C.border}08`}}>
          <span style={{color:C.text,fontWeight:600,fontSize:13}}>{a.symbol} <span style={{color:C.muted,fontWeight:400}}>{a.name}</span></span>
          <span style={{color:C.textSec,fontSize:10,textTransform:"uppercase"}}>{a.type}</span>
        </div>)}
      </div>
      <Inp label="Precio objetivo (opcional)" type="number" step="any" placeholder="USD" value={tgt} onChange={e=>setTgt(e.target.value)}/>
      <Btn onClick={async()=>{if(!aid)return;await saveWatch([...watchlist,{aid,tgt:tgt?parseFloat(tgt):null}]);setShowWatch(false);}} disabled={!aid} style={{width:"100%",justifyContent:"center"}}>Agregar</Btn>
    </Modal>;
  };

  if (loading) return <div style={{minHeight:"100vh",background:C.bg,display:"flex",alignItems:"center",justifyContent:"center",flexDirection:"column",gap:14,fontFamily:"'Inter',system-ui,sans-serif"}}>
    <Loader2 size={36} color={C.blue} style={{animation:"spin 1s linear infinite"}}/><span style={{color:C.textSec,fontSize:14}}>Cargando precios...</span>
    <style>{`@keyframes spin{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}`}</style>
  </div>;

  return <div style={{minHeight:"100vh",background:C.bg,fontFamily:"'Inter','Segoe UI',system-ui,sans-serif",color:C.text}}>
    <style>{`@keyframes spin{from{transform:rotate(0deg)}to{transform:rotate(360deg)}} *::-webkit-scrollbar{width:5px} *::-webkit-scrollbar-thumb{background:${C.border};border-radius:3px}`}</style>

    {/* Header */}
    <header style={{background:C.card,borderBottom:`1px solid ${C.border}`,padding:"12px 20px",display:"flex",justifyContent:"space-between",alignItems:"center",position:"sticky",top:0,zIndex:100}}>
      <div style={{display:"flex",alignItems:"center",gap:10}}>
        <div style={{width:34,height:34,borderRadius:10,background:`linear-gradient(135deg,${C.blue},${C.purple})`,display:"flex",alignItems:"center",justifyContent:"center"}}><BarChart3 size={18} color="#fff"/></div>
        <div><h1 style={{margin:0,fontSize:16,fontWeight:800}}>Portfolio Pro</h1><span style={{color:C.muted,fontSize:10}}>CoinGecko Live · Supabase Sync</span></div>
      </div>
      <Btn variant="ghost" onClick={()=>fetchPrices(true)} style={{padding:"7px 10px"}}><RefreshCw size={15} style={refreshing?{animation:"spin 1s linear infinite"}:{}}/></Btn>
    </header>

    {/* Tabs */}
    <div style={{display:"flex",gap:3,padding:"10px 20px",background:C.bg2,borderBottom:`1px solid ${C.border}`,overflowX:"auto"}}>
      {[{id:"portfolio",l:"Portafolio",i:<Wallet size={14}/>},{id:"watchlist",l:"Watchlist",i:<Eye size={14}/>},{id:"history",l:"Historial",i:<Activity size={14}/>}].map(t=>
        <button key={t.id} onClick={()=>setTab(t.id)} style={{padding:"7px 14px",borderRadius:7,border:"none",background:tab===t.id?C.card:"transparent",color:tab===t.id?C.text:C.muted,fontWeight:600,fontSize:12,cursor:"pointer",display:"flex",alignItems:"center",gap:5,fontFamily:"inherit",whiteSpace:"nowrap",boxShadow:tab===t.id?"0 1px 4px rgba(0,0,0,0.3)":"none"}}>{t.i}{t.l}</button>
      )}
    </div>

    <div style={{padding:"16px 20px",maxWidth:1100,margin:"0 auto"}}>

      {/* ═══ PORTFOLIO ═══ */}
      {tab==="portfolio"&&<>
        <div style={{display:"flex",gap:12,marginBottom:16,flexWrap:"wrap"}}>
          <KPI icon={<DollarSign size={16}/>} label="Valor Total" value={fmt(totVal)} delta={totPnLP} sub="total" color={C.blue}/>
          <KPI icon={<TrendingUp size={16}/>} label="P&L" value={fmt(totPnL)} delta={totPnLP} color={totPnL>=0?C.green:C.red}/>
          <KPI icon={<Wallet size={16}/>} label="Posiciones" value={positions.length.toString()} color={C.purple}/>
          <KPI icon={<Shield size={16}/>} label="Activos" value={`${ASSETS.length}`} color={C.amber}/>
        </div>

        {posV.length>0&&<div style={{display:"flex",gap:12,marginBottom:16,flexWrap:"wrap"}}>
          <div style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:14,padding:18,flex:1,minWidth:250}}>
            <h3 style={{margin:"0 0 12px",fontSize:13,fontWeight:700,color:C.textSec}}>Por Tipo</h3>
            <ResponsiveContainer width="100%" height={200}><PieChart><Pie data={pieD} cx="50%" cy="50%" innerRadius={50} outerRadius={78} paddingAngle={3} dataKey="value" stroke="none">{pieD.map((_,i)=><Cell key={i} fill={PIE_COLORS[i%PIE_COLORS.length]}/>)}</Pie><Tooltip formatter={v=>fmt(v)} contentStyle={{background:C.card,border:`1px solid ${C.border}`,borderRadius:8,fontSize:12}}/><Legend wrapperStyle={{fontSize:11}}/></PieChart></ResponsiveContainer>
          </div>
          <div style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:14,padding:18,flex:1.4,minWidth:300}}>
            <h3 style={{margin:"0 0 12px",fontSize:13,fontWeight:700,color:C.textSec}}>Valor por Activo</h3>
            <ResponsiveContainer width="100%" height={200}><BarChart data={barD} layout="vertical"><CartesianGrid stroke={C.border} strokeDasharray="3 3" horizontal={false}/><XAxis type="number" stroke={C.muted} tick={{fontSize:10}} tickFormatter={fmtCompact}/><YAxis type="category" dataKey="name" stroke={C.muted} tick={{fontSize:11,fontWeight:600}} width={65}/><Tooltip formatter={v=>fmt(v)} contentStyle={{background:C.card,border:`1px solid ${C.border}`,borderRadius:8,fontSize:12}}/><Bar dataKey="value" radius={[0,6,6,0]}>{barD.map((_,i)=><Cell key={i} fill={PIE_COLORS[i%PIE_COLORS.length]}/>)}</Bar></BarChart></ResponsiveContainer>
          </div>
        </div>}

        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
          <h3 style={{margin:0,fontSize:15,fontWeight:700}}>Mis Posiciones</h3>
          <Btn onClick={()=>setShowTx(true)} style={{padding:"7px 14px",fontSize:12}}><Plus size={15}/> Transacción</Btn>
        </div>

        {posV.length===0?
          <div style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:14,padding:36,textAlign:"center"}}>
            <Wallet size={36} color={C.muted} style={{marginBottom:10}}/>
            <p style={{color:C.textSec,margin:"0 0 6px",fontSize:15,fontWeight:600}}>Portafolio vacío</p>
            <p style={{color:C.muted,margin:"0 0 14px",fontSize:12}}>Registra tu primera compra</p>
            <Btn onClick={()=>setShowTx(true)}><Plus size={15}/> Primera Compra</Btn>
          </div>
        :
          <div style={{display:"flex",flexDirection:"column",gap:8}}>
            {posV.map(p=><div key={p.aid} style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:12,padding:"14px 16px",display:"flex",justifyContent:"space-between",alignItems:"center",flexWrap:"wrap",gap:10}}>
              <div style={{minWidth:120}}>
                <div style={{fontWeight:800,fontSize:15,color:C.text}}>{p.a?.symbol}</div>
                <div style={{color:C.muted,fontSize:11}}>{p.a?.name}</div>
                <div style={{color:C.textSec,fontSize:11,marginTop:2}}>{p.qty.toLocaleString(undefined,{maximumFractionDigits:6})} × {p.pr>0?fmt(p.pr):"—"}</div>
              </div>
              <div style={{textAlign:"right",minWidth:100}}>
                <div style={{fontSize:17,fontWeight:800,fontFamily:"monospace",color:C.text}}>{fmt(p.val)}</div>
                <div style={{color:p.pnl>=0?C.green:C.red,fontSize:12,fontWeight:700,marginTop:2}}>{fmt(p.pnl)} ({fmtPct(p.pnlP)})</div>
                {p.ch24!==0&&<div style={{color:p.ch24>=0?C.green:C.red,fontSize:11,marginTop:2}}>24h: {fmtPct(p.ch24)}</div>}
              </div>
              <Trash2 size={14} color={C.muted} style={{cursor:"pointer",opacity:0.4}} onClick={()=>savePos(positions.filter(x=>x.aid!==p.aid))}/>
            </div>)}
          </div>
        }
      </>}

      {/* ═══ WATCHLIST ═══ */}
      {tab==="watchlist"&&<>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
          <h3 style={{margin:0,fontSize:15,fontWeight:700}}><Star size={16} style={{verticalAlign:-2,marginRight:5,color:C.gold}}/>Watchlist</h3>
          <Btn onClick={()=>setShowWatch(true)} style={{padding:"7px 14px",fontSize:12}}><Plus size={15}/> Agregar</Btn>
        </div>
        {watchlist.length===0?
          <div style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:14,padding:36,textAlign:"center"}}>
            <Eye size={36} color={C.muted} style={{marginBottom:10}}/>
            <p style={{color:C.textSec,fontSize:15,fontWeight:600,margin:"0 0 14px"}}>Watchlist vacía</p>
            <Btn onClick={()=>setShowWatch(true)}><Plus size={15}/> Agregar</Btn>
          </div>
        :
          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(240px,1fr))",gap:10}}>
            {watchlist.map(w=>{const a=ASSETS.find(x=>x.id===w.aid);const p=prices[w.aid];return<div key={w.aid} style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:12,padding:16,position:"relative"}}>
              <Trash2 size={13} color={C.muted} style={{position:"absolute",top:12,right:12,cursor:"pointer",opacity:0.4}} onClick={()=>saveWatch(watchlist.filter(x=>x.aid!==w.aid))}/>
              <div style={{fontWeight:800,fontSize:15}}>{a?.symbol}</div>
              <div style={{color:C.muted,fontSize:11,marginBottom:8}}>{a?.name}</div>
              {p?<><div style={{fontSize:20,fontWeight:800,fontFamily:"monospace"}}>{fmt(p.usd)}</div>
                <div style={{color:p.change>=0?C.green:C.red,fontSize:12,fontWeight:600,display:"flex",alignItems:"center",gap:3,marginTop:3}}>{p.change>=0?<ArrowUpRight size={13}/>:<ArrowDownRight size={13}/>}{fmtPct(p.change)}</div>
              </>:<div style={{color:C.muted,fontSize:13}}>Sin precio live</div>}
              {w.tgt&&p&&<div style={{marginTop:8,padding:"6px 10px",background:C.bg2,borderRadius:6,fontSize:11,color:C.textSec}}>
                Target: <span style={{color:C.amber,fontWeight:700}}>{fmt(w.tgt)}</span>
                <span style={{marginLeft:6,color:p.usd>=w.tgt?C.green:C.muted}}>{p.usd>=w.tgt?"Alcanzado":fmtPct(((w.tgt-p.usd)/p.usd)*100)}</span>
              </div>}
            </div>;})}
          </div>
        }
      </>}

      {/* ═══ HISTORY ═══ */}
      {tab==="history"&&<>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
          <h3 style={{margin:0,fontSize:15,fontWeight:700}}><Activity size={16} style={{verticalAlign:-2,marginRight:5}}/>Historial</h3>
          {transactions.length>0&&<Btn variant="ghost" onClick={()=>saveTxs([])} style={{padding:"6px 12px",fontSize:11}}>Limpiar</Btn>}
        </div>
        {transactions.length===0?
          <div style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:14,padding:36,textAlign:"center"}}>
            <Activity size={36} color={C.muted} style={{marginBottom:10}}/>
            <p style={{color:C.textSec,fontSize:15,fontWeight:600,margin:0}}>Sin transacciones</p>
          </div>
        :
          <div style={{display:"flex",flexDirection:"column",gap:6}}>
            {transactions.map(tx=>{const a=ASSETS.find(x=>x.id===tx.aid);return<div key={tx.id} style={{background:C.card,border:`1px solid ${C.border}`,borderRadius:10,padding:"12px 16px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
              <div style={{display:"flex",alignItems:"center",gap:10}}>
                <div style={{width:32,height:32,borderRadius:8,display:"flex",alignItems:"center",justifyContent:"center",background:tx.typ==="buy"?C.greenDim:C.redDim}}>
                  {tx.typ==="buy"?<ArrowDownRight size={16} color={C.green}/>:<ArrowUpRight size={16} color={C.red}/>}
                </div>
                <div><div style={{fontWeight:700,fontSize:13}}>{tx.typ==="buy"?"Compra":"Venta"} — {a?.symbol}</div>
                  <div style={{color:C.muted,fontSize:11}}>{tx.qty.toLocaleString(undefined,{maximumFractionDigits:6})} × {fmt(tx.pr)}</div>
                </div>
              </div>
              <div style={{textAlign:"right"}}>
                <div style={{fontWeight:700,fontFamily:"monospace",color:tx.typ==="buy"?C.green:C.red,fontSize:14}}>{tx.typ==="buy"?"-":"+"}{fmt(tx.tot)}</div>
                <div style={{color:C.muted,fontSize:10}}>{new Date(tx.date).toLocaleDateString("es-MX",{day:"2-digit",month:"short",year:"numeric"})}</div>
              </div>
            </div>;})}
          </div>
        }
      </>}
    </div>

    <TxModal/><WatchModal/>

    <footer style={{padding:"16px 20px",textAlign:"center",borderTop:`1px solid ${C.border}`,marginTop:30}}>
      <p style={{color:C.muted,fontSize:10,margin:0}}>Precios live vía CoinGecko • Datos persisten entre sesiones • DB: Supabase portfolio-angello • No es asesoría financiera</p>
    </footer>
  </div>;
}
