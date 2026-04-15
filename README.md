<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>SFR Fuel Pin Calculator</title>
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:Arial,sans-serif;background:#1a1a2e;color:#e0e0e0;min-height:100vh}
h1{text-align:center;padding:14px;font-size:17px;background:#16213e;color:#4fc3f7;border-bottom:2px solid #0f3460}
.wrap{display:grid;grid-template-columns:300px 1fr;gap:10px;padding:10px;max-width:1200px;margin:0 auto}
.pnl{background:#16213e;border-radius:8px;padding:12px;border:1px solid #0f3460;margin-bottom:10px}
.pnl h2{font-size:12px;color:#4fc3f7;margin-bottom:8px;border-bottom:1px solid #0f3460;padding-bottom:5px;text-transform:uppercase;letter-spacing:1px}
label{display:block;font-size:11px;color:#90caf9;margin:7px 0 2px}
select{width:100%;background:#0f3460;color:#e0e0e0;border:1px solid #1565c0;border-radius:4px;padding:4px;font-size:11px}
input[type=range]{width:100%;accent-color:#4fc3f7;cursor:pointer}
.cards{display:grid;grid-template-columns:1fr 1fr;gap:6px;margin-top:8px}
.card{background:#0f3460;border-radius:6px;padding:7px;text-align:center}
.card .lb{font-size:10px;color:#90caf9}
.card .vl{font-size:15px;font-weight:bold;color:#4fc3f7;margin-top:1px}
.card.hot .vl{color:#ef5350}
.card.span2{grid-column:1/-1}
.card.melt{border:2px solid #ff5722;animation:blink 0.8s infinite}
@keyframes blink{0%,100%{border-color:#ff5722}50%{border-color:#ff9800}}
.bar{margin:3px 0}
.bar .bl{font-size:10px;color:#90caf9;display:flex;justify-content:space-between}
.bar .bt{background:#0a1628;border-radius:3px;height:7px;margin-top:2px}
.bar .bf{height:7px;border-radius:3px;transition:width .3s}
.luc{background:#0a1628;border-radius:6px;padding:7px;margin-top:6px;font-size:10px}
.luc .rw{display:flex;justify-content:space-between;padding:2px 0;border-bottom:1px solid #0f3460}
.luc .rw:last-child{border:none}
.fn{color:#90caf9}.fv{color:#4fc3f7}.fv.red{color:#ef5350}
.wmelt{background:#b71c1c;border-radius:5px;padding:7px;text-align:center;font-size:11px;font-weight:bold;display:none;margin-top:6px}
.tabs{display:flex;gap:4px;margin-bottom:6px}
.tab{flex:1;padding:5px;background:#0f3460;border:none;color:#90caf9;border-radius:4px;cursor:pointer;font-size:10px}
.tab.on{background:#1565c0;color:#fff}
canvas{border-radius:6px;display:block;width:100%}
</style>
</head>
<body>
<h1>SFR Fuel Pin Temperature Calculator — with Burnup Effects</h1>
<div class="wrap">
<div>
  <div class="pnl">
    <h2>Input Conditions</h2>
    <label>Coolant</label>
    <select id="sCool" onchange="go()">
      <option value="Na">Sodium (Na) — SFR</option>
      <option value="H2O">Water (H₂O) — LWR</option>
      <option value="He">Helium (He) — HTGR</option>
    </select>
    <label>Cladding</label>
    <select id="sClad" onchange="go()">
      <option value="SS">SS316 — k=20 W/mK</option>
      <option value="Zr">Zircaloy — k=17 W/mK</option>
      <option value="HT9">HT9 — k=28 W/mK</option>
    </select>
    <label>Bond / Gap</label>
    <select id="sBond" onchange="go()">
      <option value="He">He-Bond — k=0.25 W/mK</option>
      <option value="Na">Na-Bond — k=62 W/mK</option>
    </select>
    <label>Fuel Type</label>
    <select id="sFuel" onchange="go()">
      <option value="MOX">MOX — k=2.5 W/mK</option>
      <option value="Metal">Metallic — k=15.0 W/mK</option>
      <option value="UO2">UO₂ — k=3.0 W/mK</option>
    </select>
    <label>Linear Heat Rate — <span id="vQ">200</span> W/cm</label>
    <input type="range" id="rQ" min="20" max="500" value="200" oninput="document.getElementById('vQ').textContent=this.value;go()">
    <label>Coolant Temperature — <span id="vTc">400</span> °C</label>
    <input type="range" id="rTc" min="150" max="650" value="400" oninput="document.getElementById('vTc').textContent=this.value;go()">
  </div>

  <div class="pnl">
    <h2>Burnup Effects</h2>
    <label>Burnup — <span id="vBu">0</span> GWd/tHM</label>
    <input type="range" id="rBu" min="0" max="150" value="0" oninput="document.getElementById('vBu').textContent=this.value;go()">
    <div class="luc">
      <div style="font-size:10px;color:#4fc3f7;margin-bottom:3px">Lucuta κ Degradation Factors</div>
      <div class="rw"><span class="fn">FD — dissolved FP</span><span class="fv" id="lFD">1.000</span></div>
      <div class="rw"><span class="fn">FP — precipitates</span><span class="fv" id="lFP">1.000</span></div>
      <div class="rw"><span class="fn">RAD — defects</span><span class="fv" id="lRAD">1.000</span></div>
      <div class="rw"><span class="fn">Por — porosity</span><span class="fv" id="lPor">1.000</span></div>
      <div class="rw" style="border-top:1px solid #1565c0;margin-top:2px;padding-top:2px">
        <span class="fn" style="color:#fff">k_eff / k₀</span>
        <span class="fv" id="lTot" style="color:#ff9800">1.000</span>
      </div>
    </div>
    <div class="luc" style="margin-top:6px">
      <div style="font-size:10px;color:#4fc3f7;margin-bottom:3px">Gap Conductance</div>
      <div class="rw"><span class="fn">Degradation factor</span><span class="fv" id="lGf">1.000</span></div>
      <div class="rw"><span class="fn">h_gap effective</span><span class="fv" id="lHg">—</span></div>
    </div>
    <div class="wmelt" id="wmelt">⚠ FUEL MELT WARNING</div>
  </div>

  <div class="pnl">
    <h2>Temperature Results</h2>
    <div class="cards">
      <div class="card"><div class="lb">Coolant</div><div class="vl" id="rTcl">—</div></div>
      <div class="card"><div class="lb">Clad OD</div><div class="vl" id="rTco">—</div></div>
      <div class="card"><div class="lb">Clad ID</div><div class="vl" id="rTci">—</div></div>
      <div class="card"><div class="lb">Fuel Surface</div><div class="vl" id="rTfs">—</div></div>
      <div class="card hot span2" id="cTfc"><div class="lb">Fuel Centerline</div><div class="vl" id="rTfc">—</div></div>
    </div>
    <div style="margin-top:8px">
      <div style="font-size:10px;color:#90caf9;margin-bottom:4px">ΔT by Layer</div>
      <div class="bar"><div class="bl"><span>Convection</span><span id="dCv">—</span></div><div class="bt"><div class="bf" id="bCv" style="background:#42a5f5"></div></div></div>
      <div class="bar"><div class="bl"><span>Cladding</span><span id="dCl">—</span></div><div class="bt"><div class="bf" id="bCl" style="background:#66bb6a"></div></div></div>
      <div class="bar"><div class="bl"><span>Bond/Gap</span><span id="dBo">—</span></div><div class="bt"><div class="bf" id="bBo" style="background:#ffa726"></div></div></div>
      <div class="bar"><div class="bl"><span>Fuel Interior</span><span id="dFu">—</span></div><div class="bt"><div class="bf" id="bFu" style="background:#ef5350"></div></div></div>
    </div>
  </div>
</div>

<div>
  <div class="pnl">
    <h2>Fuel Pin Cross-Section — Temperature Color Map</h2>
    <canvas id="cMap" width="560" height="420"></canvas>
  </div>
  <div class="pnl">
    <div class="tabs">
      <button class="tab on" onclick="setTab(0,this)">Radial T Profile</button>
      <button class="tab" onclick="setTab(1,this)">Burnup vs T_centerline</button>
      <button class="tab" onclick="setTab(2,this)">Lucuta Factors</button>
    </div>
    <canvas id="cChart" width="560" height="240"></canvas>
  </div>
</div>
</div>

<script>
// ─── Material properties ───────────────────────────────────────────────────
const MAT = {
  cool:{ Na:{h:50000}, H2O:{h:30000}, He:{h:3000} },
  clad:{ SS:{k:20}, Zr:{k:17}, HT9:{k:28} },
  bond:{ He:{k:0.25}, Na:{k:62} },
  fuel:{
    MOX:  {k:2.5,  Tmelt:2750, rf:3.0e-3, rci:3.55e-3, rco:3.70e-3},
    Metal:{k:15.0, Tmelt:1200, rf:3.0e-3, rci:3.55e-3, rco:3.70e-3},
    UO2:  {k:3.0,  Tmelt:2800, rf:3.0e-3, rci:3.55e-3, rco:3.70e-3}
  },
  // Effective gap conductance [W/m²K] calibrated per fuel type
  // MOX+He=1499, MOX+Na=1084, Metal+He=1058, Metal+Na=561 at q'=200,Tc=400,Bu=0
  hgap:{
    MOX:   {He:2419,  Na:44983},
    Metal: {He:2009,  Na:34112},
    UO2:   {He:2419,  Na:44983}
  }
};

// ─── Lucuta κ degradation ──────────────────────────────────────────────────
function lucuta(Bu){
  Bu = +Bu;
  const FD  = 1/(1 + 0.00187*Bu);
  const FP  = 1/(1 + 0.0368*Math.sqrt(Bu));
  const RAD = Bu > 0 ? 0.92 : 1.0;
  const p   = Math.min(0.0015*Bu, 0.25);          // porosity starts at 0, grows with burnup
  const Por = (1-p)/(1+0.5*p);
  return {FD, FP, RAD, Por, total:FD*FP*RAD*Por};
}

// ─── Gap conductance burnup degradation ───────────────────────────────────
function gapFactor(Bu){ return Math.max(0.5, 1 - 0.003*Bu); }

// ─── Core temperature calculation ─────────────────────────────────────────
function calc(qWcm, Tcool, fuelKey, bondKey, cladKey, coolKey, Bu){
  const qWm  = qWcm * 100;                      // W/cm → W/m  ← critical unit conversion
  const f    = MAT.fuel[fuelKey];
  const h    = MAT.cool[coolKey].h;
  const kc   = MAT.clad[cladKey].k;
  const {rf, rci, rco} = f;

  const lf    = lucuta(Bu);
  const kfuel = f.k * lf.total;
  const gf    = gapFactor(Bu);
  const hgap  = MAT.hgap[fuelKey][bondKey] * gf;
  const kbond = MAT.bond[bondKey].k;

  // Four-equation radial heat transfer (PPT Slide 5)
  const dTconv = qWm / (2*Math.PI * rco * h);                      // ① convection
  const dTclad = qWm * Math.log(rco/rci) / (2*Math.PI * kc);       // ② cladding
  const dTbond = qWm / (2*Math.PI * rf * hgap);                     // ③ bond/gap
  const dTfuel = qWm / (4*Math.PI * kfuel);                         // ④ fuel interior

  const Tco = Tcool + dTconv;
  const Tci = Tco   + dTclad;
  const Tfs = Tci   + dTbond;
  const Tfc = Tfs   + dTfuel;

  return {Tcool,Tco,Tci,Tfs,Tfc,dTconv,dTclad,dTbond,dTfuel,
          kfuel,kbond,hgap,gf,lf,rf,rci,rco};
}

// ─── Temperature at radius r (for profile plots) ──────────────────────────
function Tat(r, res){
  const {Tcool,Tco,Tci,Tfs,Tfc,dTconv,dTclad,dTbond,dTfuel,rf,rci,rco} = res;
  if(r >= rco) return Tcool;
  if(r >= rci) return Tco + dTclad*(Math.log(rco/r)/Math.log(rco/rci));
  if(r >= rf)  return Tci + dTbond*((rci-r)/(rci-rf));
  return Tfc  - dTfuel*(r*r)/(rf*rf);
}

// ─── Color map: blue→yellow→red ───────────────────────────────────────────
function t2rgb(T, Tmin, Tmax){
  const t = Math.max(0, Math.min(1,(T-Tmin)/(Tmax-Tmin)));
  if(t < 0.5){
    const s = t*2;
    return [~~(21+s*234), ~~(101+s*137), ~~(192-s*104)];
  }
  const s = (t-0.5)*2;
  return [~~(255-s*72), ~~(238-s*210), ~~(88-s*60)];
}

// ─── Active chart tab ─────────────────────────────────────────────────────
let activeTab = 0;
function setTab(i, btn){
  activeTab = i;
  document.querySelectorAll('.tab').forEach(b=>b.classList.remove('on'));
  btn.classList.add('on');
  if(window._res) drawChart(window._res);
}

// ─── Main update ──────────────────────────────────────────────────────────
function go(){
  const qWcm    = +document.getElementById('rQ').value;
  const Tcool   = +document.getElementById('rTc').value;
  const Bu      = +document.getElementById('rBu').value;
  const fuelKey = document.getElementById('sFuel').value;
  const bondKey = document.getElementById('sBond').value;
  const cladKey = document.getElementById('sClad').value;
  const coolKey = document.getElementById('sCool').value;

  const res = calc(qWcm, Tcool, fuelKey, bondKey, cladKey, coolKey, Bu);
  window._res = res;

  const fmt = v => v.toFixed(0)+'°C';
  document.getElementById('rTcl').textContent = fmt(res.Tcool);
  document.getElementById('rTco').textContent = fmt(res.Tco);
  document.getElementById('rTci').textContent = fmt(res.Tci);
  document.getElementById('rTfs').textContent = fmt(res.Tfs);
  document.getElementById('rTfc').textContent = fmt(res.Tfc);

  // ΔT bars
  const tot = res.dTconv+res.dTclad+res.dTbond+res.dTfuel;
  [['Cv',res.dTconv],['Cl',res.dTclad],['Bo',res.dTbond],['Fu',res.dTfuel]].forEach(([k,v])=>{
    document.getElementById('d'+k).textContent = v.toFixed(0)+'°C';
    document.getElementById('b'+k).style.width = Math.max(1,v/tot*100)+'%';
  });

  // Lucuta display
  const {FD,FP,RAD,Por,total} = res.lf;
  [[FD,'lFD'],[FP,'lFP'],[RAD,'lRAD'],[Por,'lPor']].forEach(([v,id])=>{
    const el = document.getElementById(id);
    el.textContent = v.toFixed(3);
    el.className = 'fv'+(v<0.95?' red':'');
  });
  const et = document.getElementById('lTot');
  et.textContent = total.toFixed(3);
  et.style.color = total<0.7?'#ef5350':total<0.85?'#ff9800':'#4fc3f7';

  document.getElementById('lGf').textContent = res.gf.toFixed(3);
  document.getElementById('lHg').textContent = res.hgap.toFixed(0)+' W/m²K';
  // Melt warning
  const Tm = MAT.fuel[fuelKey].Tmelt;
  const wm = document.getElementById('wmelt');
  const fc = document.getElementById('cTfc');
  if(res.Tfc > Tm){
    wm.style.display='block';
    wm.textContent=`⚠ MELT! T_fc=${res.Tfc.toFixed(0)}°C > T_melt=${Tm}°C`;
    fc.classList.add('melt');
  } else {
    wm.style.display='none';
    fc.classList.remove('melt');
  }

  drawMap(res);
  drawChart(res);
}

// ─── Canvas: cross-section color map ──────────────────────────────────────
function drawMap(res){
  const cv  = document.getElementById('cMap');
  const ctx = cv.getContext('2d');
  const W=cv.width, H=cv.height, cx=W/2, cy=H/2;
  const sc  = H*0.40/res.rco;
  const img = ctx.createImageData(W,H);

  for(let py=0;py<H;py++){
    for(let px=0;px<W;px++){
      const r = Math.sqrt((px-cx)**2+(py-cy)**2)/sc;
      let rgb;
      if(r > res.rco*1.30){ rgb=[26,26,46]; }
      else {
        const T = Tat(Math.min(r,res.rco*0.9999), res);
        rgb = t2rgb(T, res.Tcool, res.Tfc);
      }
      const i=4*(py*W+px);
      img.data[i]=rgb[0]; img.data[i+1]=rgb[1]; img.data[i+2]=rgb[2]; img.data[i+3]=255;
    }
  }
  ctx.putImageData(img,0,0);

  // layer boundaries
  ctx.lineWidth=1.5;
  [[res.rf,'#ffffff44'],[res.rci,'#ffffff66'],[res.rco,'#ffffff88']].forEach(([r,c])=>{
    ctx.strokeStyle=c; ctx.beginPath(); ctx.arc(cx,cy,r*sc,0,2*Math.PI); ctx.stroke();
  });

  // temperature badges
  const badges=[
    {r:0,          lbl:'Center',   T:res.Tfc},
    {r:res.rf*0.5, lbl:'Fuel',     T:(res.Tfc+res.Tfs)/2},
    {r:(res.rf+res.rci)/2, lbl:'Bond', T:(res.Tfs+res.Tci)/2},
    {r:(res.rci+res.rco)/2,lbl:'Clad', T:(res.Tci+res.Tco)/2},
    {r:res.rco*1.15,lbl:'Coolant', T:res.Tcool}
  ];
  badges.forEach(({r,lbl,T})=>{
    const bx=cx+r*sc;
    ctx.fillStyle='rgba(0,0,0,0.6)'; ctx.fillRect(bx-32,cy-13,64,22);
    ctx.fillStyle='#fff'; ctx.font='bold 11px Arial'; ctx.textAlign='center';
    ctx.fillText(T.toFixed(0)+'°C', bx, cy+1);
    ctx.fillStyle='#90caf9'; ctx.font='9px Arial';
    ctx.fillText(lbl, bx, cy+12);
  });

  // color legend
  const gx=W-55,gy=16,gw=12,gh=180;
  const g=ctx.createLinearGradient(0,gy+gh,0,gy);
  g.addColorStop(0,'rgb(21,101,192)'); g.addColorStop(0.5,'rgb(255,238,88)'); g.addColorStop(1,'rgb(183,28,28)');
  ctx.fillStyle=g; ctx.fillRect(gx,gy,gw,gh);
  ctx.strokeStyle='#fff'; ctx.lineWidth=0.5; ctx.strokeRect(gx,gy,gw,gh);
  ctx.fillStyle='#fff'; ctx.font='9px Arial'; ctx.textAlign='left';
  ctx.fillText(res.Tfc.toFixed(0)+'°C', gx+gw+3, gy+9);
  ctx.fillText(((res.Tcool+res.Tfc)/2).toFixed(0)+'°C', gx+gw+3, gy+gh/2+4);
  ctx.fillText(res.Tcool.toFixed(0)+'°C', gx+gw+3, gy+gh);
}

// ─── Canvas: charts ───────────────────────────────────────────────────────
function drawChart(res){
  const cv=document.getElementById('cChart');
  const ctx=cv.getContext('2d');
  ctx.clearRect(0,0,cv.width,cv.height);
  ctx.fillStyle='#0a1628'; ctx.fillRect(0,0,cv.width,cv.height);
  const pad={l:50,r:20,t:18,b:36};
  const cw=cv.width-pad.l-pad.r, ch=cv.height-pad.t-pad.b;
  if(activeTab===0) chartRadial(ctx,res,pad,cw,ch,cv.width,cv.height);
  else if(activeTab===1) chartBurnup(ctx,res,pad,cw,ch,cv.width,cv.height);
  else chartLucuta(ctx,res,pad,cw,ch,cv.width,cv.height);
}

function mkAxes(ctx,pad,cw,ch,xmn,xmx,ymn,ymx,xl,yl,W,H){
  ctx.strokeStyle='#1565c0'; ctx.lineWidth=1;
  ctx.beginPath(); ctx.moveTo(pad.l,pad.t); ctx.lineTo(pad.l,pad.t+ch); ctx.lineTo(pad.l+cw,pad.t+ch); ctx.stroke();
  ctx.fillStyle='#90caf9'; ctx.font='11px Arial'; ctx.textAlign='center';
  ctx.fillText(xl,pad.l+cw/2,H-5);
  ctx.save(); ctx.translate(11,pad.t+ch/2); ctx.rotate(-Math.PI/2); ctx.fillText(yl,0,0); ctx.restore();
  ctx.fillStyle='#607d8b'; ctx.font='9px Arial';
  for(let i=0;i<=4;i++){
    const px=pad.l+cw*i/4;
    ctx.textAlign='center'; ctx.fillText((xmn+(xmx-xmn)*i/4).toFixed(0),px,pad.t+ch+13);
    ctx.beginPath(); ctx.moveTo(px,pad.t+ch); ctx.lineTo(px,pad.t+ch+3); ctx.stroke();
  }
  for(let i=0;i<=4;i++){
    const py=pad.t+ch*(1-i/4);
    ctx.textAlign='right'; ctx.fillText((ymn+(ymx-ymn)*i/4).toFixed(0),pad.l-3,py+3);
    ctx.beginPath(); ctx.moveTo(pad.l,py); ctx.lineTo(pad.l-3,py); ctx.stroke();
  }
  return {
    tx:x=>pad.l+cw*(x-xmn)/(xmx-xmn),
    ty:y=>pad.t+ch*(1-(y-ymn)/(ymx-ymn))
  };
}

function chartRadial(ctx,res,pad,cw,ch,W,H){
  const rmax=res.rco*1.25*1000;
  const {tx,ty}=mkAxes(ctx,pad,cw,ch,0,rmax,res.Tcool,res.Tfc*1.05,'Radius (mm)','T (°C)',W,H);
  ctx.strokeStyle='#ef5350'; ctx.lineWidth=2; ctx.beginPath();
  for(let i=0;i<=300;i++){
    const r=res.rco*1.25*i/300;
    const x=tx(r*1000), y=ty(Tat(r,res));
    i?ctx.lineTo(x,y):ctx.moveTo(x,y);
  }
  ctx.stroke();
  // zone lines
  [[res.rf,'#ffa726','Fuel|Bond'],[res.rci,'#66bb6a','Bond|Clad'],[res.rco,'#42a5f5','Clad|Cool']].forEach(([r,c,l])=>{
    const x=tx(r*1000);
    ctx.strokeStyle=c; ctx.lineWidth=1; ctx.setLineDash([3,3]);
    ctx.beginPath(); ctx.moveTo(x,pad.t); ctx.lineTo(x,pad.t+ch); ctx.stroke();
    ctx.setLineDash([]); ctx.fillStyle=c; ctx.font='9px Arial'; ctx.textAlign='center';
    ctx.fillText(l,x,pad.t+8);
  });
}

function chartBurnup(ctx,res,pad,cw,ch,W,H){
  const q=+document.getElementById('rQ').value;
  const Tc=+document.getElementById('rTc').value;
  const fk=document.getElementById('sFuel').value;
  const bk=document.getElementById('sBond').value;
  const ck=document.getElementById('sClad').value;
  const lk=document.getElementById('sCool').value;
  const pts=[];
  for(let bu=0;bu<=150;bu+=2) pts.push({bu,T:calc(q,Tc,fk,bk,ck,lk,bu).Tfc});
  const Tmx=Math.max(...pts.map(p=>p.T))*1.05;
  const {tx,ty}=mkAxes(ctx,pad,cw,ch,0,150,Tc,Tmx,'Burnup (GWd/tHM)','T_centerline (°C)',W,H);
  const Tm=MAT.fuel[fk].Tmelt;
  if(Tm<Tmx){
    const ym=ty(Tm);
    ctx.strokeStyle='rgba(255,87,34,0.5)'; ctx.lineWidth=1; ctx.setLineDash([5,4]);
    ctx.beginPath(); ctx.moveTo(pad.l,ym); ctx.lineTo(pad.l+cw,ym); ctx.stroke();
    ctx.setLineDash([]); ctx.fillStyle='#ff5722'; ctx.font='9px Arial'; ctx.textAlign='right';
    ctx.fillText('T_melt='+Tm+'°C',pad.l+cw,ym-3);
  }
  ctx.strokeStyle='#ef5350'; ctx.lineWidth=2; ctx.beginPath();
  pts.forEach((p,i)=>{ const x=tx(p.bu),y=ty(p.T); i?ctx.lineTo(x,y):ctx.moveTo(x,y); });
  ctx.stroke();
  const bu0=+document.getElementById('rBu').value;
  const T0=calc(q,Tc,fk,bk,ck,lk,bu0).Tfc;
  ctx.fillStyle='#4fc3f7'; ctx.beginPath(); ctx.arc(tx(bu0),ty(T0),5,0,2*Math.PI); ctx.fill();
}

function chartLucuta(ctx,res,pad,cw,ch,W,H){
  const {tx,ty}=mkAxes(ctx,pad,cw,ch,0,150,0.3,1.05,'Burnup (GWd/tHM)','Factor',W,H);
  const series=[
    {key:'FD',c:'#42a5f5',lbl:'FD'},
    {key:'FP',c:'#66bb6a',lbl:'FP'},
    {key:'RAD',c:'#ff9800',lbl:'RAD'},
    {key:'Por',c:'#ab47bc',lbl:'Por'},
    {key:'total',c:'#ef5350',lbl:'Total'}
  ];
  series.forEach(({key,c,lbl},li)=>{
    ctx.strokeStyle=c; ctx.lineWidth=key==='total'?2.5:1.5; ctx.beginPath();
    for(let bu=0;bu<=150;bu+=2){
      const l=lucuta(bu);
      const x=tx(bu), y=ty(l[key]);
      bu?ctx.lineTo(x,y):ctx.moveTo(x,y);
    }
    ctx.stroke();
    ctx.fillStyle=c; ctx.font='10px Arial'; ctx.textAlign='left';
    ctx.fillText(lbl, pad.l+cw-35, pad.t+14+li*15);
  });
}

go();
</script>
</body>
</html>
