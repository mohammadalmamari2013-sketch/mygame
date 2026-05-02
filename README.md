<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>BATTLE PRO MAX - THE BIG UPDATE</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: sans-serif; touch-action: none; color: white; }
        
        /* واجهة التحكم العلوية */
        #ui-layer { position: fixed; top: 0; width: 100%; z-index: 100; pointer-events: none; }
        .stats { display: flex; justify-content: flex-end; padding: 10px; gap: 10px; }
        .stat-box { background: rgba(0,0,0,0.8); padding: 5px 15px; border: 2px solid #f1c40f; border-radius: 15px; pointer-events: auto; }

        /* أزرار المتجر والرقصات (ثابتة على اليمين) */
        #side-buttons { position: fixed; right: 10px; top: 100px; z-index: 101; display: flex; flex-direction: column; gap: 10px; }
        .btn { width: 70px; height: 70px; background: rgba(0,0,0,0.7); border: 2px solid #00f3ff; border-radius: 10px; color: white; cursor: pointer; font-size: 12px; font-weight: bold; }

        /* الجويستيك وزر الإطلاق */
        #controls { position: fixed; bottom: 0; width: 100%; height: 150px; z-index: 100; pointer-events: none; }
        #joystick { position: absolute; left: 30px; bottom: 30px; width: 100px; height: 100px; background: rgba(255,255,255,0.2); border-radius: 50%; border: 2px solid #fff; pointer-events: auto; }
        #stick { position: absolute; left: 30px; top: 30px; width: 40px; height: 40px; background: #fff; border-radius: 50%; }
        #fire-btn { position: absolute; right: 30px; bottom: 50px; width: 90px; height: 90px; background: rgba(255,0,0,0.4); border: 4px solid white; border-radius: 50%; pointer-events: auto; font-size: 40px; display: flex; align-items: center; justify-content: center; }

        /* النوافذ المنبثقة */
        .window { display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 80%; height: 60%; background: #111; border: 3px solid #f1c40f; z-index: 200; border-radius: 20px; padding: 20px; text-align: center; }
    </style>
</head>
<body>

<div id="ui-layer">
    <div class="stats">
        <div class="stat-box">💎 9999</div>
        <div class="stat-box">💰 50000</div>
    </div>
</div>

<div id="side-buttons">
    <button class="btn" onclick="toggleWin('shop')">🛒<br>المتجر</button>
    <button class="btn" onclick="playEmote()">🕺<br>رقصة</button>
    <button class="btn" onclick="alert('تم تفعيل الهكر!')">💀<br>مود X</button>
</div>

<div id="controls">
    <div id="joystick"><div id="stick"></div></div>
    <div id="fire-btn">🔫</div>
</div>

<div id="shop" class="window">
    <h2>متجر السكنات الأسطورية</h2>
    <div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px;">
        <button style="padding:10px;" onclick="changeSkin(0x00f3ff)">سكن نيون 🔵</button>
        <button style="padding:10px;" onclick="changeSkin(0xff0000)">سكن نار 🔴</button>
        <button style="padding:10px;" onclick="changeSkin(0xf1c40f)">سكن ذهب 🟡</button>
        <button style="padding:10px;" onclick="changeSkin(0xffffff)">سكن أبيض ⚪</button>
    </div>
    <button onclick="toggleWin('shop')" style="margin-top:20px; width:100%; background:red; color:white; border:none; padding:10px;">إغلاق</button>
</div>

<script>
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0a0a0a);
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // إضاءة شمسية قوية + إضاءة محيطة (عشان السكن يظهر غصب!)
    const sun = new THREE.DirectionalLight(0xffffff, 1.5);
    sun.position.set(10, 20, 10);
    scene.add(sun);
    scene.add(new THREE.AmbientLight(0xffffff, 0.8));

    // اللاعب (سكن أبيض فاقع كبداية عشان تفتحه وتشوفه)
    const playerGeo = new THREE.CapsuleGeometry(0.5, 1, 4, 12);
    const playerMat = new THREE.MeshStandardMaterial({ color: 0xffffff, emissive: 0x333333 });
    const player = new THREE.Mesh(playerGeo, playerMat);
    player.position.y = 1;
    scene.add(player);

    // الأرضية
    const grid = new THREE.GridHelper(1000, 100, 0x00f3ff, 0x222222);
    scene.add(grid);

    // العقاب: إضافة 500 شي تفاعلي
    const objects = [];
    for(let i=0; i<500; i++) {
        const type = Math.random();
        let mesh;
        if(type < 0.2) { // سيارات (مكعبات طويلة)
            mesh = new THREE.Mesh(new THREE.BoxGeometry(2, 1, 4), new THREE.MeshStandardMaterial({color: 0x555555}));
        } else if(type < 0.5) { // براميل متفجرة
            mesh = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.5, 1.5), new THREE.MeshStandardMaterial({color: 0xff4400}));
        } else { // مباني وأعمدة
            mesh = new THREE.Mesh(new THREE.BoxGeometry(1.5, Math.random()*10, 1.5), new THREE.MeshStandardMaterial({color: Math.random()*0xffffff}));
        }
        mesh.position.set(Math.random()*800-400, 0.5, Math.random()*800-400);
        scene.add(mesh);
        objects.push(mesh);
    }

    camera.position.set(0, 5, 10);

    // وظائف الأزرار
    function toggleWin(id) {
        const el = document.getElementById(id);
        el.style.display = el.style.display === 'block' ? 'none' : 'block';
    }

    function changeSkin(color) {
        player.material.color.setHex(color);
        toggleWin('shop');
    }

    function playEmote() {
        player.rotation.y += Math.PI * 2;
        player.position.y += 2;
        setTimeout(() => player.position.y = 1, 300);
    }

    // الحركة (جويستيك مطور)
    let moveDir = { x: 0, z: 0 };
    const joy = document.getElementById('joystick');
    const stk = document.getElementById('stick');

    joy.addEventListener('touchmove', (e) => {
        const t = e.touches[0];
        const r = joy.getBoundingClientRect();
        const dx = t.clientX - (r.left + 50);
        const dy = t.clientY - (r.top + 50);
        const d = Math.min(Math.sqrt(dx*dx + dy*dy), 40);
        const angle = Math.atan2(dy, dx);
        stk.style.transform = `translate(${Math.cos(angle)*d}px, ${Math.sin(angle)*d}px)`;
        moveDir.x = Math.cos(angle) * (d/40);
        moveDir.z = Math.sin(angle) * (d/40);
    });

    joy.addEventListener('touchend', () => {
        stk.style.transform = `translate(0,0)`;
        moveDir = { x: 0, z: 0 };
    });

    function animate() {
        requestAnimationFrame(animate);
        if(moveDir.x !== 0 || moveDir.z !== 0) {
            player.position.x += moveDir.x * 0.3;
            player.position.z += moveDir.z * 0.3;
            player.rotation.y = Math.atan2(-moveDir.x, -moveDir.z); // يلف باتجاه المشي
        }
        camera.position.lerp(new THREE.Vector3(player.position.x, 6, player.position.z + 12), 0.1);
        camera.lookAt(player.position);
        renderer.render(scene, camera);
    }
    animate();
</script>
</body>
</html>
