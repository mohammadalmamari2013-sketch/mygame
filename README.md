<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>BATTLE PRO - MAX</title>
    <style>
        :root { --gold: #f1c40f; --blue: #3498db; --red: #e74c3c; }
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Segoe UI', sans-serif; color: white; touch-action: none; }
        
        /* اللوبي الاحترافي */
        #lobby {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(to bottom, #1a2a6c, #b21f1f, #fdbb2d);
            display: flex; z-index: 100;
        }

        /* القائمة الجانبية للمتجر وعجلة الحظ */
        .side-menu {
            position: absolute; right: 20px; top: 50%; transform: translateY(-50%);
            display: flex; flex-direction: column; gap: 15px;
        }
        .menu-btn {
            background: rgba(0,0,0,0.6); border: 2px solid var(--gold); color: white;
            padding: 15px; border-radius: 12px; font-weight: bold; cursor: pointer;
        }

        /* الشخصية 3D في اللوبي */
        .char-container {
            flex: 1; display: flex; align-items: center; justify-content: center;
            perspective: 1000px;
        }
        .cube {
            width: 80px; height: 120px; position: relative; transform-style: preserve-3d;
            animation: rotate 5s infinite linear;
        }
        .cube div {
            position: absolute; width: 80px; height: 120px; background: var(--blue);
            border: 2px solid white; opacity: 0.9;
        }
        @keyframes rotate { from { transform: rotateY(0); } to { transform: rotateY(360deg); } }

        /* عجلة الحظ */
        #luck-wheel {
            position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 250px; height: 250px; border-radius: 50%; border: 5px solid var(--gold);
            background: conic-gradient(red, blue, green, yellow, purple, orange);
            display: none; z-index: 110; transition: transform 3s cubic-bezier(0.1, 0, 0.1, 1);
        }

        /* زر البدء */
        #start-btn {
            position: absolute; bottom: 50px; left: 50%; transform: translateX(-50%);
            padding: 20px 60px; font-size: 24px; background: var(--gold); color: black;
            border: none; border-radius: 50px; font-weight: bold; box-shadow: 0 0 20px var(--gold);
        }

        /* أزرار الجوال أثناء اللعب */
        #ui-game { display: none; }
        #joy-bg { position: fixed; left: 40px; bottom: 40px; width: 120px; height: 120px; background: rgba(255,255,255,0.2); border-radius: 50%; }
        #joy-thumb { position: absolute; left: 35px; top: 35px; width: 50px; height: 50px; background: #fff; border-radius: 50%; }
        #fire-btn { position: fixed; right: 40px; bottom: 60px; width: 90px; height: 90px; background: var(--red); border: 4px solid #fff; border-radius: 50%; display: flex; justify-content: center; align-items: center; font-size: 40px; box-shadow: 0 0 20px var(--red); }
        
        #stats { position: fixed; top: 10px; right: 10px; text-align: left; }
    </style>
</head>
<body>

<div id="lobby">
    <div id="stats">💎 جواهر: <span id="gems">100</span></div>
    
    <div class="side-menu">
        <div class="menu-btn" onclick="alert('المتجر: قريباً سكنات أسطورية!')">🛒 المتجر</div>
        <div class="menu-btn" onclick="openWheel()">🎡 عجلة الحظ</div>
    </div>

    <div class="char-container">
        <div class="cube">
            <div style="transform: translateZ(40px)"></div>
            <div style="transform: rotateY(180deg) translateZ(40px)"></div>
            <div style="transform: rotateY(90deg) translateZ(40px)"></div>
            <div style="transform: rotateY(-90deg) translateZ(40px)"></div>
        </div>
    </div>

    <button id="start-btn" onclick="startGame()">ابدأ المعركة 🔥</button>
</div>

<div id="luck-wheel"></div>

<div id="ui-game">
    <div id="joy-bg"><div id="joy-thumb"></div></div>
    <div id="fire-btn">🔫</div>
    <div style="position:fixed; top:20px; left:20px; width:200px; height:20px; background:#444; border:2px solid #fff;">
        <div id="hp" style="width:100%; height:100%; background:var(--red);"></div>
    </div>
</div>

<canvas id="c"></canvas>

<script>
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");
let inGame = false;
let player = { x: 500, y: 500, hp: 100, gems: 100 };
let joystick = { x: 0, y: 0, active: false };
let bullets = [];
let enemies = [];

function openWheel() {
    let w = document.getElementById("luck-wheel");
    w.style.display = "block";
    let angle = Math.floor(Math.random() * 3600) + 360;
    w.style.transform = `translate(-50%, -50%) rotate(${angle}deg)`;
    setTimeout(() => { 
        alert("مبروك! حصلت على 50 جوهرة 💎"); 
        player.gems += 50;
        document.getElementById("gems").innerText = player.gems;
        w.style.display = "none";
    }, 3500);
}

function startGame() {
    document.getElementById("lobby").style.display = "none";
    document.getElementById("ui-game").style.display = "block";
    inGame = true;
    for(let i=0; i<10; i++) enemies.push({x: Math.random()*1500, y: Math.random()*1500});
    loop();
}

// تحكم الجويستيك للجوال
window.addEventListener("touchstart", (e) => {
    let t = e.touches[0];
    if(t.clientX < 250) joystick.active = true;
});

window.addEventListener("touchmove", (e) => {
    if(!joystick.active) return;
    let t = e.touches[0];
    let rect = document.getElementById("joy-bg").getBoundingClientRect();
    let dx = t.clientX - (rect.left + 60);
    let dy = t.clientY - (rect.top + 60);
    let d = Math.sqrt(dx*dx + dy*dy);
    if(d > 40) { dx *= 40/d; dy *= 40/d; }
    joystick.x = dx / 10;
    joystick.y = dy / 10;
    document.getElementById("joy-thumb").style.transform = `translate(${dx}px, ${dy}px)`;
});

window.addEventListener("touchend", () => {
    joystick.active = false;
    joystick.x = 0; joystick.y = 0;
    document.getElementById("joy-thumb").style.transform = `translate(0,0)`;
});

// زر الإطلاق
document.getElementById("fire-btn").addEventListener("touchstart", (e) => {
    e.preventDefault();
    bullets.push({x: player.x, y: player.y, vx: joystick.x*5 || 10, vy: joystick.y*5 || 0});
});

function loop() {
    if(!inGame) return;
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    
    player.x += joystick.x;
    player.y += joystick.y;

    ctx.fillStyle = "#111";
    ctx.fillRect(0,0,canvas.width, canvas.height);

    ctx.save();
    ctx.translate(canvas.width/2 - player.x, canvas.height/2 - player.y);

    // رسم اللاعب 3D (مربع بظل)
    ctx.fillStyle = "var(--blue)";
    ctx.fillRect(player.x-20, player.y-20, 40, 40);
    ctx.strokeStyle = "white"; ctx.strokeRect(player.x-20, player.y-20, 40, 40);

    // الأعداء والرصاص
    enemies.forEach((en, i) => {
        let d = Math.hypot(player.x - en.x, player.y - en.y);
        en.x += (player.x - en.x)/d * 2;
        en.y += (player.y - en.y)/d * 2;
        ctx.fillStyle = "red"; ctx.fillRect(en.x-15, en.y-15, 30, 30);
        if(d < 30) { player.hp -= 0.5; document.getElementById("hp").style.width = player.hp + "%"; }
        if(player.hp <= 0) location.reload();
    });

    bullets.forEach((b, i) => {
        b.x += b.vx; b.y += b.vy;
        ctx.fillStyle = "yellow"; ctx.fillRect(b.x, b.y, 10, 10);
    });

    ctx.restore();
    requestAnimationFrame(loop);
}
</script>
</body>
</html>
