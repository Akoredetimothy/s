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
  <p>A low synthesized horn, sub bass, a repeating arpeggio and noise pulses — an electronic reworking of the familiar Inception-style swell, built only from oscillators.</p>
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

if(window.__incTimer){ clearInterval(window.__incTimer); window.__incTimer = null; }
if(!window.__incHashBound){
  window.__incHashBound = true;
  window.addEventListener('hashchange', ()=>{ if(window.__incTimer) stopInception(); });
}

function incOut(node){
  node.connect(window.__incMaster);
  if(window.__incDelay) node.connect(window.__incDelay);
}

function incHorn(ctx, freq, time, dur, amt){
  const filt = ctx.createBiquadFilter();
  filt.type = 'lowpass';
  filt.Q.value = 3;
  filt.frequency.setValueAtTime(240, time);
  filt.frequency.exponentialRampToValueAtTime(1650, time+0.14);
  filt.frequency.exponentialRampToValueAtTime(420, time+dur);
  const g = ctx.createGain();
  g.gain.setValueAtTime(0.0001, time);
  g.gain.exponentialRampToValueAtTime(amt, time+0.11);
  g.gain.setValueAtTime(amt, time+dur*0.62);
  g.gain.exponentialRampToValueAtTime(0.0001, time+dur);
  [-9,0,9].forEach(d=>{
    const o = ctx.createOscillator();
    o.type = 'sawtooth';
    o.frequency.setValueAtTime(freq, time);
    o.detune.value = d;
    o.connect(filt);
    o.start(time); o.stop(time+dur+0.05);
  });
  filt.connect(g); incOut(g);
}

function incSub(ctx, freq, time, dur){
  const o = ctx.createOscillator(); o.type='sine'; o.frequency.value=freq;
  const g = ctx.createGain();
  g.gain.setValueAtTime(0.0001,time);
  g.gain.exponentialRampToValueAtTime(0.26,time+0.04);
  g.gain.exponentialRampToValueAtTime(0.0001,time+dur);
  o.connect(g); incOut(g);
  o.start(time); o.stop(time+dur+0.05);
}

function incPluck(ctx, freq, time, dur){
  const o = ctx.createOscillator(); o.type='triangle'; o.frequency.value=freq;
  const o2 = ctx.createOscillator(); o2.type='sine'; o2.frequency.value=freq*2;
  const g = ctx.createGain();
  g.gain.setValueAtTime(0.0001,time);
  g.gain.exponentialRampToValueAtTime(0.085,time+0.012);
  g.gain.exponentialRampToValueAtTime(0.0001,time+dur);
  o.connect(g); o2.connect(g); incOut(g);
  o.start(time); o2.start(time);
  o.stop(time+dur+0.02); o2.stop(time+dur+0.02);
}

function incNoise(ctx, time, amount){
  const len = Math.floor(ctx.sampleRate*0.22);
  const buf = ctx.createBuffer(1,len,ctx.sampleRate);
  const data = buf.getChannelData(0);
  for(let i=0;i<len;i++) data[i]=(Math.random()*2-1)*(1-i/len);
  const src = ctx.createBufferSource(); src.buffer=buf;
  const f = ctx.createBiquadFilter(); f.type='bandpass'; f.frequency.value=1400; f.Q.value=0.7;
  const g = ctx.createGain(); g.gain.value=amount;
  src.connect(f); f.connect(g); incOut(g);
  src.start(time);
}

function playInception(){
  try{
    if(window.__incTimer){ clearInterval(window.__incTimer); window.__incTimer=null; }
    const ctx = window.__incCtx || (window.__incCtx = new (window.AudioContext||window.webkitAudioContext)());
    if(ctx.state==='suspended') ctx.resume();

    const master = ctx.createGain();
    master.gain.setValueAtTime(0.0001,ctx.currentTime);
    master.gain.exponentialRampToValueAtTime(0.6,ctx.currentTime+0.9);
    const comp = ctx.createDynamicsCompressor();
    master.connect(comp); comp.connect(ctx.destination);
    window.__incMaster = master;

    const delay = ctx.createDelay(1); delay.delayTime.value=0.31;
    const fb = ctx.createGain(); fb.gain.value=0.3;
    const wet = ctx.createGain(); wet.gain.value=0.22;
    delay.connect(fb); fb.connect(delay); delay.connect(wet); wet.connect(master);
    window.__incDelay = delay;

    const hornSeq = [
      {s:0, f:58.27, d:1.5}, {s:3, f:87.31, d:1.0}, {s:5, f:69.30, d:1.0},
      {s:8, f:58.27, d:1.6}, {s:11,f:87.31, d:1.0}, {s:13,f:69.30, d:1.2}
    ];
    const arp = [233.08, 277.18, 349.23, 466.16, 349.23, 277.18, 233.08, 311.13];
    const beat = 60/84/2;
    let step = 0;
    let next = ctx.currentTime+0.12;

    function schedule(){
      while(next < ctx.currentTime+0.4){
        const s = step%16;
        for(const h of hornSeq) if(h.s===s) incHorn(ctx,h.f,next,h.d,0.16);
        if(s===0 || s===8) incSub(ctx,29.14,next,1.2);
        if(s%2===0) incPluck(ctx,arp[(s/2)%arp.length],next,beat*1.8);
        if(s===0 || s===6 || s===8 || s===14) incNoise(ctx,next,0.045);
        next += beat;
        step++;
      }
    }
    schedule();
    window.__incTimer = setInterval(schedule,70);
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
    g.exponentialRampToValueAtTime(0.0001,ctx.currentTime+0.45);
  }
  const st = document.getElementById('incStatus');
  if(st) st.textContent='READY';
}
</script>

## Sandbox Highlights

- **Web Audio API** synth built zero-dependency using native browser APIs
- **Oscillator-based brass motif** with synthesized sub bass, arpeggio and filtered noise pulses
- **Markdown Driven Routing** without any heavy npm build tools
