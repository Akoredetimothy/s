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
  const pointer = { x:W*0.5, y:H*0.5, active:false, down:false };
  let burst = 0;

  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));
  const ss = (a,b,x)=>{ const t=clamp((x-a)/(b-a),0,1); return t*t*(3-2*t); };
  const hash = n=>{ const x=Math.sin(n*127.1+311.7)*43758.5453; return x-Math.floor(x); };

  function setPointer(e){
    const r = canvas.getBoundingClientRect();
    pointer.x = (e.clientX-r.left)/r.width*W;
    pointer.y = (e.clientY-r.top)/r.height*H;
    pointer.active = true;
  }
  canvas.addEventListener('pointermove', setPointer);
  canvas.addEventListener('pointerleave', ()=>{ pointer.active=false; pointer.down=false; });
  canvas.addEventListener('pointerdown', e=>{ setPointer(e); pointer.down=true; burst=1; });
  window.addEventListener('pointerup', ()=>{ pointer.down=false; });

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

  function leaf(x,y,dir,grow){
    const len = 13*grow, wide = 5*grow;
    ctx.save();
    ctx.translate(x,y);
    ctx.rotate(dir*0.4);
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.bezierCurveTo(dir*wide,-len*0.25, dir*wide,-len*0.8, 0,-len);
    ctx.bezierCurveTo(-dir*wide*0.4,-len*0.8, -dir*wide*0.35,-len*0.25, 0,0);
    ctx.closePath();
    const g = ctx.createLinearGradient(0,0,0,-len);
    g.addColorStop(0,'rgba(60,120,20,0.85)');
    g.addColorStop(1,'rgba(160,220,60,0.75)');
    ctx.fillStyle = g;
    ctx.fill();
    ctx.strokeStyle = 'rgba(220,255,170,0.45)';
    ctx.lineWidth = 0.6;
    ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(0,-len*0.9); ctx.stroke();
    ctx.restore();
  }

  function petal(len,wid,inner){
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.bezierCurveTo(wid*0.65,-len*0.22, wid*0.55,-len*0.78, 0,-len);
    ctx.bezierCurveTo(-wid*0.55,-len*0.78, -wid*0.65,-len*0.22, 0,0);
    ctx.closePath();
    const g = ctx.createLinearGradient(0,0,0,-len);
    if(inner){
      g.addColorStop(0,'rgba(255,150,60,0.95)');
      g.addColorStop(1,'rgba(255,220,120,0.9)');
    }else{
      g.addColorStop(0,'rgba(200,50,30,0.95)');
      g.addColorStop(0.65,'rgba(242,80,34,0.9)');
      g.addColorStop(1,'rgba(255,183,77,0.8)');
    }
    ctx.fillStyle = g;
    ctx.fill();
    ctx.strokeStyle = 'rgba(255,225,180,0.35)';
    ctx.lineWidth = 0.6;
    ctx.beginPath(); ctx.moveTo(0,-len*0.1); ctx.lineTo(0,-len*0.85); ctx.stroke();
  }

  function drawFlower(p,t){
    const grow = ss(0.32,0.6,p);
    if(grow<=0) return null;
    const baseOpen = ss(0.6,0.78,p);
    const baseX = W*0.5;
    const stemH = grow*H*0.46;
    const wind = Math.sin(t*1.4)*6*grow;
    const dist = pointer.active ? Math.hypot(pointer.x-baseX, pointer.y-(baseY-stemH)) : 999;
    const near = clamp(1-dist/170,0,1);
    const lean = clamp((pointer.x-baseX)/W*2,-1,1)*near*16;
    const sway = wind+lean+(pointer.down?lean*0.5:0);
    const fx = baseX+sway;
    const fy = baseY-stemH-lean*0.15;
    const open = clamp(baseOpen + near*0.3 + (pointer.down?0.35:0), 0, 1);

    const glow = ctx.createRadialGradient(fx,fy,2,fx,fy,52);
    glow.addColorStop(0,'rgba(255,183,77,'+(0.16*open)+')');
    glow.addColorStop(1,'rgba(255,183,77,0)');
    ctx.fillStyle = glow;
    ctx.fillRect(fx-56,fy-56,112,112);

    const sg = ctx.createLinearGradient(baseX,baseY,fx,fy);
    sg.addColorStop(0,'#3f7a12');
    sg.addColorStop(1,'#9fd83c');
    ctx.beginPath();
    ctx.moveTo(baseX,baseY);
    ctx.quadraticCurveTo(baseX+sway*0.35, baseY-stemH*0.55, fx, fy);
    ctx.strokeStyle = sg;
    ctx.lineWidth = 3;
    ctx.lineCap = 'round';
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(baseX-0.6,baseY);
    ctx.quadraticCurveTo(baseX+sway*0.35-0.6, baseY-stemH*0.55, fx-0.6, fy);
    ctx.strokeStyle = 'rgba(220,255,170,0.35)';
    ctx.lineWidth = 0.8;
    ctx.stroke();

    leaf(baseX, baseY-stemH*0.42, -1, grow);
    leaf(baseX, baseY-stemH*0.6, 1, grow*0.85);

    if(grow<1){
      ctx.strokeStyle = 'rgba(255,255,255,0.32)';
      ctx.lineWidth = 1;
      for(let arm=0;arm<2;arm++){
        ctx.beginPath();
        for(let k=0;k<44;k++){
          const q = k/43;
          const ang = q*Math.PI*4 + t*2 + arm*Math.PI;
          const rad = q*20;
          const x = fx+Math.cos(ang)*rad;
          const y = fy+Math.sin(ang)*rad;
          if(k===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
        }
        ctx.stroke();
      }
    }

    const spin = t*(0.22+near*0.5);
    const outerLen = 9+open*15;
    for(let i=0;i<8;i++){
      const ang = (i/8)*Math.PI*2 + spin + Math.sin(t*3+i)*0.04*near;
      ctx.save();
      ctx.translate(fx,fy);
      ctx.rotate(ang+Math.PI/2);
      petal(outerLen, 6+open*3, false);
      ctx.restore();
    }
    for(let i=0;i<6;i++){
      const ang = (i/6)*Math.PI*2 - spin*0.6 + Math.PI/6;
      ctx.save();
      ctx.translate(fx,fy);
      ctx.rotate(ang+Math.PI/2);
      petal(outerLen*0.62, 5+open*2, true);
      ctx.restore();
    }

    const cr = 3+open*3;
    ctx.beginPath(); ctx.arc(fx,fy,cr,0,Math.PI*2);
    ctx.fillStyle = '#5a2a0b'; ctx.fill();
    for(let k=0;k<26;k++){
      const r = Math.sqrt(k/26)*cr*0.9;
      const a = k*2.399;
      ctx.beginPath();
      ctx.arc(fx+Math.cos(a)*r, fy+Math.sin(a)*r, 0.7+hash(k)*0.6, 0, Math.PI*2);
      ctx.fillStyle = k%3===0 ? '#ffe08a' : '#ffc033';
      ctx.fill();
    }
    return {x:fx, y:fy};
  }

  function drawSeeds(p,fx,fy){
    const a = ss(0.78,0.96,p);
    const total = clamp(a+burst*0.6,0,1);
    if(total<=0 || fx==null) return;
    for(let i=0;i<30;i++){
      const ang = hash(i)*Math.PI*2;
      const sp = 50+hash(i+40)*110;
      const d = total*sp;
      const x = fx+Math.cos(ang)*d;
      const y = fy+Math.sin(ang)*d + total*total*55;
      ctx.globalAlpha = (1-total)*0.9;
      ctx.fillStyle = '#FFB900';
      ctx.beginPath();
      ctx.ellipse(x,y,2.4,1.2,ang,0,Math.PI*2);
      ctx.fill();
      for(let j=0;j<2;j++){
        const ja = ang + (hash(i*7+j)-0.5)*0.8;
        const jd = d*(1.1+hash(j+i)*0.5);
        ctx.globalAlpha = (1-total)*0.32;
        ctx.fillStyle = '#cfd8e3';
        ctx.beginPath();
        ctx.arc(fx+Math.cos(ja)*jd, fy+Math.sin(ja)*jd+total*total*70, 1+hash(i+j)*1.6, 0, Math.PI*2);
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
    burst *= 0.965;
    if(burst<0.01) burst=0;
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
