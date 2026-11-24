# Creative Mix & Sandbox

> Experiments, audio-visual synthesis, code snippets, and ideas in motion.

## Interactive Sound Synth

<div class="box" style="text-align:center;">
  <span class="eyebrow">WEB AUDIO API SYNTH</span>
  <h3>Cyber Sound Generator</h3>
  <p>Generate retro Symbian OS / Metro audio tones using native Web Audio API in your browser.</p>
  <div style="display:flex; gap:12px; justify-content:center; margin-top:16px;">
    <button class="btn btn--accent" onclick="playTone(440, 'sine')">Play 440Hz Sine</button>
    <button class="btn btn--solid" onclick="playTone(880, 'square')">Play 880Hz Retro</button>
    <button class="btn" onclick="playArpeggio()">Play Cyber Arp</button>
  </div>
</div>

## Inception — Electronic Recreation

<div class="box" style="text-align:center;">
  <span class="eyebrow">WEB AUDIO SEQUENCE</span>
  <h3>The Brass Motif</h3>
  <p>The trailer's low horn call — B♭, F, D♭, B♭ — rebuilt from saturated oscillators, sub-bass impacts, breath noise, a rising tension sweep and a synthetic hall reverb.</p>
  <div style="display:flex; gap:12px; justify-content:center; margin-top:16px; flex-wrap:wrap;">
    <button class="btn btn--accent" onclick="playInception()">Play Sequence</button>
    <button class="btn" onclick="stopInception()">Stop</button>
  </div>
  <p class="eyebrow" style="margin-top:14px;">STATUS: <span id="incStatus">READY</span></p>
</div>

<script>
function playTone(freq, type) {
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = type || 'sine';
    osc.frequency.setValueAtTime(freq, ctx.currentTime);
    gain.gain.setValueAtTime(0.15, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.5);
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.start();
    osc.stop(ctx.currentTime + 0.5);
  } catch(e){}
}

function playArpeggio() {
  const notes = [261.63, 329.63, 392.00, 523.25, 659.25, 783.99];
  notes.forEach((freq, idx) => {
    setTimeout(() => playTone(freq, 'square'), idx * 90);
  });
}

if(window.__incTimer){ clearInterval(window.__incTimer); window.__incTimer=null; }
if(!window.__incHashBound){
  window.__incHashBound = true;
  window.addEventListener('hashchange', ()=>{ if(window.__incTimer) stopInception(); });
}

function incImpulse(ctx, seconds, decay){
  const rate = ctx.sampleRate, len = Math.floor(rate*seconds);
  const buf = ctx.createBuffer(2,len,rate);
  for(let c=0;c<2;c++){
    const d = buf.getChannelData(c);
    for(let i=0;i<len;i++) d[i]=(Math.random()*2-1)*Math.pow(1-i/len,decay);
  }
  return buf;
}

function incCurve(amount){
  const n = 1024, curve = new Float32Array(n);
  for(let i=0;i<n;i++){
    const x = i*2/n-1;
    curve[i] = Math.tanh(x*amount);
  }
  return curve;
}

function incSend(node){
  node.connect(window.__incMaster);
  if(window.__incVerb) node.connect(window.__incVerb);
}

function incBreath(ctx,time,amt){
  const len = Math.floor(ctx.sampleRate*0.3);
  const buf = ctx.createBuffer(1,len,ctx.sampleRate);
  const d = buf.getChannelData(0);
  for(let i=0;i<len;i++) d[i]=(Math.random()*2-1)*(1-i/len);
  const src = ctx.createBufferSource(); src.buffer=buf;
  const f = ctx.createBiquadFilter(); f.type='bandpass'; f.frequency.value=900; f.Q.value=0.6;
  const g = ctx.createGain();
  g.gain.setValueAtTime(0.0001,time);
  g.gain.exponentialRampToValueAtTime(amt,time+0.02);
  g.gain.exponentialRampToValueAtTime(0.0001,time+0.26);
  src.connect(f); f.connect(g); incSend(g);
  src.start(time);
}

function incBrass(ctx,freq,time,dur,amt){
  const shaper = ctx.createWaveShaper();
  shaper.curve = incCurve(3.2);
  shaper.oversample = '4x';

  const filt = ctx.createBiquadFilter();
  filt.type = 'lowpass';
  filt.Q.value = 1.4;
  filt.frequency.setValueAtTime(170,time);
  filt.frequency.exponentialRampToValueAtTime(1500,time+0.16);
  filt.frequency.exponentialRampToValueAtTime(520,time+dur*0.72);
  filt.frequency.exponentialRampToValueAtTime(160,time+dur);

  const g = ctx.createGain();
  g.gain.setValueAtTime(0.0001,time);
  g.gain.exponentialRampToValueAtTime(amt,time+0.075);
  g.gain.setValueAtTime(amt*0.86,time+dur*0.55);
  g.gain.exponentialRampToValueAtTime(0.0001,time+dur);

  const mix = ctx.createGain(); mix.gain.value=0.42;
  [-11,0,11,6].forEach((det,i)=>{
    const o = ctx.createOscillator();
    o.type = i%2 ? 'sawtooth' : 'square';
    o.frequency.setValueAtTime(freq*0.93,time);
    o.frequency.exponentialRampToValueAtTime(freq,time+0.075);
    o.detune.value = det;
    o.connect(mix);
    o.start(time); o.stop(time+dur+0.06);
    if(i===0) o.onended = ()=>{ try{ mix.disconnect(); shaper.disconnect(); filt.disconnect(); g.disconnect(); }catch(e){} };
  });
  mix.connect(shaper); shaper.connect(filt); filt.connect(g); incSend(g);
  incBreath(ctx,time,0.035);
}

function incSub(ctx,freq,time,dur){
  const o = ctx.createOscillator(); o.type='sine'; o.frequency.value=freq;
  const g = ctx.createGain();
  g.gain.setValueAtTime(0.0001,time);
  g.gain.exponentialRampToValueAtTime(0.3,time+0.05);
  g.gain.exponentialRampToValueAtTime(0.0001,time+dur);
  o.connect(g); incSend(g);
  o.start(time); o.stop(time+dur+0.05);
}

function incBoom(ctx,time,amt){
  const o = ctx.createOscillator(); o.type='sine';
  o.frequency.setValueAtTime(130,time);
  o.frequency.exponentialRampToValueAtTime(34,time+0.5);
  const g = ctx.createGain();
  g.gain.setValueAtTime(amt,time);
  g.gain.exponentialRampToValueAtTime(0.0001,time+0.62);
  o.connect(g); incSend(g);
  o.start(time); o.stop(time+0.66);
}

function incRiser(ctx,time,dur){
  const len = Math.floor(ctx.sampleRate*dur);
  const buf = ctx.createBuffer(1,len,ctx.sampleRate);
  const d = buf.getChannelData(0);
  for(let i=0;i<len;i++) d[i]=Math.random()*2-1;
  const src = ctx.createBufferSource(); src.buffer=buf;
  const f = ctx.createBiquadFilter(); f.type='bandpass'; f.Q.value=1.4;
  f.frequency.setValueAtTime(260,time);
  f.frequency.exponentialRampToValueAtTime(5200,time+dur);
  const g = ctx.createGain();
  g.gain.setValueAtTime(0.0001,time);
  g.gain.exponentialRampToValueAtTime(0.1,time+dur*0.92);
  g.gain.exponentialRampToValueAtTime(0.0001,time+dur);
  src.connect(f); f.connect(g); incSend(g);
  src.start(time);
}

function playInception(){
  try{
    if(window.__incTimer){ clearInterval(window.__incTimer); window.__incTimer=null; }
    const ctx = window.__incCtx || (window.__incCtx = new (window.AudioContext||window.webkitAudioContext)());
    if(ctx.state==='suspended') ctx.resume();

    const master = ctx.createGain();
    master.gain.setValueAtTime(0.0001,ctx.currentTime);
    master.gain.exponentialRampToValueAtTime(0.72,ctx.currentTime+0.7);
    const comp = ctx.createDynamicsCompressor();
    comp.threshold.value=-16; comp.ratio.value=4;
    master.connect(comp); comp.connect(ctx.destination);
    window.__incMaster = master;

    const verb = ctx.createGain(); verb.gain.value=0.5;
    const conv = ctx.createConvolver(); conv.buffer=incImpulse(ctx,3.4,2.6);
    verb.connect(conv); conv.connect(master);
    window.__incVerb = verb;

    const motif = [
      {t:0.0, f:58.27, d:1.85},
      {t:1.95,f:87.31, d:0.95},
      {t:2.98,f:69.30, d:0.95},
      {t:4.02,f:58.27, d:2.35}
    ];
    const CYCLE = 7.7;
    let start = ctx.currentTime+0.15;

    function schedule(){
      while(start < ctx.currentTime+0.6){
        for(const n of motif){
          incBrass(ctx,n.f,start+n.t,n.d,0.19);
          incSub(ctx,n.f/2,start+n.t,n.d*0.92);
          incBoom(ctx,start+n.t,0.3);
        }
        incRiser(ctx,start+6.45,1.15);
        start += CYCLE;
      }
    }
    schedule();
    window.__incTimer = setInterval(schedule,90);
    const st = document.getElementById('incStatus');
    if(st) st.textContent='PLAYING';
  }catch(e){
    const st = document.getElementById('incStatus');
    if(st) st.textContent='AUDIO UNAVAILABLE';
  }
}

function stopInception(){
  if(window.__incTimer){ clearInterval(window.__incTimer); window.__incTimer=null; }
  const ctx = window.__incCtx;
  if(ctx && window.__incMaster){
    const g = window.__incMaster.gain;
    g.cancelScheduledValues(ctx.currentTime);
    g.setValueAtTime(Math.max(g.value,0.0001),ctx.currentTime);
    g.exponentialRampToValueAtTime(0.0001,ctx.currentTime+0.5);
  }
  const st = document.getElementById('incStatus');
  if(st) st.textContent='READY';
}
</script>

## Sandbox Highlights

- **Web Audio API** synth built zero-dependency using native browser APIs
- **Saturated brass engine** with waveshaping, pitch scoops, sub hits and generated convolution reverb
- **Markdown Driven Routing** without any heavy npm build tools
