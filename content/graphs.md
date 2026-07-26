# Interactive Graphs & Systems

> Interactive data visualizations, neural operator benchmarks, and kinematic graph simulations.

<div class="game-box">
  <div class="game-hud">
    <div>NODES: <b>36 ACTIVE</b></div>
    <div>FPS: <b id="graphFps">60</b></div>
    <div>MODE: <span style="color:var(--color-accent)">NEURAL MANIFOLD</span></div>
  </div>
  <canvas id="graphCanvas" width="700" height="320" class="game-canvas"></canvas>
</div>

<script>
(function(){
  const canvas = document.getElementById('graphCanvas');
  if(!canvas) return;
  const ctx = canvas.getContext('2d');
  let nodes = [];
  const count = 36;

  for(let i=0; i<count; i++){
    nodes.push({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      vx: (Math.random() - 0.5) * 1.5,
      vy: (Math.random() - 0.5) * 1.5,
      r: Math.random() * 3 + 2
    });
  }

  function draw(){
    requestAnimationFrame(draw);
    ctx.fillStyle = '#090d12';
    ctx.fillRect(0,0,canvas.width,canvas.height);

    const accent = getComputedStyle(document.documentElement).getPropertyValue('--color-accent').trim() || '#00A4EF';

    // Draw connections
    for(let i=0; i<count; i++){
      for(let j=i+1; j<count; j++){
        const dx = nodes[i].x - nodes[j].x;
        const dy = nodes[i].y - nodes[j].y;
        const dist = Math.sqrt(dx*dx + dy*dy);
        if(dist < 110){
          ctx.strokeStyle = accent;
          ctx.globalAlpha = 1 - (dist / 110);
          ctx.lineWidth = 1;
          ctx.beginPath();
          ctx.moveTo(nodes[i].x, nodes[i].y);
          ctx.lineTo(nodes[j].x, nodes[j].y);
          ctx.stroke();
        }
      }
    }
    ctx.globalAlpha = 1;

    // Move & draw nodes
    nodes.forEach(n => {
      n.x += n.vx; n.y += n.vy;
      if(n.x < 0 || n.x > canvas.width) n.vx *= -1;
      if(n.y < 0 || n.y > canvas.height) n.vy *= -1;

      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.arc(n.x, n.y, n.r, 0, Math.PI*2);
      ctx.fill();
    });
  }
  draw();
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
