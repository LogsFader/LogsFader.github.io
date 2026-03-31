<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Terraria: Survival Web Mobile</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; color: white; touch-action: none; }
        canvas { display: block; cursor: crosshair; }
        .ui-panel { position: absolute; background: rgba(0,0,0,0.85); border: 1px solid #555; padding: 12px; border-radius: 8px; pointer-events: none; z-index: 10; }
        .stats { top: 10px; left: 10px; width: 190px; }
        .crafting { top: 160px; right: 10px; width: 160px; pointer-events: auto; }
        .map-box { top: 10px; right: 10px; border: 2px solid #444; }
        #menu-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: radial-gradient(circle, #2c3e50, #000); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; }
        #menu-overlay h1 { font-size: 40px; color: #2ecc71; text-shadow: 3px 3px #000; margin-bottom: 20px; }
        .menu-btn { background: #27ae60; color: white; border: none; padding: 15px 40px; margin: 10px; cursor: pointer; font-size: 20px; border-radius: 5px; transition: 0.2s; pointer-events: auto; }
        .hotbar { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); display: flex; gap: 5px; background: rgba(0,0,0,0.8); padding: 8px; border-radius: 10px; pointer-events: auto; z-index: 10; }
        .slot { width: 44px; height: 44px; border: 2px solid #444; display: flex; align-items: center; justify-content: center; background: #222; position: relative; cursor: pointer; font-size: 20px; }
        .active { border-color: #f1c40f; background: #555; box-shadow: 0 0 10px #f1c40f; }
        .count { position: absolute; bottom: 1px; right: 3px; font-weight: bold; font-size: 11px; color: #fff; }
        #hp-bar { width: 100%; height: 12px; background: #333; margin-top: 5px; border-radius: 6px; overflow: hidden; }
        #hp-fill { height: 100%; background: #ff4757; width: 100%; transition: width 0.3s; }
        .craft-btn { background: #2980b9; color: white; border: none; padding: 6px; margin: 3px 0; cursor: pointer; width: 100%; font-size: 12px; border-radius: 4px; }
        .save-btn { background: #8e44ad; }
        .mobile-controls { position: absolute; bottom: 30px; left: 20px; display: flex; gap: 20px; z-index: 20; }
        .m-btn { width: 70px; height: 70px; background: rgba(255,255,255,0.15); border: 2px solid rgba(255,255,255,0.5); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 30px; user-select: none; -webkit-tap-highlight-color: transparent; pointer-events: auto; }
        .jump-btn { position: absolute; bottom: 30px; right: 20px; width: 85px; height: 85px; background: rgba(46, 204, 113, 0.2); }
        @media (min-width: 1024px) { .mobile-controls, .jump-btn { display: none !important; } }
        @media (max-width: 768px) { .hotbar { scale: 0.8; bottom: 110px; } .crafting { scale: 0.8; top: 10px; right: auto; left: 10px; margin-top: 150px; } }
    </style>
</head>
<body>

<div id="menu-overlay">
    <h1>TERRARIA JS</h1>
    <button class="menu-btn" onclick="startGame()">START NEW WORLD</button>
    <button class="menu-btn save-btn" onclick="loadGame()">LOAD SAVE</button>
</div>

<div class="mobile-controls">
    <div class="m-btn" id="btn-left">◀️</div>
    <div class="m-btn" id="btn-right">▶️</div>
</div>
<div class="m-btn jump-btn" id="btn-jump">JUMP</div>

<div class="ui-panel stats">
    <div><b>SURVIVOR</b> HP: <span id="hp-val">100</span></div>
    <div id="hp-bar"><div id="hp-fill"></div></div>
    <div style="font-size: 11px; margin-top:5px; color:#aaa;" id="armor-txt">Armor: None</div>
    <hr style="border: 0; border-top: 1px solid #444;">
    <button class="craft-btn save-btn" onclick="saveGame()">💾 SAVE WORLD</button>
    <div id="boss-log" style="color: #ff4757; font-weight: bold; display: none; text-align: center; margin-top: 10px;">BOSS: <span id="boss-hp">500</span></div>
</div>

<div class="ui-panel map-box">
    <canvas id="minimap" width="100" height="60" style="display:block; background:#111;"></canvas>
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

<canvas id="gameCanvas"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const mCanvas = document.getElementById('minimap');
const mctx = mCanvas.getContext('2d');

const TILE = 32, ROWS = 60, COLS = 100;
const EMPTY = 0, GRASS = 1, DIRT = 2, STONE = 3, IRON = 4;
const COLORS = { [GRASS]:'#2ecc71', [DIRT]:'#795548', [STONE]:'#95a5a6', [IRON]:'#dcdde1' };

let world = [], keys = {}, selectedTool = 1, camX = 0, camY = 0, gameTime = 0;
let projs = [], enemies = [], boss = null, flash = 0, isRunning = false;

const player = { 
    x: 0, y: 0, w: 20, h: 32, vx: 0, vy: 0, hp: 100, 
    grounded: false, iFrames: 0, armor: 0,
    inv: { [DIRT]: 0, [STONE]: 0, [GRASS]: 0, [IRON]: 0, arrows: 15, bullets: 10 }
};

function startGame() {
    document.getElementById('menu-overlay').style.display = 'none';
    isRunning = true;
    initWorld();
    spawnPlayer();
    requestAnimationFrame(gameLoop);
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
        if (world[r][50] !== EMPTY) { player.y = (r - 2) * TILE; break; }
    }
}

function spawnEnemy(x, y) {
    enemies.push({
        x: x, y: y, w: 24, h: 18, vx: 0, vy: 0,
        hp: 30, grounded: false, jumpTimer: Math.random() * 60, type: 'slime'
    });
}

function updateEnemies() {
    // Spawn slime every 5 seconds if < 5 exist
    if (gameTime % 300 === 0 && enemies.length < 5) {
        let spawnX = player.x + (Math.random() > 0.5 ? 500 : -500);
        spawnEnemy(spawnX, player.y - 100);
    }

    for (let i = enemies.length - 1; i >= 0; i--) {
        let en = enemies[i];
        en.jumpTimer++;
        
        if (en.grounded && en.jumpTimer > 80) {
            en.vy = -8;
            en.vx = (player.x > en.x) ? 3.5 : -3.5;
            en.grounded = false;
            en.jumpTimer = 0;
        }

        en.vy += 0.45;
        en.y += en.vy;
        en.grounded = false;
        collide(en, 'y');
        en.x += en.vx;
        collide(en, 'x');

        if (en.grounded) en.vx *= 0.85;

        // Player collision
        if (Math.abs(player.x - en.x) < 20 && Math.abs(player.y - en.y) < 20 && player.iFrames === 0) {
            player.hp -= 15 * (1 - player.armor);
            player.iFrames = 40;
        }

        if (en.hp <= 0 || en.y > ROWS * TILE) enemies.splice(i, 1);
    }
}

function saveGame() {
    const saveData = { world, player, gameTime };
    localStorage.setItem('terraria_save', JSON.stringify(saveData));
    alert("World Saved!");
}

function loadGame() {
    const data = localStorage.getItem('terraria_save');
    if (!data) return alert("No save file found!");
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

    updateEnemies();

    if (player.inv[STONE] >= 20 && !boss) {
        boss = { x: player.x, y: player.y - 400, hp: 500, state: 'HOVER', timer: 0, vx: 0, vy: 0 };
        document.getElementById('boss-log').style.display = 'block';
    }

    if (boss) {
        boss.timer++;
        if (boss.state === 'HOVER') {
            boss.vx = (player.x - boss.x) * 0.03; boss.vy = (player.y - 250 - boss.y) * 0.03;
            if (boss.timer > 100) { boss.state = 'CHARGE'; boss.timer = 0; }
        } else {
            let a = Math.atan2(player.y - (boss.y+40), player.x - (boss.x+40));
            boss.vx = Math.cos(a) * 10; boss.vy = Math.sin(a) * 10;
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
        // Projectile hit slimes
        enemies.forEach(en => {
            if (p.x > en.x && p.x < en.x + en.w && p.y > en.y && p.y < en.y + en.h) {
                en.hp -= p.dmg; projs.splice(i, 1);
            }
        });
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

    for (let r = 0; r < ROWS; r++) {
        for (let c = 0; c < COLS; c++) {
            if (world[r][c] !== EMPTY) {
                ctx.fillStyle = COLORS[world[r][c]];
                ctx.fillRect(c * TILE, r * TILE, TILE, TILE);
            }
        }
    }

    projs.forEach(p => { ctx.fillStyle = p.c; ctx.fillRect(p.x, p.y, 8, 2); });

    enemies.forEach(en => {
        ctx.fillStyle = '#27ae60';
        ctx.fillRect(en.x, en.y, en.w, en.h);
        ctx.fillStyle = 'white';
        ctx.fillRect(en.x + 4, en.y + 4, 4, 4); ctx.fillRect(en.x + en.w - 8, en.y + 4, 4, 4);
    });

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
    const mX = (clientX - rect.left) + camX;
    const mY = (clientY - rect.top) + camY;
    const c = Math.floor(mX/TILE), r = Math.floor(mY/TILE);

    if (selectedTool === 1 && world[r] && world[r][c]) { player.inv[world[r][c]]++; world[r][c] = 0; }
    else if (selectedTool === 2) { // Sword hitting boss OR slimes
        if (boss && Math.abs(mX-(boss.x+40))<70 && Math.abs(mY-(boss.y+40))<70) boss.hp -= 12;
        enemies.forEach(en => {
            if (Math.abs(mX-(en.x+en.w/2))<40 && Math.abs(mY-(en.y+en.h/2))<40) en.hp -= 15;
        });
    }
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

canvas.addEventListener('mousedown', e => handleInteraction(e.clientX, e.clientY));
canvas.addEventListener('touchstart', e => {
    e.preventDefault(); 
    handleInteraction(e.touches[0].clientX, e.touches[0].clientY);
}, { passive: false });

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
