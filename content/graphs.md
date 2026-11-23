# Generative Bloom

> A living field: points resolve into a mountain ridge, sky drifts overhead, and an ancient Roman warrior steps into the frame with sword raised and shield at the ready. Move across the canvas to shift the figure; click to stir a burst of dust.

<div class="bloom-stage">
  <div class="bloom-stage__frame">
    <canvas id="graphCanvas" width="1000" height="560"></canvas>
    <div class="bloom-stage__overlay">
      <span class="bloom-stage__index" id="bloomIndex">01</span>
      <span class="bloom-stage__label" id="bloomLabel">Point field</span>
    </div>
  </div>
  <div class="bloom-stage__bar">
    <div class="bloom-steps">
      <span class="active">Field</span><span>Ridge</span><span>Warrior</span><span>Reveal</span><span>Dust</span>
    </div>
    <div class="bloom-stage__actions">
      <button type="button" id="bloomPause">Pause</button>
      <button type="button" id="bloomReset">Restart</button>
    </div>
  </div>
  <p class="bloom-stage__hint">Move across the scene to shift the figure &middot; click to stir the dust</p>
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
  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));
  const ss = (a,b,x)=>{ const t=clamp((x-a)/(b-a),0,1); return t*t*(3-2*t); };
  const hash = n=>{ const x=Math.sin(n*127.1+311.7)*43758.5453; return x-Math.floor(x); };

  const indexEl = document.getElementById('bloomIndex');
  const labelEl = document.getElementById('bloomLabel');
  const steps = Array.from(document.querySelectorAll('.bloom-steps span'));
  const pauseBtn = document.getElementById('bloomPause');
  const resetBtn = document.getElementById('bloomReset');
  const pointer = { x:W/2, y:H/2, active:false, down:false, impulse:0 };
  const PHASES = [
    { at:0,    name:'Point field' },
    { at:0.16, name:'Mountain ridge' },
    { at:0.34, name:'The warrior' },
    { at:0.6,  name:'Reveal' },
    { at:0.72, name:'Dust & dispersal' }
  ];
  let particles = [], seedSpawned = false, lastCycle = -1;
  let simTime = 0, paused = false, stepIndex = -1;

  if(pauseBtn) pauseBtn.addEventListener('click', ()=>{
    paused = !paused;
    pauseBtn.textContent = paused ? 'Resume' : 'Pause';
  });
  if(resetBtn) resetBtn.addEventListener('click', ()=>{
    simTime = 0; particles = []; seedSpawned = false; lastCycle = -1;
    paused = false;
    if(pauseBtn) pauseBtn.textContent = 'Pause';
  });

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

  const warrior = new Image();
  warrior.src = 'images/roman-warrior.svg';

  function drawWarrior(p,t){
    if(!warrior.complete || !warrior.naturalWidth) return null;
    const reveal = ss(0.34,0.6,p);
    if(reveal<=0) return null;
    const baseX = W/2;
    const baseY = H*0.9;
    const dist = pointer.active ? Math.hypot(pointer.x-baseX, pointer.y-H*0.5) : 999;
    const near = clamp(1-dist/230,0,1);
    const lean = clamp((pointer.x-baseX)/(W/2),-1,1)*near*18;
    const x = baseX+lean;
    const h = 400*reveal;
    const w = h*(800/560);
    const y = baseY-h;

    const glow = ctx.createRadialGradient(x,y+h*0.45,10,x,y+h*0.45,h*0.55);
    glow.addColorStop(0,'rgba(255,200,120,'+(0.12+near*0.1)+')');
    glow.addColorStop(1,'rgba(255,200,120,0)');
    ctx.fillStyle = glow;
    ctx.fillRect(x-h,y,h*2,h);

    ctx.globalAlpha = reveal;
    ctx.drawImage(warrior, x-w/2, y, w, h);
    ctx.globalAlpha = 1;

    return {x:x, y:y+h*0.32};
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

  let last = performance.now(), lastFps = performance.now(), frames = 0;
  function draw(now){
    requestAnimationFrame(draw);
    const dt = Math.min((now-last)/1000,0.05);
    last = now;
    if(!paused) simTime += dt;
    const t = simTime;
    const cycle = Math.floor(t/CYCLE);
    const p = (t%CYCLE)/CYCLE;
    if(cycle!==lastCycle){
      lastCycle = cycle; particles = []; seedSpawned = false;
    }
    pointer.impulse *= 0.94;
    const wind = Math.sin(t*0.5)*18;

    drawSky(t);
    drawField(p,t);
    drawRidges(p,t);
    const figure = drawWarrior(p,t);

    const release = ss(0.72,0.9,p);
    if(!seedSpawned && release>0.02 && figure){
      spawnSeeds(figure.x,figure.y,52,1);
      seedSpawned = true;
    }
    if(pointer.impulse>0.55 && figure && !pointer._burst){
      spawnSeeds(figure.x,figure.y,34,0.9);
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
    <h3>The warrior</h3>
    <p>An ancient Roman legionary emerges with a gladius raised and a scutum shield, rendered as a self-contained illustration layered into the scene.</p>
  </div>
  <div class="box">
    <h3>Seed drift</h3>
    <p>Seeds and dust are released, carried by wind, and pulled down by gravity.</p>
  </div>
</div>
