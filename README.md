<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>BlockCraft Pro Fix</title>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; touch-action: none; font-family: sans-serif; }
        canvas { display: block; }
        .controls { position: absolute; bottom: 40px; left: 0; right: 0; display: flex; justify-content: space-around; width: 100%; pointer-events: none; }
        .btn { 
            width: 90px; height: 90px; background: rgba(0,0,0,0.6); border: 3px solid white; 
            border-radius: 20px; color: white; display: flex; align-items: center; 
            justify-content: center; font-weight: bold; font-size: 18px; pointer-events: auto;
            -webkit-user-select: none; user-select: none;
        }
        .btn:active { background: rgba(255,255,255,0.4); }
    </style>
</head>
<body>
    <div class="controls">
        <div style="display: flex; gap: 30px;">
            <div class="btn" id="btnLeft">LINKS</div>
            <div class="btn" id="btnRight">RECHTS</div>
        </div>
        <div class="btn" id="btnJump">SPRUNG</div>
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
        const player = { x: 400, y: 100, vx: 0, vy: 0, w: 28, h: 50, g: 0.8, jump: -16, grounded: false };
        const keys = { left: false, right: false };

        const COLORS = { 1: '#4CAF50', 2: '#8B4513', 3: '#707070', 4: '#5D4037', 5: '#2E7D32' };

        function generateWorld() {
            for (let x = 0; x < 500; x++) {
                let groundY = 10 + Math.floor(Math.sin(x * 0.12) * 2);
                for (let y = 0; y < 25; y++) {
                    let type = 0;
                    if (y > groundY) type = 2;
                    if (y === groundY) type = 1;
                    if (y > groundY + 7) type = 3;
                    world[`${x},${y}`] = type;
                }
                if (x % 15 === 0 && x > 15) {
                    let treeY = groundY;
                    for(let i=1; i<=4; i++) world[`${x},${treeY-i}`] = 4;
                    for(let lx=-2; lx<=2; lx++) for(let ly=-7; ly<=-5; ly++) world[`${x+lx},${treeY+ly}`] = 5;
                }
            }
        }

        function isSolid(px, py) {
            let gx = Math.floor(px / bSize);
            let gy = Math.floor(py / bSize);
            return (world[`${gx},${gy}`] && world[`${gx},${gy}`] !== 0);
        }

        function update() {
            if (keys.left) player.vx = -6;
            else if (keys.right) player.vx = 6;
            else player.vx = 0;

            player.vy += player.g;

            // X-Kollision (Seitlich)
            let nextX = player.x + player.vx;
            if (!isSolid(nextX, player.y + 2) && !isSolid(nextX + player.w, player.y + 2) &&
                !isSolid(nextX, player.y + player.h - 2) && !isSolid(nextX + player.w, player.y + player.h - 2)) {
                player.x = nextX;
            }

            // Y-Kollision (Oben/Unten)
            let nextY = player.y + player.vy;
            player.grounded = false;

            if (player.vy < 0) { // Aufwärts (Kopf-Check)
                if (!isSolid(player.x + 4, nextY) && !isSolid(player.x + player.w - 4, nextY)) {
                    player.y = nextY;
                } else { player.vy = 0; }
            } else { // Abwärts (Boden-Check)
                if (!isSolid(player.x + 4, nextY + player.h) && !isSolid(player.x + player.w - 4, nextY + player.h)) {
                    player.y = nextY;
                } else {
                    player.y = Math.floor((player.y + player.h) / bSize) * bSize - player.h;
                    player.vy = 0;
                    player.grounded = true;
                }
            }

            cameraX += (player.x - cameraX - w/2) * 0.1;
        }

        function draw() {
            ctx.fillStyle = "#87CEEB";
            ctx.fillRect(0, 0, w, h);

            ctx.save();
            ctx.translate(-cameraX, 0);

            let startX = Math.floor(cameraX / bSize) - 1;
            let endX = startX + Math.ceil(w / bSize) + 2;

            for (let x = startX; x < endX; x++) {
                for (let y = 0; y < 25; y++) {
                    let type = world[`${x},${y}`];
                    if (type) {
                        ctx.fillStyle = COLORS[type];
                        ctx.fillRect(x * bSize, y * bSize, bSize, bSize);
                        ctx.strokeStyle = "rgba(0,0,0,0.05)";
                        ctx.strokeRect(x * bSize, y * bSize, bSize, bSize);
                    }
                }
            }

            ctx.fillStyle = "#3498db";
            ctx.fillRect(player.x, player.y, player.w, player.h);
            ctx.fillStyle = "#ffdbac";
            ctx.fillRect(player.x + 2, player.y - 20, 24, 24);

            ctx.restore();
            update();
            requestAnimationFrame(draw);
        }

        // --- FIXED CONTROLS ---
        const btnL = document.getElementById('btnLeft');
        const btnR = document.getElementById('btnRight');
        const btnJ = document.getElementById('btnJump');

        btnL.addEventListener('touchstart', (e) => { e.preventDefault(); keys.left = true; });
        btnL.addEventListener('touchend', (e) => { e.preventDefault(); keys.left = false; });
        
        btnR.addEventListener('touchstart', (e) => { e.preventDefault(); keys.right = true; });
        btnR.addEventListener('touchend', (e) => { e.preventDefault(); keys.right = false; });

        // Jump Fix: Nur ein Impuls pro Touch
        btnJ.addEventListener('touchstart', (e) => { 
            e.preventDefault(); 
            if (player.grounded) {
                player.vy = player.jump;
                player.grounded = false;
            }
        });

        // Abbauen
        canvas.addEventListener('touchstart', (e) => {
            const t = e.touches[0];
            if (t.clientY < h - 160) {
                let wx = Math.floor((t.clientX + cameraX) / bSize);
                let wy = Math.floor(t.clientY / bSize);
                world[`${wx},${wy}`] = 0;
            }
        }, {passive: false});

        window.addEventListener('resize', () => { w = canvas.width = window.innerWidth; h = canvas.height = window.innerHeight; });

        generateWorld();
        draw();
    </script>
</body>
</html>
