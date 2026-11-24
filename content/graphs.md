# Northern Ridges

> A living survey of Scandinavian terrain: point fields resolve into layered ridges, deep valleys cut between them, fjord water gathers below, and snow drifts through the pass. Move across the scene to shift the parallax; click for a gust of wind.

<div class="scene-stage">
  <div class="scene-stage__frame">
    <canvas id="graphCanvas" width="1000" height="560"></canvas>
    <div class="scene-stage__overlay">
      <span class="scene-stage__index" id="sceneIndex">01</span>
      <span class="scene-stage__label" id="sceneLabel">Point field</span>
    </div>
  </div>
  <div class="scene-stage__bar">
    <div class="scene-steps">
      <span class="active">Field</span><span>Ridges</span><span>Valleys</span><span>Snow</span><span>Drift</span>
    </div>
    <div class="scene-stage__actions">
      <button type="button" id="scenePause">Pause</button>
      <button type="button" id="sceneReset">Restart</button>
    </div>
  </div>
  <p class="scene-stage__hint">Move to shift the parallax &middot; click to stir a gust</p>
</div>

<script>
(function(){
  const canvas = document.getElementById('graphCanvas');
  if(!canvas) return;
  const ctx = canvas.getContext('2d');
  const W = 1000, H = 560;
  const dpr = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width = W*dpr; canvas.height = H*dpr;
  ctx.setTransform(dpr,0,0,dpr,0,0);

  const CYCLE = 18;
  const TAU = Math.PI*2;
  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));
  const ss = (a,b,x)=>{ const t=clamp((x-a)/(b-a),0,1); return t*t*(3-2*t); };
  const hash = n=>{ const x=Math.sin(n*127.1+311.7)*43758.5453; return x-Math.floor(x); };

  const indexEl = document.getElementById('sceneIndex');
  const labelEl = document.getElementById('sceneLabel');
  const steps = Array.from(document.querySelectorAll('.scene-steps span'));
  const pauseBtn = document.getElementById('scenePause');
  const resetBtn = document.getElementById('sceneReset');
  const pointer = { x:W/2, y:H/2, active:false, gust:0 };
  const PHASES = [
    { at:0,    name:'Point field' },
    { at:0.14, name:'Ridge survey' },
    { at:0.34, name:'Valleys & fjord' },
    { at:0.58, name:'Snow line' },
    { at:0.74, name:'Drift' }
  ];

  let paused = false, simTime = 0, stepIndex = -1, lastCycle = -1, last = performance.now();

  const motes = Array.from({length:80},(_,i)=>({
    x: hash(i)*W,
    y: hash(i+200)*H,
    r: 0.6+hash(i+400)*1.9,
    vy: 8+hash(i+600)*26,
    ph: hash(i+800)*TAU
  }));

  if(pauseBtn) pauseBtn.addEventListener('click', ()=>{
    paused = !paused;
    pauseBtn.textContent = paused ? 'Resume' : 'Pause';
  });
  if(resetBtn) resetBtn.addEventListener('click', ()=>{
    simTime = 0; paused = false; lastCycle = -1;
    if(pauseBtn) pauseBtn.textContent = 'Pause';
  });
  canvas.addEventListener('pointermove', e=>{
    const r = canvas.getBoundingClientRect();
    pointer.x = (e.clientX-r.left)/r.width*W;
    pointer.y = (e.clientY-r.top)/r.height*H;
    pointer.active = true;
  });
  canvas.addEventListener('pointerleave', ()=>{ pointer.active=false; });
  canvas.addEventListener('pointerdown', ()=>{ pointer.gust = 1; });

  function drawSky(t){
    const g = ctx.createLinearGradient(0,0,0,H);
    g.addColorStop(0,'#050b16');
    g.addColorStop(0.42,'#0c1f33');
    g.addColorStop(0.78,'#1b3a4d');
    g.addColorStop(1,'#264b58');
    ctx.fillStyle = g;
    ctx.fillRect(0,0,W,H);

    for(let b=0;b<3;b++){
      ctx.strokeStyle = 'rgba(120,230,190,'+(0.035+b*0.012)+')';
      ctx.lineWidth = 22-b*5;
      ctx.beginPath();
      for(let x=0;x<=W;x+=12){
        const y = 70+b*38 + Math.sin(x*0.006+t*0.35+b)*22 + Math.sin(x*0.014-t*0.2)*10;
        if(x===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
      }
      ctx.stroke();
    }

    for(let i=0;i<80;i++){
      const x = hash(i)*W;
      const y = hash(i+90)*H*0.55;
      const tw = 0.15+0.35*(0.5+0.5*Math.sin(t*1.4+i));
      ctx.fillStyle = 'rgba(220,236,255,'+tw+')';
      ctx.beginPath(); ctx.arc(x,y,0.5+hash(i+31)*1.1,0,TAU); ctx.fill();
    }
  }

  function drawField(p,t){
    const a = ss(0,0.13,p);
    if(a<=0) return;
    const gx = 26, gy = 15;
    const pos = (i,j)=>({
      x:i/(gx-1)*W + Math.sin(t*0.45+j*0.6)*3,
      y:j/(gy-1)*H*0.98 + Math.cos(t*0.4+i*0.5)*3
    });
    ctx.lineWidth = 0.6;
    for(let j=0;j<gy;j++){
      for(let i=0;i<gx;i++){
        const q = pos(i,j);
        if(i<gx-1){
          const r = pos(i+1,j);
          const d = Math.hypot(r.x-q.x,r.y-q.y);
          ctx.strokeStyle = 'rgba(140,205,235,'+(a*0.16*(1-d/95))+')';
          ctx.beginPath(); ctx.moveTo(q.x,q.y); ctx.lineTo(r.x,r.y); ctx.stroke();
        }
        if(j<gy-1){
          const r = pos(i,j+1);
          ctx.strokeStyle = 'rgba(140,205,235,'+(a*0.09)+')';
          ctx.beginPath(); ctx.moveTo(q.x,q.y); ctx.lineTo(r.x,r.y); ctx.stroke();
        }
        const pulse = 0.5+0.5*Math.sin(t*2+hash(i*gy+j)*7);
        ctx.fillStyle = 'rgba(170,225,250,'+(a*(0.22+pulse*0.4))+')';
        ctx.beginPath(); ctx.arc(q.x,q.y,1.1+pulse*1,0,TAU); ctx.fill();
      }
    }
  }

  function ridgeY(layer,u,t){
    const seed = [0.75,1.35,2.2,3.1][layer];
    const base = H*[0.56,0.65,0.75,0.86][layer];
    const amp  = [46,68,92,118][layer];
    const h = Math.sin(u*5.4*seed + t*0.05 + layer)*0.5
            + Math.sin(u*11.8*seed - t*0.04)*0.28
            + Math.sin(u*24.6*seed + layer*2)*0.16;
    const s = Math.pow(clamp(0.5+0.5*h,0,1),1.75);
    return base - s*amp*1.75;
  }

  function drawSnowCaps(layer,t,shift){
    const base = H*[0.56,0.65,0.75,0.86][layer];
    const amp  = [46,68,92,118][layer];
    const cutoff = base - amp*0.95;
    let candidates = [];
    for(let x=0;x<=W;x+=6){
      const y = ridgeY(layer,(x-shift)/W,t);
      if(y<cutoff) candidates.push({x,y});
    }
    if(!candidates.length) return;
    let groups = [], cur = [];
    for(const c of candidates){
      if(cur.length && c.x-cur[cur.length-1].x>14){ groups.push(cur); cur=[]; }
      cur.push(c);
    }
    if(cur.length) groups.push(cur);
    for(const g of groups){
      if(g.length<2) continue;
      const peak = g.reduce((a,b)=>a.y<b.y?a:b);
      const span = Math.min(g[g.length-1].x-g[0].x,46);
      ctx.fillStyle = 'rgba(235,245,255,'+(0.5+layer*0.12)+')';
      ctx.beginPath();
      ctx.moveTo(peak.x,peak.y);
      for(let i=0;i<g.length;i+=3) ctx.lineTo(g[i].x,g[i].y+span*0.28);
      ctx.lineTo(g[g.length-1].x,g[g.length-1].y+span*0.28);
      ctx.lineTo(peak.x,peak.y);
      ctx.fill();
    }
  }

  function drawRidges(p,t){
    const a = ss(0.08,0.42,p);
    if(a<=0) return;
    const reveal = clamp(a*1.1,0,1);
    const parallax = pointer.active ? (pointer.x-W/2)/W : 0;
    const palettes = [
      {fill:'rgba(122,158,182,0.34)', top:'rgba(8,22,36,0)', line:'rgba(190,220,235,0.5)'},
      {fill:'rgba(72,112,140,0.5)',  top:'rgba(8,20,34,0.02)', line:'rgba(170,210,235,0.62)'},
      {fill:'rgba(31,66,90,0.72)',   top:'rgba(6,16,28,0.1)', line:'rgba(140,200,230,0.78)'},
      {fill:'rgba(12,34,50,0.96)',   top:'rgba(4,12,22,0.4)', line:'rgba(0,210,255,0.92)'}
    ];
    for(let layer=0;layer<4;layer++){
      const shift = parallax*(layer+1)*14;
      const end = Math.floor(W*reveal)+60;
      if(end<4) continue;
      ctx.save();
      ctx.translate(shift,0);
      const C = palettes[layer];

      ctx.beginPath();
      ctx.moveTo(-60,H);
      for(let x=-60;x<=end;x+=5) ctx.lineTo(x,ridgeY(layer,x/W,t));
      ctx.lineTo(end,H);
      ctx.closePath();
      const g = ctx.createLinearGradient(0,H*0.25,0,H);
      g.addColorStop(0,C.fill); g.addColorStop(1,C.top);
      ctx.fillStyle = g;
      ctx.fill();

      ctx.beginPath();
      for(let x=-60;x<=end;x+=5){
        const y = ridgeY(layer,x/W,t);
        if(x===-60) ctx.moveTo(x,y); else ctx.lineTo(x,y);
      }
      ctx.strokeStyle = C.line;
      ctx.lineWidth = layer===3?2:1.2;
      ctx.stroke();

      if(reveal>0.55) drawSnowCaps(layer,t,shift);

      if(layer>=2){
        for(let x=-40;x<=end;x+=22){
          const y = ridgeY(layer,x/W,t);
          ctx.fillStyle = 'rgba(220,245,255,'+(layer===3?0.9:0.6)+')';
          ctx.beginPath(); ctx.arc(x,y,layer===3?2.2:1.6,0,TAU); ctx.fill();
        }
      }

      const mist = ctx.createLinearGradient(0,H*0.55,0,H*0.95);
      mist.addColorStop(0,'rgba(200,225,240,0)');
      mist.addColorStop(0.5,'rgba(190,220,238,'+(0.05+layer*0.02)+')');
      mist.addColorStop(1,'rgba(190,220,238,0)');
      ctx.fillStyle = mist;
      ctx.fillRect(-60,H*0.55,W+120,H*0.4);
      ctx.restore();
    }
  }

  function drawFjord(p,t){
    const a = ss(0.3,0.55,p);
    if(a<=0) return;
    const y0 = H*0.84;
    const g = ctx.createLinearGradient(0,y0,0,H);
    g.addColorStop(0,'rgba(28,66,84,'+(0.85*a)+')');
    g.addColorStop(1,'rgba(6,20,32,'+(0.95*a)+')');
    ctx.fillStyle = g;
    ctx.fillRect(0,y0,W,H-y0);

    ctx.save();
    ctx.globalAlpha = 0.13*a;
    ctx.translate(0,y0*2);
    ctx.scale(1,-1);
    ctx.beginPath();
    for(let x=0;x<=W;x+=6) ctx.lineTo(x,ridgeY(3,x/W,t));
    ctx.lineTo(W,H); ctx.lineTo(0,H); ctx.closePath();
    ctx.fillStyle = '#5ea8c8';
    ctx.fill();
    ctx.restore();

    ctx.lineWidth = 1;
    for(let i=0;i<16;i++){
      const y = y0+6+i*5.2;
      ctx.strokeStyle = 'rgba(170,215,235,'+(0.11*(1-i/16)*a)+')';
      ctx.beginPath();
      for(let x=0;x<=W;x+=16){
        const yy = y + Math.sin(x*0.02 + t*0.8 + i)*1.6;
        if(x===0) ctx.moveTo(x,yy); else ctx.lineTo(x,yy);
      }
      ctx.stroke();
    }
  }

  function drawMotes(dt,t){
    const wind = Math.sin(t*0.4)*10 + pointer.gust*90;
    for(const m of motes){
      m.x += (wind*0.3 + Math.sin(t*0.8+m.ph)*5)*dt;
      m.y += (m.vy + pointer.gust*30)*dt;
      if(m.y>H+8){ m.y=-8; m.x=hash(m.x|0+Math.floor(t))*W; }
      if(m.x>W+8) m.x=-8;
      if(m.x<-8) m.x=W+8;
      ctx.fillStyle = 'rgba(226,240,255,'+(0.18+0.25*(0.5+0.5*Math.sin(t+m.ph)))+')';
      ctx.beginPath(); ctx.arc(m.x,m.y,m.r,0,TAU); ctx.fill();
    }
    pointer.gust *= 0.94;
    if(pointer.gust<0.01) pointer.gust=0;
  }

  function draw(now){
    requestAnimationFrame(draw);
    const dt = Math.min((now-last)/1000,0.05);
    last = now;
    if(!paused) simTime += dt;
    const t = simTime;
    const p = (t%CYCLE)/CYCLE;
    if(Math.floor(t/CYCLE)!==lastCycle) lastCycle=Math.floor(t/CYCLE);

    drawSky(t);
    drawField(p,t);
    drawRidges(p,t);
    drawFjord(p,t);
    drawMotes(dt,t);

    const fade = 1-ss(0.96,1,p);
    if(fade<1){
      ctx.globalAlpha = 1-fade;
      ctx.fillStyle = '#050b16';
      ctx.fillRect(0,0,W,H);
      ctx.globalAlpha = 1;
    }

    let idx = 0;
    for(let i=0;i<PHASES.length;i++) if(p>=PHASES[i].at) idx=i;
    if(idx!==stepIndex){
      stepIndex = idx;
      if(indexEl) indexEl.textContent = String(idx+1).padStart(2,'0');
      if(labelEl) labelEl.textContent = PHASES[idx].name;
      steps.forEach((s,i)=>s.classList.toggle('active', i===idx));
    }
  }
  requestAnimationFrame(draw);
})();
</script>

## Layers of the range

<div class="grid-2">
  <div class="box">
    <h3>Point field</h3>
    <p>A lattice of nodes underlies the survey and resolves into the terrain.</p>
  </div>
  <div class="box">
    <h3>Ridges &amp; peaks</h3>
    <p>Four overlapping ranges sharpen into jagged Scandinavian summits with snow caps.</p>
  </div>
  <div class="box">
    <h3>Valleys &amp; fjord</h3>
    <p>Deep valleys open into dark water that mirrors the nearest ridge.</p>
  </div>
  <div class="box">
    <h3>Snow &amp; drift</h3>
    <p>Mist settles in the passes while snow drifts down through the scene.</p>
  </div>
</div>
