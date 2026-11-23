# Generative Bloom

> A living field: points resolve into a mountain ridge, sky drifts overhead, and a bloom unfurls to scatter its seeds into the wind. Move across the canvas to lean the flower; click to release a fresh burst.

<div class="game-box">
  <div class="game-hud">
    <div>PHASE: <b id="graphPhase">FIELD</b></div>
    <div>FPS: <b id="graphFps">60</b></div>
    <div>SEEDS: <b id="graphSeeds">000</b></div>
  </div>
  <canvas id="graphCanvas" width="1000" height="520" class="game-canvas"></canvas>
  <p class="game-hint">Move to lean the bloom &middot; click to scatter seeds</p>
</div>

<script>
(function(){
  const canvas = document.getElementById('graphCanvas');
  if(!canvas) return;
  const ctx = canvas.getContext('2d');
  const W = 1000, H = 520;
  const dpr = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width = W*dpr; canvas.height = H*dpr;
  ctx.setTransform(dpr,0,0,dpr,0,0);

  const CYCLE = 18;
  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));
  const ss = (a,b,x)=>{ const t=clamp((x-a)/(b-a),0,1); return t*t*(3-2*t); };
  const hash = n=>{ const x=Math.sin(n*127.1+311.7)*43758.5453; return x-Math.floor(x); };

  const phaseEl = document.getElementById('graphPhase');
  const fpsEl = document.getElementById('graphFps');
  const seedsEl = document.getElementById('graphSeeds');
  const pointer = { x:W/2, y:H/2, active:false, down:false, impulse:0 };
  let particles = [], seedSpawned = false, lastCycle = -1;

  function setPointer(e){
    const r = canvas.getBoundingClientRect();
    pointer.x = (e.clientX-r.left)/r.width*W;
    pointer.y = (e.clientY-r.top)/r.height*H;
    pointer.active = true;
  }
  canvas.addEventListener('pointermove', setPointer);
  canvas.addEventListener('pointerleave', ()=>{ pointer.active=false; pointer.down=false; });
  canvas.addEventListener('pointerdown', e=>{ setPointer(e); pointer.impulse=1; });

  function spawnSeeds(x,y,n,power){
    for(let i=0;i<n;i++){
      const a = Math.random()*Math.PI*2;
      const sp = (45+Math.random()*150)*power;
      particles.push({
        x:x, y:y,
        vx:Math.cos(a)*sp, vy:Math.sin(a)*sp-55,
        life:1, size:1.3+Math.random()*2.4,
        dust:Math.random()<0.6, rot:a
      });
    }
  }

  function updateParticles(dt, wind){
    for(const q of particles){
      q.x += (q.vx + wind*(q.dust?1.4:0.7))*dt;
      q.y += q.vy*dt;
      q.vy += (q.dust?22:52)*dt;
      q.vx *= 0.991;
      q.rot += dt*2.4;
      q.life -= dt*(q.dust?0.62:0.3);
    }
    particles = particles.filter(q=>q.life>0);
  }

  function drawSky(t){
    const g = ctx.createLinearGradient(0,0,0,H);
    g.addColorStop(0,'#050b18');
    g.addColorStop(0.45,'#0b1a2e');
    g.addColorStop(0.78,'#173246');
    g.addColorStop(1,'#1e3b46');
    ctx.fillStyle = g;
    ctx.fillRect(0,0,W,H);

    const sun = ctx.createRadialGradient(W*0.5,H*0.62,4,W*0.5,H*0.62,240);
    sun.addColorStop(0,'rgba(255,196,120,0.22)');
    sun.addColorStop(0.4,'rgba(0,164,239,0.10)');
    sun.addColorStop(1,'rgba(0,164,239,0)');
    ctx.fillStyle = sun;
    ctx.fillRect(0,0,W,H);

    for(let i=0;i<70;i++){
      const x = hash(i)*W;
      const y = hash(i+90)*H*0.55;
      const tw = 0.18+0.32*(0.5+0.5*Math.sin(t*1.6+i));
      ctx.fillStyle = 'rgba(220,236,255,'+tw+')';
      ctx.beginPath(); ctx.arc(x,y,0.6+hash(i+31)*1.1,0,Math.PI*2); ctx.fill();
    }

    const layers = [
      {y:H*0.17, sp:7,  a:0.05, s:1.0},
      {y:H*0.27, sp:12, a:0.07, s:1.35},
      {y:H*0.38, sp:18, a:0.09, s:0.8}
    ];
    layers.forEach((L,li)=>{
      ctx.fillStyle = 'rgba(196,220,244,'+L.a+')';
      for(let i=0;i<6;i++){
        const drift = (t*L.sp + i*260 + li*90) % (W+320);
        const cx = drift-160;
        const cy = L.y + Math.sin(t*0.2+i)*6;
        ctx.beginPath();
        ctx.ellipse(cx,cy,96*L.s,13*L.s,0,0,Math.PI*2);
        ctx.ellipse(cx+52,cy-7,64*L.s,11*L.s,0,0,Math.PI*2);
        ctx.ellipse(cx-46,cy+3,58*L.s,10*L.s,0,0,Math.PI*2);
        ctx.fill();
      }
    });
  }

  function drawField(p,t){
    const a = ss(0,0.14,p);
    if(a<=0) return;
    const gx = 26, gy = 14;
    const pos = (i,j)=>({
      x:i/(gx-1)*W + Math.sin(t*0.5+j*0.6)*3,
      y:j/(gy-1)*H*0.98 + Math.cos(t*0.45+i*0.5)*3
    });
    ctx.lineWidth = 0.6;
    for(let j=0;j<gy;j++){
      for(let i=0;i<gx;i++){
        const q = pos(i,j);
        if(i<gx-1){
          const r = pos(i+1,j);
          const d = Math.hypot(r.x-q.x,r.y-q.y);
          ctx.strokeStyle = 'rgba(0,164,239,'+(a*0.16*(1-d/90))+')';
          ctx.beginPath(); ctx.moveTo(q.x,q.y); ctx.lineTo(r.x,r.y); ctx.stroke();
        }
        if(j<gy-1){
          const r = pos(i,j+1);
          ctx.strokeStyle = 'rgba(0,164,239,'+(a*0.1)+')';
          ctx.beginPath(); ctx.moveTo(q.x,q.y); ctx.lineTo(r.x,r.y); ctx.stroke();
        }
        const pulse = 0.5+0.5*Math.sin(t*2+hash(i*gy+j)*7);
        ctx.fillStyle = 'rgba(120,210,255,'+(a*(0.25+pulse*0.45))+')';
        ctx.beginPath(); ctx.arc(q.x,q.y,1.2+pulse*1.1,0,Math.PI*2); ctx.fill();
      }
    }
  }

  function ridgeY(layer,u,t){
    const f = [1,1.55,2.15][layer];
    const amp = [46,72,96][layer];
    const base = H*[0.6,0.71,0.83][layer];
    const h = Math.sin(u*6.2*f+t*0.13+layer)*0.5
            + Math.sin(u*13.5*f-t*0.1)*0.27
            + Math.sin(u*3.1*f+layer*1.7)*0.44 + 0.85;
    return base - h*amp;
  }

  function drawRidges(p,t){
    const a = ss(0.08,0.36,p);
    if(a<=0) return;
    const reveal = clamp(a*1.15,0,1);
    const colors = [
      ['rgba(90,170,215,0.28)','rgba(90,170,215,0.02)','rgba(150,215,245,0.55)'],
      ['rgba(35,110,160,0.42)','rgba(20,70,110,0.05)','rgba(120,205,245,0.75)'],
      ['rgba(10,45,70,0.85)','rgba(6,22,36,0.4)','rgba(0,200,255,0.95)']
    ];
    for(let layer=0; layer<3; layer++){
      const end = Math.floor(W*reveal);
      if(end<2) continue;
      ctx.beginPath();
      ctx.moveTo(0,H);
      for(let x=0;x<=end;x+=5) ctx.lineTo(x,ridgeY(layer,x/W,t));
      ctx.lineTo(end,H);
      ctx.closePath();
      const g = ctx.createLinearGradient(0,H*0.35,0,H);
      g.addColorStop(0,colors[layer][0]);
      g.addColorStop(1,colors[layer][1]);
      ctx.fillStyle = g;
      ctx.fill();

      ctx.beginPath();
      for(let x=0;x<=end;x+=5){
        const y = ridgeY(layer,x/W,t);
        if(x===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
      }
      ctx.strokeStyle = colors[layer][2];
      ctx.lineWidth = layer===2?2:1.2;
      ctx.stroke();

      if(layer===2){
        for(let x=0;x<=end;x+=25){
          const y = ridgeY(layer,x/W,t);
          ctx.fillStyle = '#d8f4ff';
          ctx.beginPath(); ctx.arc(x,y,2.4,0,Math.PI*2); ctx.fill();
        }
      }
    }
  }

  function leaf(x,y,dir,grow){
    const len = 20*grow, wide = 8*grow;
    ctx.save();
    ctx.translate(x,y);
    ctx.rotate(dir*0.42);
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.bezierCurveTo(dir*wide,-len*0.22, dir*wide,-len*0.78, 0,-len);
    ctx.bezierCurveTo(-dir*wide*0.45,-len*0.8, -dir*wide*0.4,-len*0.22, 0,0);
    ctx.closePath();
    const g = ctx.createLinearGradient(0,0,0,-len);
    g.addColorStop(0,'#2f6614'); g.addColorStop(1,'#a9e34f');
    ctx.fillStyle = g; ctx.fill();
    ctx.strokeStyle = 'rgba(222,255,180,0.5)';
    ctx.lineWidth = 0.8;
    ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(0,-len*0.9); ctx.stroke();
    ctx.restore();
  }

  function petal(len,wid,kind){
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.bezierCurveTo(wid*0.7,-len*0.2, wid*0.58,-len*0.76, 0,-len);
    ctx.bezierCurveTo(-wid*0.58,-len*0.76, -wid*0.7,-len*0.2, 0,0);
    ctx.closePath();
    const g = ctx.createLinearGradient(0,0,0,-len);
    if(kind===2){
      g.addColorStop(0,'#ff9a3c'); g.addColorStop(1,'#ffe9a8');
    }else if(kind===1){
      g.addColorStop(0,'#e8531f'); g.addColorStop(0.7,'#ff9d43'); g.addColorStop(1,'#ffd98a');
    }else{
      g.addColorStop(0,'#a52116'); g.addColorStop(0.55,'#ef5b28'); g.addColorStop(1,'#ffb45c');
    }
    ctx.fillStyle = g; ctx.fill();
    ctx.strokeStyle = 'rgba(255,232,190,0.35)';
    ctx.lineWidth = 0.7;
    ctx.beginPath(); ctx.moveTo(0,-len*0.12); ctx.lineTo(0,-len*0.86); ctx.stroke();
  }

  function drawBloom(p,t){
    const grow = ss(0.32,0.64,p);
    if(grow<=0) return null;
    const baseOpen = ss(0.6,0.8,p);
    const baseX = W/2;
    const baseH = H*0.86;
    const stemH = grow*H*0.42;
    const dist = pointer.active ? Math.hypot(pointer.x-baseX, pointer.y-(baseH-stemH)) : 999;
    const near = clamp(1-dist/210,0,1);
    const lean = clamp((pointer.x-baseX)/(W/2),-1,1)*near*22;
    const sway = Math.sin(t*1.1)*8*grow + lean + pointer.impulse*Math.sin(t*9)*6;
    const fx = baseX+sway;
    const fy = baseH-stemH-lean*0.12;
    const open = clamp(baseOpen + near*0.28 + pointer.impulse*0.35,0,1);

    const glow = ctx.createRadialGradient(fx,fy,2,fx,fy,90);
    glow.addColorStop(0,'rgba(255,186,96,'+(0.2*open)+')');
    glow.addColorStop(1,'rgba(255,186,96,0)');
    ctx.fillStyle = glow;
    ctx.fillRect(fx-95,fy-95,190,190);

    const sg = ctx.createLinearGradient(baseX,baseH,fx,fy);
    sg.addColorStop(0,'#2f6614'); sg.addColorStop(1,'#b6e85c');
    ctx.beginPath();
    ctx.moveTo(baseX,baseH);
    ctx.quadraticCurveTo(baseX+sway*0.32, baseH-stemH*0.55, fx,fy);
    ctx.strokeStyle = sg; ctx.lineWidth = 4; ctx.lineCap='round'; ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(baseX-1,baseH);
    ctx.quadraticCurveTo(baseX+sway*0.32-1, baseH-stemH*0.55, fx-1, fy);
    ctx.strokeStyle = 'rgba(226,255,180,0.4)'; ctx.lineWidth=1; ctx.stroke();

    leaf(baseX,baseH-stemH*0.4,-1,grow);
    leaf(baseX,baseH-stemH*0.56,1,grow*0.9);
    leaf(baseX,baseH-stemH*0.24,1,grow*0.7);

    if(grow<1 || open>0.2){
      ctx.strokeStyle = 'rgba(255,232,190,'+(grow<1?0.3:0.12)+')';
      ctx.lineWidth=1.1;
      for(let arm=0;arm<3;arm++){
        ctx.beginPath();
        for(let k=0;k<52;k++){
          const q=k/51;
          const ang=q*Math.PI*5 + t*1.7 + arm*Math.PI*2/3;
          const rad=q*32*(1+open*0.4);
          const x=fx+Math.cos(ang)*rad;
          const y=fy+Math.sin(ang)*rad;
          if(k===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
        }
        ctx.stroke();
      }
    }

    const spin = t*(0.2+near*0.45) + pointer.impulse*t;
    const rings = [
      {n:10, len:13+open*27, wid:8+open*7, kind:0, dir:1},
      {n:8,  len:10+open*20, wid:7+open*5, kind:1, dir:-0.65},
      {n:6,  len:7+open*12,  wid:5+open*4, kind:2, dir:0.4}
    ];
    rings.forEach((R,ri)=>{
      for(let i=0;i<R.n;i++){
        const ang=(i/R.n)*Math.PI*2 + spin*R.dir + Math.sin(t*2.4+i*1.3+ri)*0.05*near;
        ctx.save();
        ctx.translate(fx,fy);
        ctx.rotate(ang+Math.PI/2);
        petal(R.len,R.wid,R.kind);
        ctx.restore();
      }
    });

    const cr=4+open*5;
    ctx.beginPath(); ctx.arc(fx,fy,cr,0,Math.PI*2);
    ctx.fillStyle='#4a1f08'; ctx.fill();
    for(let k=0;k<38;k++){
      const r=Math.sqrt(k/38)*cr*0.92;
      const a=k*2.399;
      ctx.beginPath();
      ctx.arc(fx+Math.cos(a)*r,fy+Math.sin(a)*r,0.7+hash(k)*0.9,0,Math.PI*2);
      ctx.fillStyle=k%4===0?'#fff0b0':'#ffc23c';
      ctx.fill();
    }
    return {x:fx,y:fy,open:open};
  }

  function drawParticles(wind){
    for(const q of particles){
      const alpha = clamp(q.life,0,1);
      if(q.dust){
        ctx.fillStyle='rgba(210,226,240,'+(alpha*0.42)+')';
        ctx.beginPath(); ctx.arc(q.x,q.y,q.size*0.8,0,Math.PI*2); ctx.fill();
      }else{
        ctx.save();
        ctx.translate(q.x,q.y); ctx.rotate(q.rot);
        ctx.fillStyle='rgba(255,198,86,'+alpha+')';
        ctx.beginPath(); ctx.ellipse(0,0,q.size*1.6,q.size*0.7,0,0,Math.PI*2); ctx.fill();
        ctx.restore();
      }
    }
  }

  let last = performance.now(), frames = 0, seedCount = 0;
  function draw(now){
    requestAnimationFrame(draw);
    const t = now/1000;
    const cycle = Math.floor(t/CYCLE);
    const p = (t%CYCLE)/CYCLE;
    if(cycle!==lastCycle){
      lastCycle = cycle; particles = []; seedSpawned = false;
    }
    const dt = Math.min((now-last)/1000,0.05);
    last = now;
    pointer.impulse *= 0.94;
    const wind = Math.sin(t*0.5)*18;

    drawSky(t);
    drawField(p,t);
    drawRidges(p,t);
    const flower = drawBloom(p,t);

    const release = ss(0.72,0.9,p);
    if(!seedSpawned && release>0.02 && flower){
      spawnSeeds(flower.x,flower.y,52,1);
      seedSpawned = true;
    }
    if(pointer.impulse>0.55 && flower && !pointer._burst){
      spawnSeeds(flower.x,flower.y,34,0.9);
      pointer._burst = true;
    }
    if(pointer.impulse<0.2) pointer._burst = false;
    updateParticles(dt,wind);
    drawParticles(wind);

    const fade = 1-ss(0.95,1,p);
    if(fade<1){
      ctx.globalAlpha=1-fade;
      ctx.fillStyle='#050b18';
      ctx.fillRect(0,0,W,H);
      ctx.globalAlpha=1;
    }

    const phase = p<0.16?'FIELD':p<0.38?'RIDGE':p<0.6?'BLOOM':p<0.72?'OPEN':p<0.94?'DISPERSE':'RESET';
    if(phaseEl && phaseEl.textContent!==phase) phaseEl.textContent=phase;
    seedCount = particles.filter(q=>!q.dust).length;
    if(seedsEl) seedsEl.textContent=String(seedCount).padStart(3,'0');
    frames++;
    if(fpsEl && now-lastFps>1000){
      fpsEl.textContent=Math.round(frames*1000/(now-lastFps));
      frames=0; lastFps=now;
    }
  }
  let lastFps = performance.now();
  requestAnimationFrame(draw);
})();
</script>

## Layers of the piece

<div class="grid-2">
  <div class="box">
    <h3>Point field &amp; ridge</h3>
    <p>A lattice of nodes resolves into three morphing mountain ranges.</p>
  </div>
  <div class="box">
    <h3>Drifting sky</h3>
    <p>Cloud bands, a low sun glow, and a slow field of stars pass overhead.</p>
  </div>
  <div class="box">
    <h3>The bloom</h3>
    <p>A stem grows with a swirling trail, then three rings of petals unfurl.</p>
  </div>
  <div class="box">
    <h3>Seed drift</h3>
    <p>Seeds and dust are released, carried by wind, and pulled down by gravity.</p>
  </div>
</div>
