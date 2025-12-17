<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Minecraft iPad Fix</title>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; touch-action: none; font-family: sans-serif; }
        canvas { display: block; }
        .controls {
            position: absolute; bottom: 20px; left: 20px; right: 20px;
            display: flex; justify-content: space-between; pointer-events: none;
        }
        .btn-group { display: flex; gap: 15px; pointer-events: auto; }
        .btn {
            width: 70px; height: 70px; background: rgba(0,0,0,0.4);
            border: 2px solid white; border-radius: 10px;
            display: flex; align-items: center; justify-content: center;
            color: white; font-weight: bold; font-size: 24px;
            -webkit-user-select: none;
        }
        .btn:active { background: rgba(255,255,255,0.4); }
    </style>
</head>
<body>
    <div class="controls">
        <div class="btn-group">
            <div class="btn" id="left">←</div>
            <div class="btn" id="right">→</div>
        </div>
        <div class="btn" id="jump">JUMP</div>
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        let w = canvas.width = window.innerWidth;
        let h = canvas.height = window.innerHeight;
        const bSize = 40;

        let world = {};
        let cameraX = 0;
        const player = { x: 200, y: 0, vx: 0, vy: 0, w: 30, h: 50, g: 0.6, jump: -14 };
        const keys = { left: false, right: false };

        const COLORS = { 1: '#4CAF50', 2: '#8B4513', 3: '#707070', 4: '#5D4037', 5: '#2E7D32' };

        function generateWorld() {
            for (let x = 0; x < 300; x++) {
                let groundY = 12 + Math.floor(Math.sin(x * 0.15) * 2);
                for (let y = 0; y < 30; y++) {
                    let type = 0;
                    if (y > groundY) type = 2;
                    if (y === groundY) type = 1;
                    if (y > groundY + 6) type = 3;
                    world[`${x},${y}`] = type;
                }
                // Bäume
                if (x % 12 === 0 && Math.random() > 0.4) {
                    for(let i=1; i<=3; i++) world[`${x},${groundY-i}`] = 4;
                    for(let lx=-1; lx<=1; lx++) {
                        for(let ly=-5; ly<=-4; ly++) world[`${x+lx},${groundY+ly}`] = 5;
                    }
                }
            }
        }

        function update() {
            // Bewegung
            if (keys.left) player.vx = -5;
            else if (keys.right) player.vx = 5;
            else player.vx = 0;

            player.vy += player.g;
            player.x += player.vx;
            player.y += player.vy;

            cameraX += (player.x - cameraX - w/2) * 0.1;

            // Kollision (Boden/Blöcke)
            let gridX = Math.floor((player.x + player.w/2) / bSize);
            let gridY = Math.floor((player.y + player.h) / bSize);
            if (world[`${gridX},${gridY}`] && world[`${gridX},${gridY}`] !== 0) {
                player.y = gridY * bSize - player.h;
                player.vy = 0;
            }
        }

        function draw() {
            ctx.fillStyle = "#87CEEB";
            ctx.fillRect(0, 0, w, h);

            ctx.save();
            ctx.translate(-cameraX, 0);

            let startX = Math.floor(cameraX / bSize);
            let endX = startX + Math.ceil(w / bSize) + 1;

            for (let x = startX; x < endX; x++) {
                for (let y = 0; y < 30; y++) {
                    let type = world[`${x},${y}`];
                    if (type) {
                        ctx.fillStyle = COLORS[type];
                        ctx.fillRect(x * bSize, y * bSize, bSize, bSize);
                    }
                }
            }

            ctx.fillStyle = "#3498db";
            ctx.fillRect(player.x, player.y, player.w, player.h);
            ctx.restore();

            update();
            requestAnimationFrame(draw);
        }

        // --- STEUERUNG FIX ---
        function setupButton(id, key) {
            const btn = document.getElementById(id);
            const start = (e) => { e.preventDefault(); if(key === 'jump') { if(player.vy === 0) player.vy = player.jump; } else { keys[key] = true; } };
            const end = (e) => { e.preventDefault(); if(key !== 'jump') keys[key] = false; };
            btn.addEventListener('touchstart', start);
            btn.addEventListener('touchend', end);
        }

        setupButton('left', 'left');
        setupButton('right', 'right');
        setupButton('jump', 'jump');

        // Abbauen (Tippen auf Welt)
        canvas.addEventListener('touchstart', (e) => {
            if (e.touches[0].clientX > 150 && e.touches[0].clientX < w - 100) {
                const touch = e.touches[0];
                const realX = Math.floor((touch.clientX + cameraX) / bSize);
                const realY = Math.floor(touch.clientY / bSize);
                world[`${realX},${realY}`] = 0;
            }
        });

        generateWorld();
        draw();
    </script>
</body>
</html>
