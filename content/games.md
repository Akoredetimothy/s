# Cyber Arcade Games

> **Yes! GitHub Pages natively supports HTML5 & Canvas Web Games.** Below is a built-in retro Cyber-Snake arcade game inspired by Nokia 3310 and Metro Symbian OS.

<div class="game-box">
  <div class="game-hud">
    <div>SCORE: <b id="gameScore" style="color:var(--color-accent)">0</b></div>
    <div>HIGH SCORE: <b id="gameHiScore">0</b></div>
    <div>STATUS: <span id="gameStatus" style="color:var(--color-text-dim)">PRESS START</span></div>
  </div>
  
  <canvas id="snakeCanvas" width="400" height="300" class="game-canvas"></canvas>

  <div style="display:flex; gap:12px; justify-content:center; margin-bottom:16px;">
    <button class="btn btn--accent" id="btnStartGame">START GAME</button>
    <button class="btn" id="btnPauseGame">PAUSE</button>
  </div>

  <div style="font-family:var(--font-mono); font-size:.78rem; color:var(--color-text-dim);">
    Use <b>Arrow Keys</b> or <b>W A S D</b> to steer the snake. On mobile, tap controls below:
  </div>

  <div style="display:grid; grid-template-columns:repeat(3, 50px); gap:6px; justify-content:center; margin-top:12px;">
    <div></div><button class="btn" id="btnUp" style="padding:10px">▲</button><div></div>
    <button class="btn" id="btnLeft" style="padding:10px">◄</button>
    <button class="btn" id="btnDown" style="padding:10px">▼</button>
    <button class="btn" id="btnRight" style="padding:10px">►</button>
  </div>
</div>

<script>
(function(){
  const canvas = document.getElementById('snakeCanvas');
  if(!canvas) return;
  const ctx = canvas.getContext('2d');
  const scoreEl = document.getElementById('gameScore');
  const hiScoreEl = document.getElementById('gameHiScore');
  const statusEl = document.getElementById('gameStatus');

  const grid = 15;
  let count = 0;
  let score = 0;
  let hiScore = parseInt(localStorage.getItem('cyber_snake_hi') || '0', 10);
  hiScoreEl.textContent = hiScore;

  let snake = { x: 150, y: 150, dx: grid, dy: 0, cells: [], maxCells: 4 };
  let food = { x: 240, y: 150 };
  let running = false;
  let animId = null;

  function getRandomInt(min, max) {
    return Math.floor(Math.random() * (max - min)) + min;
  }

  function resetGame() {
    score = 0;
    scoreEl.textContent = score;
    snake.x = 150;
    snake.y = 150;
    snake.cells = [];
    snake.maxCells = 4;
    snake.dx = grid;
    snake.dy = 0;
    food.x = getRandomInt(0, canvas.width / grid) * grid;
    food.y = getRandomInt(0, canvas.height / grid) * grid;
  }

  function loop() {
    animId = requestAnimationFrame(loop);
    if (!running) return;
    if (++count < 6) return; // 10 FPS retro feel
    count = 0;

    ctx.fillStyle = '#0a0a0a';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Draw grid lines
    ctx.strokeStyle = '#181818';
    ctx.lineWidth = 1;
    for(let x=0; x<canvas.width; x+=grid){
      ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,canvas.height); ctx.stroke();
    }
    for(let y=0; y<canvas.height; y+=grid){
      ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(canvas.width,y); ctx.stroke();
    }

    snake.x += snake.dx;
    snake.y += snake.dy;

    // Wrap around screen edges
    if (snake.x < 0) snake.x = canvas.width - grid;
    else if (snake.x >= canvas.width) snake.x = 0;
    if (snake.y < 0) snake.y = canvas.height - grid;
    else if (snake.y >= canvas.height) snake.y = 0;

    snake.cells.unshift({ x: snake.x, y: snake.y });
    if (snake.cells.length > snake.maxCells) snake.cells.pop();

    // Draw Food
    const accentColor = getComputedStyle(document.documentElement).getPropertyValue('--color-accent').trim() || '#00A4EF';
    ctx.fillStyle = '#F25022';
    ctx.shadowBlur = 10; ctx.shadowColor = '#F25022';
    ctx.fillRect(food.x + 1, food.y + 1, grid - 2, grid - 2);

    // Draw Snake
    ctx.fillStyle = accentColor;
    ctx.shadowBlur = 6; ctx.shadowColor = accentColor;
    snake.cells.forEach((cell, index) => {
      ctx.fillRect(cell.x + 1, cell.y + 1, grid - 2, grid - 2);

      // Check collision with food
      if (cell.x === food.x && cell.y === food.y) {
        snake.maxCells++;
        score += 10;
        scoreEl.textContent = score;
        if(score > hiScore){
          hiScore = score;
          hiScoreEl.textContent = hiScore;
          localStorage.setItem('cyber_snake_hi', hiScore);
        }
        food.x = getRandomInt(0, canvas.width / grid) * grid;
        food.y = getRandomInt(0, canvas.height / grid) * grid;
      }

      // Check self-collision
      for (let i = index + 1; i < snake.cells.length; i++) {
        if (cell.x === snake.cells[i].x && cell.y === snake.cells[i].y) {
          statusEl.textContent = 'GAME OVER';
          statusEl.style.color = '#F25022';
          running = false;
        }
      }
    });
    ctx.shadowBlur = 0;
  }

  document.addEventListener('keydown', function(e) {
    if(!running) return;
    if ((e.key === 'ArrowLeft' || e.key === 'a') && snake.dx === 0) { snake.dx = -grid; snake.dy = 0; }
    else if ((e.key === 'ArrowUp' || e.key === 'w') && snake.dy === 0) { snake.dy = -grid; snake.dx = 0; }
    else if ((e.key === 'ArrowRight' || e.key === 'd') && snake.dx === 0) { snake.dx = grid; snake.dy = 0; }
    else if ((e.key === 'ArrowDown' || e.key === 's') && snake.dy === 0) { snake.dy = grid; snake.dx = 0; }
  });

  document.getElementById('btnStartGame').onclick = function(){
    resetGame();
    running = true;
    statusEl.textContent = 'PLAYING';
    statusEl.style.color = '#7FBA00';
  };
  document.getElementById('btnPauseGame').onclick = function(){
    running = !running;
    statusEl.textContent = running ? 'PLAYING' : 'PAUSED';
    statusEl.style.color = running ? '#7FBA00' : '#FFB900';
  };

  document.getElementById('btnUp').onclick = function(){ if(snake.dy === 0){ snake.dy = -grid; snake.dx = 0; } };
  document.getElementById('btnDown').onclick = function(){ if(snake.dy === 0){ snake.dy = grid; snake.dx = 0; } };
  document.getElementById('btnLeft').onclick = function(){ if(snake.dx === 0){ snake.dx = -grid; snake.dy = 0; } };
  document.getElementById('btnRight').onclick = function(){ if(snake.dx === 0){ snake.dx = grid; snake.dx = 0; } };

  resetGame();
  loop();
})();
</script>
