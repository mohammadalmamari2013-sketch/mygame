# mygame
<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>Battle Car Pro - المطور</title>
<style>
    body { margin: 0; overflow: hidden; background: #000; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
    canvas { display: block; }

    /* واجهة اللوبي */
    #lobby {
        position: fixed; top: 0; left: 0; width: 100%; height: 100%;
        background: radial-gradient(circle, #2c3e50, #000);
        display: flex; flex-direction: column; align-items: center; justify-content: center;
        color: white; z-index: 100;
    }
    #lobby h1 { font-size: 50px; text-shadow: 0 0 20px #3498db; margin-bottom: 20px; }
    .btn {
        padding: 15px 40px; font-size: 20px; background: #3498db; color: white;
        border: none; border-radius: 50px; cursor: pointer; transition: 0.3s;
    }
    .btn:hover { background: #2980b9; transform: scale(1.1); }

    /* واجهة المستخدم أثناء اللعب */
    #ui { 
        position: fixed; top: 20px; left: 20px; color: white; pointer-events: none; display: none;
    }
    #health-bar { width: 200px; height: 20px; background: #444; border: 2px solid #fff; border-radius: 10px; overflow: hidden; }
    #health-inner { width: 100%; height: 100%; background: #ff4444; transition: 0.2s; }

    /* أزرار الجوال */
    #joy-container {
        position: fixed; left: 40px; bottom: 40px; width: 120px; height: 120px;
        background: rgba(255,255,255,0.1); border: 2px solid rgba(255,255,255,0.3);
        border-radius: 50%; display: none; touch-action: none;
    }
    #joy-stick {
        position: absolute; left: 35px; top: 35px; width: 50px; height: 50px;
        background: #fff; border-radius: 50%; box-shadow: 0 0 15px rgba(0,0,0,0.5);
    }
    #fire-btn {
        position: fixed; right: 40px; bottom: 60px; width: 80px; height: 80px;
        background: rgba(255,0,0,0.6); border: 4px solid #fff; border-radius: 50%;
        display: none; justify-content: center; align-items: center; font-size: 35px;
        color: white; touch-action: none;
    }
</style>
</head>
<body>

<div id="lobby">
    <h1>BATTLE PRO</h1>
    <button class="btn" onclick="startGame()">ابدأ اللعب 🔥</button>
</div>

<div id="ui">
    <div id="health-bar"><div id="health-inner"></div></div>
    <div style="font-size: 20px; margin-top: 10px;">💎 <span id="gems">0</span></div>
</div>

<div id="joy-container"><div id="joy-stick"></div></div>
<div id="fire-btn">🔫</div>

<canvas id="c"></canvas>

<script>
const c = document.getElementById("c");
const ctx = c.getContext("2d");
let gameActive = false;

// إعدادات اللاعب
let player = { x: 500, y: 500, size: 30, speed: 4, health: 100, gems: 0, color: "#3498db" };
let move = { x: 0, y: 0 };
let enemies = [];
let bullets = [];
let gems = [];

// توليد خريطة عشوائية
for(let i=0; i<50; i++) gems.push({x: Math.random()*2000, y: Math.random()*2000});
for(let i=0; i<15; i++) enemies.push({x: Math.random()*2000, y: Math.random()*2000, hp: 3});

function startGame() {
    document.getElementById("lobby").style.display = "none";
    document.getElementById("ui").style.display = "block";
    document.getElementById("joy-container").style.display = "block";
    document.getElementById("fire-btn").style.display = "flex";
    gameActive = true;
    loop();
}

// التحكم بالجويستيك للموبايل
const joyContainer = document.getElementById("joy-container");
const joyStick = document.getElementById("joy-stick");

joyContainer.addEventListener("touchstart", handleTouch);
joyContainer.addEventListener("touchmove", handleTouch);
joyContainer.addEventListener("touchend", () => {
    move.x = 0; move.y = 0;
    joyStick.style.left = "35px"; joyStick.style.top = "35px";
});

function handleTouch(e) {
    e.preventDefault();
    let r = joyContainer.getBoundingClientRect();
    let touch = e.touches[0];
    let dx = touch.clientX - (r.left + 60);
    let dy = touch.clientY - (r.top + 60);
    let dist = Math.sqrt(dx*dx + dy*dy);
    let maxDist = 40;

    if(dist > maxDist) {
        dx *= maxDist / dist;
        dy *= maxDist / dist;
    }

    move.x = dx / maxDist;
    move.y = dy / maxDist;
    joyStick.style.left = (35 + dx) + "px";
    joyStick.style.top = (35 + dy) + "px";
}

// زر الإطلاق
document.getElementById("fire-btn").addEventListener("touchstart", (e) => {
    e.preventDefault();
    bullets.push({ x: player.x, y: player.y, vx: move.x * 10 || 10, vy: move.y * 10 || 0 });
});

function update() {
    if(!gameActive) return;

    // حركة اللاعب
    player.x += move.x * player.speed;
    player.y += move.y * player.speed;

    // موت اللاعب
    if(player.health <= 0) {
        alert("Game Over! لقد مت.");
        location.reload();
    }

    // الأعداء
    enemies.forEach((en, i) => {
        let dx = player.x - en.x;
        let dy = player.y - en.y;
        let dist = Math.hypot(dx, dy);
        en.x += (dx/dist) * 1.5;
        en.y += (dy/dist) * 1.5;

        if(dist < 30) player.health -= 0.5; // نقص الدم عند اللمس
    });

    // تحديث الواجهة
    document.getElementById("health-inner").style.width = player.health + "%";
    document.getElementById("gems").innerText = player.gems;
}

function draw() {
    ctx.fillStyle = "#111";
    ctx.fillRect(0,0,c.width,c.height);

    ctx.save();
    ctx.translate(c.width/2 - player.x, c.height/2 - player.y);

    // رسم الأرضية
    ctx.strokeStyle = "#222";
    for(let i=0; i<2000; i+=100) {
        ctx.beginPath(); ctx.moveTo(i, 0); ctx.lineTo(i, 2000); ctx.stroke();
        ctx.beginPath(); ctx.moveTo(0, i); ctx.lineTo(2000, i); ctx.stroke();
    }

    // رسم الجواهر
    ctx.fillStyle = "gold";
    gems.forEach((g, i) => {
        ctx.beginPath(); ctx.arc(g.x, g.y, 8, 0, 7); ctx.fill();
        if(Math.hypot(player.x-g.x, player.y-g.y) < 30) { gems.splice(i,1); player.gems++; }
    });

    // رسم الأعداء
    ctx.fillStyle = "red";
    enemies.forEach(en => ctx.fillRect(en.x-15, en.y-15, 30, 30));

    // رسم الرصاص
    ctx.fillStyle = "yellow";
    bullets.forEach((b, i) => {
        b.x += b.vx; b.y += b.vy;
        ctx.fillRect(b.x, b.y, 10, 10);
        if(b.x > 3000 || b.x < -1000) bullets.splice(i,1);
    });

    // رسم اللاعب
    ctx.fillStyle = player.color;
    ctx.fillRect(player.x-15, player.y-15, 30, 30);

    ctx.restore();
}

function loop() {
    update();
    draw();
    if(gameActive) requestAnimationFrame(loop);
}

c.width = window.innerWidth;
c.height = window.innerHeight;
</script>
</body>
</html>

