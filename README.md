<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>BlockCraft iPad Pro</title>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; touch-action: none; font-family: sans-serif; }
        canvas { display: block; }
        #controls-hint { 
            position: absolute; top: 10px; width: 100%; text-align: center; 
            color: white; text-shadow: 1px 1px 2px black; pointer-events: none; 
        }
    </style>
</head>
<body>
    <div id="controls-hint">Links halten: Laufen/Springen | Rechts tippen: Abbauen</div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let w = canvas.width = window.innerWidth;
        let h = canvas.height = window.innerHeight;
        const bSize = 40; 

        // Welt & Kamera
        let world = {};
        let cameraX = 0;
        const player = { x: 400, y: 0, vx: 0, vy: 0, w: 30, h: 55, g: 0.6, jump: -13 };

        // Block-Typen
        const COLORS = { 1: '#4CAF50', 2: '#8B4513', 3: '#707070', 4: '#5D4037', 5: '#2E7D32' }; // Gras, Erde, Stein, Holz, Laub

        function generateWorld() {
            for (let x = 0; x < 200; x++) {
                // Hügel-Berechnung
                let groundHeight = 10 + Math.floor(Math.sin(x * 0.2) * 2); 
                
                for (let y = 0; y < 25; y++) {
                    let type = 0;
                    if (y > groundHeight) type = 2; // Erde
                    if (y === groundHeight) type = 1; // Gras
                    if (y > groundHeight + 5) type = 3; // Stein
                    world[`${x},${y}`] = type;
                }

                // Bäume spawnen (alle 8-12 Blöcke)
                if (x % 10 === 0 && Math.random() > 0.3) {
                    let gY = groundHeight;
                    // Stamm
                    world[`${x},${gY-1}`] = 4; world[`${x},${gY-2}`] = 4; world[`${x},${gY-3}`] = 4;
                    // Blätter
                    world[`${x-1},${gY-4}`] = 5; world[`${x},${gY-4}`] = 5; world[`${x+1},${gY-4}`] = 5;
                    world[`${x-1},${gY-5}`] = 5; world[`${x},${gY-5}`] = 5; world[`${x+1},${gY-5}`] = 5;
                    world[`${x},${gY-6}`] = 5;
                }
            }
        }

        function draw() {
            ctx.fillStyle = "#87CEEB"; // Himmel
            ctx.fillRect(0, 0, w, h);

            ctx.save();
            ctx.translate(-cameraX, 0);

            // Welt zeichnen (nur was im Bild ist)
            let startX = Math.floor(cameraX / bSize);
            let endX = startX + Math.ceil(w / bSize) + 1;

            for (let x = startX; x < endX; x++) {
                for (let y = 0; y < 30; y++) {
                    let type = world[`${x},${y}`];
                    if (type && type !== 0) {
                        ctx.fillStyle = COLORS[type];
                        ctx.fillRect(x * bSize, y * bSize, bSize, bSize);
                        ctx.strokeStyle = "rgba(0,0,0,0.05)";
                        ctx.strokeRect(x * bSize, y * bSize, bSize, bSize);
                    }
                }
            }

            // Spieler
            ctx.fillStyle = "#3498db";
            ctx.fillRect(player.x, player.y, player.w, player.h);
            ctx.fillStyle = "#ffdbac";
            ctx.fillRect(player.x + 5, player.y - 15, 20, 20); // Kopf

            ctx.restore();

            update();
            requestAnimationFrame(draw);
        }

        function update() {
            player.vy += player.g;
            player.x += player.vx;
            player.y += player.vy;

            // Kamera folgt Spieler
            cameraX += (player.x - cameraX - w/2) * 0.1;

            // Einfache Kollision
            let pX = Math.floor((player.x + player.w/2) / bSize);
            let pY = Math.floor((player.y + player.h) / bSize);
            if (world[`${pX},${pY}`]) {
                player.y = pY * bSize - player.h;
                player.vy = 0;
            }
        }

        // iPad Steuerung
        window.addEventListener('touchstart', (e) => {
            const t = e.touches[0];
            const touchX = t.clientX;
            const worldX = Math.floor((t.clientX + cameraX) / bSize);
            const worldY = Math.floor(t.clientY / bSize);

            if (touchX < w / 3) { // Links tippen: Rückwärts
                player.vx = -5;
            } else if (touchX < (w / 3) * 2) { // Mitte: Springen
                if (player.vy === 0) player.vy = player.jump;
            } else { // Rechts: Abbauen
                world[`${worldX},${worldY}`] = 0;
            }
        });

        window.addEventListener('touchend', () => { player.vx = 0; });

        generateWorld();
        draw();
    </script>
</body>
</html>
