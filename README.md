<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Minecraft iPad Edition</title>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; touch-action: none; font-family: sans-serif; }
        canvas { display: block; }
        .ui-overlay { position: absolute; top: 10px; left: 10px; color: white; text-shadow: 1px 1px 2px black; pointer-events: none; }
        .controls { position: absolute; bottom: 30px; left: 0; right: 0; display: flex; justify-content: space-around; width: 100%; pointer-events: none; }
        .btn { 
            width: 80px; height: 80px; background: rgba(0,0,0,0.5); border: 3px solid white; 
            border-radius: 15px; color: white; display: flex; align-items: center; 
            justify-content: center; font-weight: bold; font-size: 20px; pointer-events: auto;
            -webkit-user-select: none;
        }
        .btn:active { background: rgba(255,255,255,0.5); }
    </style>
</head>
<body>
    <div class="ui-overlay"><b>BlockCraft 2.0</b></div>
    <div class="controls">
        <div style="display: flex; gap: 20px;">
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
        const player = { x: 400, y: 100, vx: 0, vy: 0, w: 32, h: 52, g: 0.7, jump: -15, grounded: false };
        const keys = { left: false, right: false, jump: false };

        const COLORS = { 1: '#4CAF50', 2: '#8B4513', 3: '#707070', 4: '#5D4037', 5: '#2E7D32' };

        function generateWorld() {
            for (let x = 0; x < 500; x++) {
                let groundY = 10 + Math.floor(Math.sin(x * 0.1) * 3);
                for (let y = 0; y < 25; y++) {
                    let type = 0;
                    if (y > groundY) type = 2;
                    if (y === groundY) type = 1;
                    if (y > groundY + 7) type = 3;
                    world[`${x},${y}`] = type;
                }
                if (x % 15 === 0 && x > 10) { // Bäume
                    for(let i=1; i<=4; i++) world[`${x},${groundY-i}`] = 4;
                    for(let lx=-1; lx<=1; lx++) for(let ly=-6; ly<=-4; ly++) world[`${x+lx},${groundY+ly}`] = 5;
                }
            }
        }

        function getBlock(px, py) {
            let gx = Math.floor(px / bSize);
            let gy = Math.floor(py / bSize);
            return world[`${gx},${gy}`] || 0;
        }

        function update() {
            // Horizontaler Speed
            if (keys.left) player.vx = -6;
            else if (keys.right) player.vx = 6;
            else player.vx = 0;

            // Gravitation
            player.vy += player.g;

            // X-Kollision
            let nextX = player.x + player.vx;
            if (!getBlock(nextX, player.y) && !getBlock(nextX + player.w, player.y) &&
                !getBlock(nextX, player.y + player.h - 5) && !getBlock(nextX + player.w, player.y + player.h - 5)) {
                player.x = nextX;
            }

            // Y-Kollision
            player.grounded = false;
            let nextY = player.y + player.vy;
            if (player.vy < 0) { // Springen
                if (!getBlock(player.x + 5, nextY) && !getBlock(player.x + player.w - 5, nextY)) {
                    player.y = nextY;
                } else { player.vy = 0; }
            } else { // Fallen
                if (!getBlock(player.x + 5, nextY + player.h) && !getBlock(player.x + player.w - 5, nextY + player.h)) {
                    player.y = nextY;
                } else {
                    player.y = Math.floor((player.y + player.h) / bSize) * bSize - player.h;
                    player.vy = 0;
                    player.grounded = true;
                }
            }

            // Sprung ausführen
            if (keys.jump && player.grounded) {
                player.vy = player.jump;
                player.grounded = false;
            }

            cameraX += (player.x - cameraX - w/2) * 0.1;
        }

        function draw() {
            ctx.fillStyle = "#87CEEB";
            ctx.fillRect(0, 0, w, h);

            ctx.save();
            ctx.translate(-cameraX, 0);

            // Sichtbare Welt
            let startX = Math.floor(cameraX / bSize) - 1;
            let endX = startX + Math.ceil(w / bSize) + 2;

            for (let x = startX; x < endX; x++) {
                for (let y = 0; y < 25; y++) {
                    let type = world[`${x},${y}`];
                    if (type) {
                        ctx.fillStyle = COLORS[type];
                        ctx.fillRect(x * bSize, y * bSize, bSize, bSize);
                        ctx.strokeStyle = "rgba(0,0,0,0.1)";
                        ctx.strokeRect(x * bSize, y * bSize, bSize, bSize);
                    }
                }
            }

            // Spieler
            ctx.fillStyle = "#3498db";
            ctx.fillRect(player.x, player.y, player.w, player.h);
            ctx.fillStyle = "#ffdbac";
            ctx.fillRect(player.x + 4, player.y - 18, 24, 24);

            ctx.restore();
            update();
            requestAnimationFrame(draw);
        }

        // --- IPAD CONTROLS ---
        const setup = (id, action, isJump = false) => {
            const el = document.getElementById(id);
            el.addEventListener('touchstart', (e) => { e.preventDefault(); if(isJump) keys.jump = true; else keys[action] = true; });
            el.addEventListener('touchend', (e) => { e.preventDefault(); if(isJump) keys.jump = false; else keys[action] = false; });
        };

        setup('btnLeft', 'left');
        setup('btnRight', 'right');
        setup('btnJump', '', true);

        // Abbauen
        canvas.addEventListener('touchstart', (e) => {
            const t = e.touches[0];
            if (t.clientY < h - 150) { // Nur im oberen Bereich abbauen
                let wx = Math.floor((t.clientX + cameraX) / bSize);
                let wy = Math.floor(t.clientY / bSize);
                world[`${wx},${wy}`] = 0;
            }
        });

        generateWorld();
        draw();
    </script>
</body>
</html>
