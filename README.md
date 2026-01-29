<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Space Shooter Spotify Edition</title>
    <style>
        body { margin: 0; background: #000; overflow: hidden; touch-action: none; font-family: 'Segoe UI', sans-serif; }
        canvas { display: block; }
        
        #stats { position: absolute; top: 10px; left: 10px; color: white; font-weight: bold; z-index: 10; pointer-events: none; }
        
        #menu, #settingsPanel { 
            position: absolute; width: 100%; height: 100%; 
            display: flex; flex-direction: column; justify-content: center; 
            align-items: center; color: white; background: rgba(0,0,0,0.9); z-index: 20;
        }
        
        #settingsPanel { display: none; }

        .btn-container { display: flex; gap: 15px; }
        button {
            padding: 15px 25px; font-size: 18px; border: none; border-radius: 8px; 
            cursor: pointer; color: white; font-weight: bold;
        }
        #playBtn { background: #1DB954; font-size: 22px; } /* Цвет Spotify */
        #settingsBtn { background: #444; }
        #backBtn { background: #e74c3c; margin-top: 15px; }

        /* Стили для плеера Spotify */
        #spotify-player {
            margin-top: 20px;
            border-radius: 12px;
            overflow: hidden;
            width: 300px;
            height: 80px;
        }
    </style>
</head>
<body>

    <div id="stats">Монеты: <span id="coins">0</span></div>

    <div id="menu">
        <h1 style="text-shadow: 0 0 15px #1DB954;">SPACE SHOOTER</h1>
        <p>Музыка от Spotify</p>
        <div class="btn-container">
            <button id="settingsBtn">⚙️</button>
            <button id="playBtn">START GAME</button>
        </div>
        <div id="spotify-player">
            <iframe src="https://open.spotify.com/embed/playlist/37i9dQZF1DX8S0uLYCKiCI?utm_source=generator&theme=0" 
                    width="100%" height="80" frameBorder="0" allowfullscreen="" 
                    allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy">
            </iframe>
        </div>
    </div>

    <div id="settingsPanel">
        <h2>НАСТРОЙКИ</h2>
        <p>Управляй музыкой через плеер в меню!</p>
        <button id="backBtn">НАЗАД В МЕНЮ</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const coinsElement = document.getElementById('coins');

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    let coins = 0;
    let gameActive = false;
    let player = { x: canvas.width/2 - 25, y: canvas.height - 120, w: 50, h: 50 };
    let bullets = [], enemies = [], walls = [], stars = [];

    // Звезды
    for(let i=0; i<80; i++) {
        stars.push({ x: Math.random()*canvas.width, y: Math.random()*canvas.height, s: Math.random()*2, sp: Math.random()*3 + 1 });
    }

    // Управление кнопками
    document.getElementById('playBtn').onclick = () => {
        document.getElementById('menu').style.display = 'none';
        gameActive = true;
        spawnObjects();
    };

    document.getElementById('settingsBtn').onclick = () => {
        document.getElementById('menu').style.display = 'none';
        document.getElementById('settingsPanel').style.display = 'flex';
    };

    document.getElementById('backBtn').onclick = () => {
        document.getElementById('settingsPanel').style.display = 'none';
        document.getElementById('menu').style.display = 'flex';
    };

    // Движение пальцем
    window.addEventListener('touchmove', (e) => {
        if(gameActive) player.x = e.touches[0].clientX - player.w/2;
    });

    function spawnObjects() {
        if (!gameActive) return;
        if (Math.random() < 0.6) {
            enemies.push({ x: Math.random()*(canvas.width-40), y: -50, w: 40, h: 40, speed: 4 });
        } else {
            walls.push({ x: Math.random()*(canvas.width-100), y: -100, w: 100, h: 25, speed: 2 });
        }
        setTimeout(spawnObjects, 700);
    }

    // Стрельба
    setInterval(() => {
        if(gameActive) bullets.push({ x: player.x + 22, y: player.y, w: 6, h: 15 });
    }, 250);

    function update() {
        stars.forEach(s => { s.y += s.sp; if(s.y > canvas.height) s.y = 0; });
        if(!gameActive) return;

        bullets.forEach((b, i) => { b.y -= 10; if(b.y < 0) bullets.splice(i, 1); });
        
        enemies.forEach((en, i) => {
            en.y += en.speed;
            if(checkCol(player, en)) end();
            if(en.y > canvas.height) enemies.splice(i, 1);
        });

        walls.forEach((w, i) => {
            w.y += w.speed;
            if(checkCol(player, w)) end();
            if(w.y > canvas.height) walls.splice(i, 1);
        });

        bullets.forEach((b, bi) => {
            enemies.forEach((en, ei) => {
                if(checkCol(b, en)) {
                    enemies.splice(ei, 1); bullets.splice(bi, 1);
                    coins += 10; coinsElement.innerText = coins;
                }
            });
        });
    }

    function checkCol(a, b) {
        return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
    }

    function end() {
        gameActive = false;
        alert("Корабль разбит! Монет: " + coins);
        location.reload();
    }

    function draw() {
        ctx.fillStyle = '#000';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = '#fff';
        stars.forEach(s => ctx.fillRect(s.x, s.y, s.s, s.s));

        if(gameActive) {
            ctx.fillStyle = '#1DB954'; // Цвет игрока как у Spotify
            ctx.fillRect(player.x, player.y, player.w, player.h);
            ctx.fillStyle = '#fff';
            bullets.forEach(b => ctx.fillRect(b.x, b.y, b.w, b.h));
            ctx.fillStyle = '#ff4d4d';
            enemies.forEach(en => ctx.fillRect(en.x, en.y, en.w, en.h));
            ctx.fillStyle = '#777';
            walls.forEach(w => ctx.fillRect(w.x, w.y, w.w, w.h));
        }
        update();
        requestAnimationFrame(draw);
    }
    draw();
</script>
</body>
</html>

  
