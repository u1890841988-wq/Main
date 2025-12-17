<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>BlockCraft 2D</title>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; touch-action: none; font-family: sans-serif; }
        canvas { display: block; image-rendering: pixelated; }
        #inventory {
            position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%);
            display: flex; gap: 10px; background: rgba(0,0,0,0.5); padding: 10px; border-radius: 8px;
        }
        .slot { width: 40px; height: 40px; border: 2px solid #fff; }
        #debug { position: absolute; top: 10px; left: 10px; color: white; pointer-events: none; }
    </style>
</head>
<body>
    <div id="debug">Tippe Blöcke zum Abbauen an</div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Einstellungen
        const blockSize = 40;
        let world = [];
        const cols = Math.ceil(window.innerWidth / blockSize) + 1;
        const rows = Math.ceil(window.innerHeight / blockSize);

        // Spieler
        const player = {
            x: 100, y: 0, w: 30, h: 50,
            dy: 0, speed: 5, jump: -12, g: 0.6
        };

        function initWorld() {
            for (let x = 0; x < cols; x++) {
                world[x] = [];
                for (let y = 0; y < rows; y++) {
                    let type = 0; // Luft
                    let groundLevel = 8;
                    if (y > groundLevel) type = 2; // Erde
                    if (y === groundLevel) type = 1; // Gras
                    if (y > groundLevel + 4) type = 3; // Stein
                    world[x][y] = type;
                }
            }
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Welt zeichnen
            for (let x = 0; x < cols; x++) {
                for (let y = 0; y < rows; y++) {
                    if (world[x][y] === 0) continue;
                    
                    if (world[x][y] === 1) ctx.fillStyle = '#4CAF50'; // Gras
                    if (world[x][y] === 2) ctx.fillStyle = '#8B4513'; // Erde
                    if (world[x][y] === 3) ctx.fillStyle = '#808080'; // Stein
                    
                    ctx.fillRect(x * blockSize, y * blockSize, blockSize, blockSize);
                    ctx.strokeStyle = 'rgba(0,0,0,0.1)';
                    ctx.strokeRect(x * blockSize, y * blockSize, blockSize, blockSize);
                }
            }

            // Spieler zeichnen
            ctx.fillStyle = '#FF0000';
            ctx.fillRect(player.x, player.y, player.w, player.h);

            update();
            requestAnimationFrame(draw);
        }

        function update() {
            player.dy += player.g;
            player.y += player.dy;

            // Einfache Boden-Kollision (auf Gras-Ebene)
            const pCol = Math.floor((player.x + player.w/2) / blockSize);
            const pRow = Math.floor((player.y + player.h) / blockSize);

            if (world[pCol] && world[pCol][pRow] !== 0) {
                player.y = pRow * blockSize - player.h;
                player.dy = 0;
            }
        }

        // Interaktion: Block abbauen
        canvas.addEventListener('touchstart', (e) => {
            const touch = e.touches[0];
            const bX = Math.floor(touch.clientX / blockSize);
            const bY = Math.floor(touch.clientY / blockSize);
            
            if (world[bX] && world[bX][bY] !== undefined) {
                world[bX][bY] = 0; // Block wird zu Luft
            }
            
            // Sprung-Check (wenn linke Bildschirmhälfte getippt wird)
            if (touch.clientX < window.innerWidth / 2 && player.dy === 0) {
                player.dy = player.jump;
            }
        });

        window.addEventListener('resize', () => {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        });

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        initWorld();
        draw();
    </script>
</body>
</html>
