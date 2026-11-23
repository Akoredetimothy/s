# Interactive Graphs & Systems

> Interactive data visualizations, neural operator benchmarks, and kinematic graph simulations.

<div class="game-box">
  <div class="game-hud">
    <div>NODES: <b>36 ACTIVE</b></div>
    <div>FPS: <b id="graphFps">60</b></div>
    <div>MODE: <span style="color:var(--color-accent)">BLOOM CYCLE</span></div>
  </div>
  <canvas id="graphCanvas" width="700" height="320" class="game-canvas"></canvas>
</div>

<script>
(function(){
  const canvas = document.getElementById('graphCanvas');
  if(!canvas) return;
  const ctx = canvas.getContext('2d');
  const W = canvas.width, H = canvas.height;
  const accent = '#00A4EF';
  const baseY = H * 0.78;
  const cols = 20, rows = 8;

  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));
  const ss = (a,b,x)=>{ const t=clamp((x-a)/(b-a),0,1); return t*t*(3-2*t); };
  const hash = n=>{ const x=Math.sin(n*127.1+311.7)*43758.5453; return x-Math.floor(x); };

  function mountainY(i){
    const t = i/20;
    return baseY - (Math.sin(t*8.1)*0.5 + Math.sin(t*15.3+1.1)*0.28 + Math.sin(t*3.7+0.4)*0.45 + 0.78) * H * 0.34;
  }

  function drawSky(p, t){
    ctx.globalAlpha = ss(0.12,0.3,p);
    const g = ctx.createLinearGradient(0,0,0,H);
    g.addColorStop(0,'#0b1324');
    g.addColorStop(0.6,'#122033');
    g.addColorStop(1,'#1b2b3a');
    ctx.fillStyle = g;
    ctx.fillRect(0,0,W,H);
    const drift = (t*16) % (W+260);
    ctx.fillStyle = 'rgba(190,212,240,0.08)';
    for(let i=0;i<5;i++){
      const cx = ((drift + i*180) % (W+260)) - 130;
      const cy = 34 + (i%3)*28;
      ctx.beginPath();
      ctx.ellipse(cx,cy,74,13,0,0,Math.PI*2);
      ctx.ellipse(cx+40,cy-7,50,11,0,0,Math.PI*2);
      ctx.fill();
    }
    ctx.globalAlpha = 1;
  }

  function drawGrid(p){
    const a = ss(0,0.12,p);
    if(a<=0) return;
    ctx.strokeStyle = accent;
    ctx.lineWidth = 0.6;
    ctx.globalAlpha = a*0.26;
    for(let c=0;c<=cols;c++){
      const x = c/cols*W;
      ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,H); ctx.stroke();
    }
    for(let r=0;r<=rows;r++){
      const y = r/rows*H;
      ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(W,y); ctx.stroke();
    }
    for(let r=0;r<=rows;r++){
      for(let c=0;c<=cols;c++){
        const delay = ((r*(cols+1)+c)/((rows+1)*(cols+1)))*0.12;
        const pa = clamp((a-delay)/0.035,0,1);
        if(pa<=0) continue;
        ctx.globalAlpha = a*pa*0.85;
        ctx.fillStyle = accent;
        ctx.beginPath();
        ctx.arc(c/cols*W, r/rows*H, 1.6, 0, Math.PI*2);
        ctx.fill();
      }
    }
    ctx.globalAlpha = 1;
  }

  function drawMountain(p){
    const a = ss(0.1,0.32,p);
    if(a<=0) return;
    const shown = Math.max(1, Math.floor(a*20));
    ctx.beginPath();
    ctx.moveTo(0,H);
    ctx.lineTo(0,mountainY(0));
    for(let i=1;i<=shown;i++) ctx.lineTo(i/20*W, mountainY(i));
    ctx.lineTo(shown/20*W,H);
    ctx.closePath();
    const g = ctx.createLinearGradient(0,H*0.3,0,H);
    g.addColorStop(0,'rgba(0,164,239,0.22)');
    g.addColorStop(1,'rgba(0,164,239,0.02)');
    ctx.fillStyle = g;
    ctx.fill();
    ctx.beginPath();
    ctx.moveTo(0,mountainY(0));
    for(let i=1;i<=shown;i++) ctx.lineTo(i/20*W, mountainY(i));
    ctx.strokeStyle = accent;
    ctx.lineWidth = 1.6;
    ctx.stroke();
    for(let i=0;i<=shown;i++){
      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.arc(i/20*W, mountainY(i), 2.4, 0, Math.PI*2);
      ctx.fill();
    }
  }

  function drawFlower(p,t){
    const grow = ss(0.32,0.6,p);
    if(grow<=0) return null;
    const open = ss(0.6,0.78,p);
    const baseX = W*0.5;
    const stemH = grow*H*0.46;
    const sway = Math.sin(t*1.4)*7*grow;
    const fx = baseX+sway;
    const fy = baseY-stemH;

    ctx.beginPath();
    ctx.moveTo(baseX,baseY);
    ctx.quadraticCurveTo(baseX+sway*0.4, baseY-stemH*0.55, fx, fy);
    ctx.strokeStyle = '#7FBA00';
    ctx.lineWidth = 2.4;
    ctx.stroke();

    if(grow<1){
      ctx.strokeStyle = 'rgba(255,255,255,0.32)';
      ctx.lineWidth = 1;
      ctx.beginPath();
      for(let k=0;k<44;k++){
        const q = k/43;
        const ang = q*Math.PI*4 + t*2;
        const rad = q*19;
        const x = fx+Math.cos(ang)*rad;
        const y = fy+Math.sin(ang)*rad;
        if(k===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
      }
      ctx.stroke();
    }

    for(const s of [-1,1]){
      ctx.beginPath();
      ctx.ellipse(baseX+s*10, baseY-stemH*0.45, 10, 4, s*0.5, 0, Math.PI*2);
      ctx.fillStyle = 'rgba(127,186,0,0.55)';
      ctx.fill();
    }

    const petals = 7;
    const pr = 4+open*15;
    for(let i=0;i<petals;i++){
      const ang = (i/petals)*Math.PI*2 + t*0.25;
      const px = fx+Math.cos(ang)*pr*0.55;
      const py = fy+Math.sin(ang)*pr*0.55;
      ctx.save();
      ctx.translate(px,py);
      ctx.rotate(ang);
      ctx.beginPath();
      ctx.ellipse(0,0, pr*(0.45+open*0.55), pr*0.36, 0, 0, Math.PI*2);
      ctx.fillStyle = 'rgba(242,80,34,'+(0.35+open*0.5)+')';
      ctx.fill();
      ctx.restore();
    }
    ctx.beginPath();
    ctx.arc(fx,fy,3.5+open*2,0,Math.PI*2);
    ctx.fillStyle = '#FFB900';
    ctx.fill();
    return {x:fx, y:fy};
  }

  function drawSeeds(p,fx,fy){
    const a = ss(0.78,0.96,p);
    if(a<=0 || fx==null) return;
    for(let i=0;i<30;i++){
      const ang = hash(i)*Math.PI*2;
      const sp = 50+hash(i+40)*110;
      const d = a*sp;
      const x = fx+Math.cos(ang)*d;
      const y = fy+Math.sin(ang)*d + a*a*55;
      ctx.globalAlpha = (1-a)*0.9;
      ctx.fillStyle = '#FFB900';
      ctx.beginPath();
      ctx.ellipse(x,y,2.4,1.2,ang,0,Math.PI*2);
      ctx.fill();
      for(let j=0;j<2;j++){
        const ja = ang + (hash(i*7+j)-0.5)*0.8;
        const jd = d*(1.1+hash(j+i)*0.5);
        ctx.globalAlpha = (1-a)*0.32;
        ctx.fillStyle = '#cfd8e3';
        ctx.beginPath();
        ctx.arc(fx+Math.cos(ja)*jd, fy+Math.sin(ja)*jd+a*a*70, 1+hash(i+j)*1.6, 0, Math.PI*2);
        ctx.fill();
      }
    }
    ctx.globalAlpha = 1;
  }

  let last = performance.now(), frames = 0;
  const fpsEl = document.getElementById('graphFps');
  function draw(now){
    requestAnimationFrame(draw);
    const t = now/1000;
    const p = (t%14)/14;
    ctx.fillStyle = '#090d12';
    ctx.fillRect(0,0,W,H);
    drawSky(p,t);
    drawGrid(p);
    drawMountain(p);
    const f = drawFlower(p,t);
    drawSeeds(p, f ? f.x : null, f ? f.y : null);
    const fade = 1-ss(0.96,1,p);
    if(fade<1){
      ctx.globalAlpha = 1-fade;
      ctx.fillStyle = '#090d12';
      ctx.fillRect(0,0,W,H);
      ctx.globalAlpha = 1;
    }
    frames++;
    if(now-last>1000){
      if(fpsEl) fpsEl.textContent = Math.round(frames*1000/(now-last));
      frames = 0; last = now;
    }
  }
  requestAnimationFrame(draw);
})();
</script>

## System Benchmarks

<div class="grid-2">
  <div class="box">
    <h3>Neural Operator Speedup</h3>
    <p>surrogate vs standard CFD grid solvers</p>
    <div style="font-size: 2rem; font-weight: 900; color: var(--color-accent); margin-top:8px;">140x faster</div>
  </div>
  <div class="box">
    <h3>Kinematic Precision</h3>
    <p>multi-camera vision feedback loop accuracy</p>
    <div style="font-size: 2rem; font-weight: 900; color: var(--a3); margin-top:8px;">99.4% precision</div>
  </div>
</div>
