# Creative Mix & Sandbox

> Experiments, audio-visual synthesis, code snippets, and ideas in motion.

## Interactive Sound Synth

<div class="box" style="text-align:center;">
  <span class="eyebrow">// WEB AUDIO API SYNTH</span>
  <h3>Cyber Sound Generator</h3>
  <p>Generate retro Symbian OS / Metro audio tones using native Web Audio API in your browser.</p>
  <div style="display:flex; gap:12px; justify-content:center; margin-top:16px;">
    <button class="btn btn--accent" onclick="playTone(440, 'sine')">Play 440Hz Sine</button>
    <button class="btn btn--solid" onclick="playTone(880, 'square')">Play 880Hz Retro</button>
    <button class="btn" onclick="playArpeggio()">Play Cyber Arp</button>
  </div>
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
</script>

## Sandbox Highlights

- **Web Audio API** synth built zero-dependency using native browser APIs
- **Canvas 2D Acceleration** for smooth 60 FPS renders
- **Markdown Driven Routing** without any heavy npm build tools
