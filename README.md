<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MiniTerraria | 2D Sandbox Survival</title>
    <meta name="description" content="A browser-based 2D survival game inspired by Terraria. Mine, craft, and fight bosses!">
    
    <style>
        :root { --primary: #2ecc71; --accent: #f1c40f; --danger: #ff4757; }
        body { margin: 0; overflow: hidden; background: #0c0c1a; font-family: 'Segoe UI', sans-serif; color: white; touch-action: none; }
        canvas { display: block; cursor: crosshair; }
        
        /* UI Panels */
        .ui-panel { position: absolute; background: rgba(0,0,0,0.85); border: 1px solid #555; padding: 12px; border-radius: 8px; pointer-events: none; z-index: 10; backdrop-filter: blur(4px); }
        .stats { top: 10px; left: 10px; width: 190px; }
        .crafting { top: 160px; right: 10px; width: 160px; pointer-events: auto; }
        .map-box { top: 10px; right: 10px; border: 2px solid #444; }
        
        /* Menu Overlay */
        #menu-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: radial-gradient(circle, #2c3e50, #000); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; }
        #menu-overlay h1 { font-size: 50px; color: var(--primary); text-shadow: 4px 4px #000; margin-bottom: 10px; letter-spacing: 2px; }
        .menu-btn { background: #27ae60; color: white; border: none; padding: 15px 40px; margin: 10px; cursor: pointer; font-size: 20px; border-radius: 8px; transition: 0.2s; pointer-events: auto; font-weight: bold; box-shadow: 0 4px #1e8449; }
        .menu-btn:active { transform: translateY(2px); box-shadow: 0 2px #1e8449; }

        /* Hotbar */
        .hotbar { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); display: flex; gap: 5px; background: rgba(0,0,0,0.8); padding: 8px; border-radius: 10px; pointer-events: auto; z-index: 10; border: 1px solid #444; }
        .slot { width: 44px; height: 44px; border: 2px solid #444; display: flex; align-items: center; justify-content: center; background: #222; position: relative; cursor: pointer; font-size: 20px; border-radius: 4px; }
        .active { border-color: var(--accent); background: #555; box-shadow: 0 0 15px rgba(241, 196, 15, 0.4); }
        .count { position: absolute; bottom: 1px; right: 3px; font-weight: bold; font-size: 11px; color: #fff; text-shadow: 1px 1px #000; }
        
        #hp-bar { width: 100%; height: 12px; background: #333; margin-top: 5px; border-radius: 6px; overflow: hidden; border: 1px solid #000; }
        #hp-fill { height: 100%; background: var(--danger); width: 100%; transition: width 0.3s; }
        .craft-btn { background: #2980b9; color: white; border: none; padding: 8px; margin: 3px 0; cursor: pointer; width: 100%; font-size: 12px; border-radius: 4px; font-weight: bold; }
        .save-btn { background: #8e44ad; }

        /* Mobile Controls */
        .mobile-controls { position: absolute; bottom: 30px; left: 20px; display: flex; gap: 15px; z-index: 20; }
        .m-btn { width: 75px; height: 75px; background: rgba(255,255,255,0.1); border: 2px solid rgba(255,255,255,0.4); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 30px; user-select: none; -webkit-tap-highlight-color: transparent; }
        .jump-btn { position: absolute; bottom: 30px; right: 20px; width: 90px; height: 90px; background: rgba(46, 204, 113, 0.2); font-weight: bold; font-size: 16px; border: 2px solid var(--primary); }

        footer { position: absolute; bottom: 5px; width: 100%; text-align: center; font-size: 10px; color: rgba(255,255,255,0.3); pointer-events: none; }

        @media (min-width: 1025px) { .mobile-controls, .jump-btn { display: none !important; } }
        @media (max-width: 768px) {
            .hotbar { bottom: 120px; scale: 0.9; }
            .crafting { top: auto; bottom: 190px; right: 10px; scale: 0.8; }
            .stats { scale: 0.8; top: 5px; left: 5px; }
            .map-box { scale: 0.8; top: 5px; right: 5px; }
        }
    </style>
</head>
<body>

<div id="menu-overlay">
    <h1>MINI TERRARIA</h1>
    <button class="menu-btn" onclick="startGame()">START ADVENTURE</button>
    <button class="menu-btn save-btn" onclick="loadGame()">LOAD WORLD</button>
    <div style="margin-top: 20px; color: #888; font-size: 14px;">WASD to Move • Click to Mine • 1-6 Tools</div>
</div>

<div class="mobile-controls">
    <div class="m-btn" id="btn-left">◀</div>
    <div class="m-btn" id="btn-right">▶</div>
</div>
<div class="m-btn jump-btn" id="btn-jump">JUMP</div>

<div class="ui-panel stats">
    <div><b>SURVIVOR</b> HP: <span id="hp-val">100</span></div>
    <div id="hp-bar"><div id="hp-fill"></div></div>
    <div style="font-size: 11px; margin-top:5px; color:#aaa;" id="armor-txt">Armor: None</div>
    <hr style="border: 0; border-top: 1px solid #444; margin: 8px 0;">
    <button class="craft-btn save-btn" onclick="saveGame()">💾 SAVE WORLD</button>
    <div id="boss-log" style="color: var(--danger); font-weight: bold; display: none; text-align: center; margin-top: 10px; border: 1px solid var(--danger); padding: 4px;">BOSS: <span id="boss-hp">500</span></div>
</div>

<div class="ui-panel map-box">
    <canvas id="minimap" width="100" height="60"></canvas>
</div>

<div class="ui-panel crafting">
    <b>CRAFTING</b>
    <button class="craft-btn" onclick="craft('heal')">Potion (10 Grass)</button>
    <button class="craft-btn" onclick="craft('bullets')">Ammo x20 (2 Iron)</button>
    <button class="craft-btn" onclick="craft('armor')">Iron Plate (10 Iron)</button>
</div>

<div class="hotbar">
    <div id="s1" class="slot active" onclick="setSlot(1)">⛏️</div>
    <div id="s2" class="slot" onclick="setSlot(2)">⚔️</div>
    <div id="s3" class="slot" onclick="setSlot(3)">🏹<div id="cnt-arr" class="count">0</div></div>
    <div id="s6" class="slot" onclick="setSlot(6)">🔫<div id="cnt-bul" class="count">0</div></div>
    <div id="s4" class="slot" onclick="setSlot(4)">🟫<div id="cnt-dirt" class="count">0</div></div>
    <div id="s5" class="slot" onclick="setSlot(5)">⬜<div id="cnt-stone" class="count">0</div></div>
</div>

<footer>&copy; 2024 MiniTerraria.com - Built with JavaScript</footer>

<canvas id="gameCanvas"></canvas>

<script>
/* CORE LOGIC */
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d', { alpha: false });
const mCanvas = document.getElementById('minimap');
const mctx = mCanvas.getContext('2d');

const TILE = 32, ROWS = 60, COLS = 100;
const EMPTY = 0, GRASS = 1, DIRT = 2, STONE = 3, IRON = 4;
const COLORS = { [GRASS]:'#2ecc71', [DIRT]:'#795548', [STONE]:'#95a5a6', [IRON]:'#dcdde1' };

let world = [], keys = {}, selectedTool = 1, camX = 0, camY = 0, gameTime = 0;
let projs = [], boss = null, flash = 0, isRunning = false;

const player = { 
    x: 0, y: 0, w: 20, h: 32, vx: 0, vy: 0, hp: 100, 
    grounded: false, iFrames: 0, armor: 0,
    inv: { [DIRT]: 0, [STONE]: 0, [GRASS]: 0, [IRON]: 0, arrows: 15, bullets: 10 }
};

/* FUNCTIONS */
function startGame() {
    document.getElementById('menu-overlay').style.display = 'none';
    if(!isRunning) {
        initWorld();
        spawnPlayer();
        isRunning = true;
        requestAnimationFrame(gameLoop);
    }
}

function initWorld() {
    world = [];
    for (let r = 0; r < ROWS; r++) {
        world[r] = [];
        for (let c = 0; c < COLS; c++) {
            if (r < 25) world[r][c] = EMPTY;
            else if (r === 25) world[r][c] = GRASS;
            else world[r][c] = (r < 35 && Math.random() > 0.05) ? DIRT : (Math.random() < 0.04) ? IRON : STONE;
        }
    }
}

function spawnPlayer() {
    player.x = (COLS / 2) * TILE;
    for (let r = 0; r < ROWS; r++) {
        if (world[r] && world[r][50] !== EMPTY) { player.y = (r - 2) * TILE; break; }
    }
}

function saveGame() {
    const saveData = { world, player, gameTime };
    localStorage.setItem('miniterraria_save', JSON.stringify(saveData));
    alert("Progress Saved to Browser!");
}

function loadGame() {
    const data = localStorage.getItem('miniterraria_save');
    if (!data) return alert("No save found!");
    const parsed = JSON.parse(data);
    world = parsed.world;
    Object.assign(player, parsed.player);
    gameTime = parsed.gameTime;
    startGame();
}

function collide(ent, axis) {
    let sC = Math.floor(ent.x / TILE) - 1, eC = Math.floor((ent.x + ent.w) / TILE) + 1;
    let sR = Math.floor(ent.y / TILE) - 1, eR = Math.floor((ent.y + ent.h) / TILE) + 1;
    for (let r = sR; r <= eR; r++) {
        for (let c = sC; c <= eC; c++) {
            if (world[r] && world[r][c] !== EMPTY) {
                let tX = c * TILE, tY = r * TILE;
                if (ent.x < tX + TILE && ent.x + ent.w > tX && ent.y < tY + TILE && ent.y + ent.h > tY) {
                    if (axis === 'y') {
                        if (ent.vy > 0) { ent.y = tY - ent.h; ent.vy = 0; ent.grounded = true; }
                        else if (ent.vy < 0) { ent.y = tY + TILE; ent.vy = 0; }
                    } else {
                        if (ent.vx > 0) ent.x = tX - ent.w;
                        if (ent.vx < 0) ent.x = tX + TILE;
                    }
                }
            }
        }
    }
}

function update() {
    if (!isRunning) return;
    gameTime++;
    player.vx = (keys['KeyD'] || keys['ArrowRight']) ? 4.5 : (keys['KeyA'] || keys['ArrowLeft']) ? -4.5 : 0;
    if ((keys['KeyW'] || keys['Space'] || keys['ArrowUp']) && player.grounded) { player.vy = -11; player.grounded = false; }
    player.vy += 0.55;
    player.y += player.vy; player.grounded = false; collide(player, 'y');
    player.x += player.vx; collide(player, 'x');
    
    if (player.iFrames > 0) player.iFrames--;
    if (flash > 0) flash--;

    // Boss Spawn
    if (player.inv[STONE] >= 25 && !boss) {
        boss = { x: player.x, y: player.y - 400, hp: 600, state: 'HOVER', timer: 0, vx: 0, vy: 0 };
        document.getElementById('boss-log').style.display = 'block';
    }

    if (boss) {
        boss.timer++;
        if (boss.state === 'HOVER') {
            boss.vx = (player.x - boss.x) * 0.03; boss.vy = (player.y - 250 - boss.y) * 0.03;
            if (boss.timer > 100) { boss.state = 'CHARGE'; boss.timer = 0; }
        } else {
            let a = Math.atan2(player.y - (boss.y+40), player.x - (boss.x+40));
            boss.vx = Math.cos(a) * 11; boss.vy = Math.sin(a) * 11;
            if (boss.timer > 45) { boss.state = 'HOVER'; boss.timer = 0; }
        }
        boss.x += boss.vx; boss.y += boss.vy;
        if (Math.abs(player.x - (boss.x+40)) < 45 && Math.abs(player.y - (boss.y+40)) < 45 && player.iFrames === 0) {
            player.hp -= 25 * (1 - player.armor); player.iFrames = 40;
        }
        if (boss.hp <= 0) { boss = null; document.getElementById('boss-log').style.display = 'none'; }
    }

    camX += (player.x - canvas.width/2 - camX) * 0.1;
    camY += (player.y - canvas.height/2 - camY) * 0.1;

    projs.forEach((p, i) => {
        p.x += p.vx; p.y += p.vy;
        let r = Math.floor(p.y/TILE), c = Math.floor(p.x/TILE);
        if (world[r] && world[r][c] !== EMPTY) projs.splice(i, 1);
        if (boss && Math.abs(p.x-(boss.x+40))<40 && Math.abs(p.y-(boss.y+40))<40) { boss.hp-=p.dmg; projs.splice(i,1); }
    });

    if (player.hp <= 0 || player.y > ROWS * TILE) location.reload();
}

function draw() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    ctx.fillStyle = (gameTime % 6000 > 3000) ? '#0c0c1a' : '#87CEEB';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.save();
    ctx.translate(-camX, -camY);

    // Render Blocks
    for (let r = 0; r < ROWS; r++) {
        for (let c = 0; c < COLS; c++) {
            if (world[r][c] !== EMPTY) {
                // Optimization: Only draw visible tiles
                if(c*TILE > camX-TILE && c*TILE < camX+canvas.width && r*TILE > camY-TILE && r*TILE < camY+canvas.height) {
                    ctx.fillStyle = COLORS[world[r][c]];
                    ctx.fillRect(c * TILE, r * TILE, TILE, TILE);
                }
            }
        }
    }

    projs.forEach(p => { ctx.fillStyle = p.c; ctx.fillRect(p.x, p.y, 8, 2); });

    if (boss) {
        ctx.fillStyle = '#ff4757'; ctx.beginPath(); ctx.arc(boss.x+40, boss.y+40, 40, 0, Math.PI*2); ctx.fill();
        ctx.fillStyle = 'white'; ctx.beginPath(); ctx.arc(boss.x+40, boss.y+40, 20, 0, Math.PI*2); ctx.fill();
    }

    ctx.fillStyle = (player.iFrames % 10 > 5) ? 'transparent' : '#e74c3c';
    ctx.fillRect(player.x, player.y, player.w, player.h);
    if (flash > 0) { ctx.fillStyle = 'yellow'; ctx.beginPath(); ctx.arc(player.x+10, player.y+10, 20, 0, Math.PI*2); ctx.fill(); }

    ctx.restore();
    drawMinimap();
    updateUI();
}

function drawMinimap() {
    mctx.fillStyle = '#111'; mctx.fillRect(0, 0, 100, 60);
    for (let r = 0; r < ROWS; r++) {
        for (let c = 0; c < COLS; c++) {
            if (world[r][c] !== EMPTY) { mctx.fillStyle = COLORS[world[r][c]]; mctx.fillRect(c, r, 1, 1); }
        }
    }
    mctx.fillStyle = "white"; mctx.fillRect(player.x/TILE, player.y/TILE, 2, 2);
    if (boss) { mctx.fillStyle = "red"; mctx.fillRect(boss.x/TILE, boss.y/TILE, 2, 2); }
}

function updateUI() {
    document.getElementById('hp-fill').style.width = player.hp + '%';
    document.getElementById('hp-val').innerText = Math.max(0, Math.round(player.hp));
    document.getElementById('cnt-arr').innerText = player.inv.arrows;
    document.getElementById('cnt-bul').innerText = player.inv.bullets;
    document.getElementById('cnt-dirt').innerText = player.inv[DIRT];
    document.getElementById('cnt-stone').innerText = player.inv[STONE];
    if (boss) document.getElementById('boss-hp').innerText = Math.floor(boss.hp);
}

function craft(type) {
    if (type === 'heal' && player.inv[GRASS] >= 10) { player.inv[GRASS] -= 10; player.hp = Math.min(100, player.hp + 50); }
    if (type === 'bullets' && player.inv[IRON] >= 2) { player.inv[IRON] -= 2; player.inv.bullets += 20; }
    if (type === 'armor' && player.inv[IRON] >= 10) { player.inv[IRON] -= 10; player.armor = 0.5; document.getElementById('armor-txt').innerText = "Armor: Iron (50%)"; }
}

function setSlot(n) { 
    selectedTool = n; 
    for(let i=1; i<=6; i++){ 
        let s = document.getElementById('s'+i); 
        if(s) s.className = (i === n) ? 'slot active' : 'slot';
    } 
}

function handleInteraction(clientX, clientY) {
    if (!isRunning) return;
    const rect = canvas.getBoundingClientRect();
    const mX = (clientX - rect.left) + camX, mY = (clientY - rect.top) + camY;
    const c = Math.floor(mX/TILE), r = Math.floor(mY/TILE);

    if (selectedTool === 1 && world[r] && world[r][c]) { player.inv[world[r][c]]++; world[r][c] = 0; }
    else if (selectedTool === 2) { if (boss && Math.abs(mX-(boss.x+40))<70 && Math.abs(mY-(boss.y+40))<70) boss.hp -= 12; }
    else if (selectedTool === 3 && player.inv.arrows > 0) {
        let a = Math.atan2(mY - player.y, mX - player.x);
        projs.push({ x: player.x, y: player.y, vx: Math.cos(a)*12, vy: Math.sin(a)*12, dmg: 18, c: '#fff' });
        player.inv.arrows--;
    }
    else if (selectedTool === 6 && player.inv.bullets > 0) {
        let a = Math.atan2(mY - player.y, mX - player.x);
        projs.push({ x: player.x, y: player.y, vx: Math.cos(a)*25, vy: Math.sin(a)*25, dmg: 35, c: '#f1c40f' });
        player.inv.bullets--; flash = 3;
    }
    else if (selectedTool >= 4 && world[r] && world[r][c] === 0) {
        let t = selectedTool === 4 ? DIRT : STONE;
        if (player.inv[t] > 0) { world[r][c] = t; player.inv[t]--; }
    }
}

/* EVENT LISTENERS */
canvas.addEventListener('mousedown', e => handleInteraction(e.clientX, e.clientY));
canvas.addEventListener('touchstart', e => { e.preventDefault(); handleInteraction(e.touches[0].clientX, e.touches[0].clientY); }, { passive: false });

window.addEventListener('keydown', e => { 
    keys[e.code] = true; 
    if (['1','2','3','4','5','6'].includes(e.key)) setSlot(parseInt(e.key)); 
});
window.addEventListener('keyup', e => keys[e.code] = false);

const bindBtn = (id, keyCode) => {
    const btn = document.getElementById(id);
    btn.addEventListener('touchstart', (e) => { e.preventDefault(); keys[keyCode] = true; }, { passive: false });
    btn.addEventListener('touchend', (e) => { e.preventDefault(); keys[keyCode] = false; }, { passive: false });
};
bindBtn('btn-left', 'KeyA');
bindBtn('btn-right', 'KeyD');
bindBtn('btn-jump', 'Space');

function gameLoop() { update(); draw(); requestAnimationFrame(gameLoop); }
</script>
</body>
</html>
