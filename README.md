# Jogo-de-heroi-
<!DOCTYPE html><html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>Lendas da Luz</title>
  <style>
    canvas { background: #111; display: block; margin: 0 auto; border: 2px solid #fff; }
    #hud {
      color: white;
      font-family: Arial;
      text-align: center;
      margin-top: 10px;
    }
    #controls {
      position: absolute;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 10px;
    }
    .btn {
      padding: 10px 15px;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      background: #444;
      color: white;
    }
  </style>
</head>
<body>
<canvas id="gameCanvas" width="800" height="400"></canvas>
<div id="hud">
  <p>Vida: <span id="vida">100</span> | Pontos: <span id="pontos">0</span> | Fase: <span id="fase">1</span></p>
  <p><button onclick="resetGame()">Reiniciar</button> <button onclick="modoSobrevivencia()">Modo Sobrevivência</button></p>
</div>
<div id="controls">
  <button class="btn" onclick="move('left')">Esq</button>
  <button class="btn" onclick="jump()">Pulo</button>
  <button class="btn" onclick="move('right')">Dir</button>
  <button class="btn" onclick="attack()">Ataque</button>
</div><script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

let keys = {}, gravity = 1, pontos = 0, fase = 1, modoBoss = false, sobrevivencia = false;

let hero = {
  x: 100, y: 300, w: 40, h: 60, cor: 'gold', dx: 0, dy: 0,
  vida: 100, atacando: false, pulando: false
};

let inimigos = [];

function gerarInimigo(boss = false) {
  const vida = boss ? 300 : 50 + Math.random() * 50;
  inimigos.push({
    x: 800, y: 300, w: 40, h: 60, cor: boss ? 'purple' : 'crimson',
    vida: vida, maxVida: vida, boss: boss
  });
}

function salvarProgresso() {
  localStorage.setItem('jogo_heroi', JSON.stringify({vida: hero.vida, pontos, fase}));
}

function carregarProgresso() {
  const dados = JSON.parse(localStorage.getItem('jogo_heroi'));
  if (dados) {
    hero.vida = dados.vida;
    pontos = dados.pontos;
    fase = dados.fase;
  }
}

function resetGame() {
  hero.x = 100; hero.y = 300; hero.vida = 100; pontos = 0; fase = 1;
  inimigos = []; sobrevivencia = false;
  gerarInimigo();
  salvarProgresso();
}

function modoSobrevivencia() {
  resetGame();
  sobrevivencia = true;
  setInterval(() => {
    if (hero.vida > 0) gerarInimigo();
  }, 2000);
}

function move(dir) {
  hero.dx = dir === 'left' ? -5 : 5;
}

function jump() {
  if (!hero.pulando) {
    hero.dy = -15;
    hero.pulando = true;
  }
}

function attack() {
  hero.atacando = true;
  setTimeout(() => hero.atacando = false, 200);
}

function update() {
  hero.x += hero.dx;
  hero.y += hero.dy;
  hero.dy += gravity;

  if (hero.y > 300) {
    hero.y = 300;
    hero.dy = 0;
    hero.pulando = false;
  }

  for (let i = inimigos.length - 1; i >= 0; i--) {
    const e = inimigos[i];
    e.x -= 2;

    if (hero.atacando && colidiu(hero, e)) {
      e.vida -= 10;
    }

    if (colidiu(hero, e)) {
      hero.vida -= 0.5;
    }

    if (e.vida <= 0) {
      pontos += e.boss ? 100 : 10;
      inimigos.splice(i, 1);
    }
  }

  if (inimigos.length === 0 && !sobrevivencia) {
    fase++;
    if (fase % 3 === 0) gerarInimigo(true);
    else gerarInimigo();
  }

  salvarProgresso();
}

function colidiu(a, b) {
  return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Heroi
  ctx.fillStyle = hero.cor;
  ctx.fillRect(hero.x, hero.y, hero.w, hero.h);

  // Inimigos
  for (let e of inimigos) {
    ctx.fillStyle = e.cor;
    ctx.fillRect(e.x, e.y, e.w, e.h);
    ctx.fillStyle = 'red';
    ctx.fillRect(e.x, e.y - 10, e.w, 5);
    ctx.fillStyle = 'lime';
    ctx.fillRect(e.x, e.y - 10, e.w * (e.vida / e.maxVida), 5);
  }

  document.getElementById('vida').innerText = Math.max(0, Math.floor(hero.vida));
  document.getElementById('pontos').innerText = pontos;
  document.getElementById('fase').innerText = fase;

  if (hero.vida <= 0) {
    ctx.fillStyle = 'white';
    ctx.font = '30px Arial';
    ctx.fillText('Você morreu! Clique em Reiniciar.', 200, 200);
    return;
  }
}

function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

carregarProgresso();
gerarInimigo();
gameLoop();
</script></body>
</html>
