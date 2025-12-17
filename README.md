<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Minecraft iPad Edition</title>
    <style>
        body { margin: 0; overflow: hidden; background: #0080ff; touch-action: none; font-family: 'Arial', sans-serif; }
        #gameCanvas { display: block; background: #87CEEB; }
        #ui { position: absolute; top: 10px; left: 10px; color: white; text-shadow: 2px 2px black; pointer-events: none; }
        .controls { position: absolute; bottom: 20px; right: 20px; display: flex; gap: 10px; }
        .btn { width: 60px; height: 60px; background: rgba(255,255,255,0.3); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; border: 2px solid white; }
    </style>
</head>
<body>
    <div id="ui">
        <b>BlockCraft 2D</b><br>
        Tippe Blöcke zum Abbauen<br>
        Tippe links zum Springen
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Fenster-Setup
        let w = canvas.width = window.innerWidth;
        let h = canvas.height = window.innerHeight;
        const bSize = 40; // Blockgröße

        // Welt-Daten
        let world = {}; // Wir nutzen ein Objekt für unendliche Welten
        const player = { x: 100, y: 100, vx: 0, vy: 0, w: 30, h: 50, speed: 4, jump: -12, g: 0.6 };

        function generateWorld() {
            for (let x = 0; x < Math.ceil(w / bSize) + 5; x++) {
                for (let y = 0; y < Math.ceil(h / bSize); y++) {
                    let type = 0; // Luft
                    if (y > 10) type = 2; // Erde
                    if (y === 10) type = 1; // Gras
                    if (y > 14) type = 3; // Stein
                    world[`${x},${y}`] = type;
                }
            }
        }

        function draw() {
            ctx.clearRect(0, 0, w, h);

            // Welt zeichnen
            for (let key in world) {
                let [x, y] = key.split(',').map(Number);
                let type = world[key];
                if (type === 0) continue;

                if (type === 1) ctx.fillStyle = "#45a049"; // Gras
                if (type === 2) ctx.fillStyle = "#8B4513"; // Erde
                if (type === 3) ctx.fillStyle = "#707070"; // Stein
                
                ctx.fillRect(x * bSize, y * bSize, bSize, bSize);
                ctx.strokeStyle = "rgba(0,0,0,0.1)";
                ctx.strokeRect(x * bSize, y * bSize, bSize, bSize);
            }

            // Spieler zeichnen (Steve)
            ctx.fillStyle = "#3498db"; // Blaues Shirt
            ctx.fillRect(player.x, player.y, player.w, player.h);
            ctx.fillStyle = "#ffdbac"; // Hautfarbe Kopf
            ctx.fillRect(player.x + 5, player.y - 15, 20, 20);

            update();
            requestAnimationFrame(draw);
        }

        function update() {
            player.vy += player.g;
            player.y += player.vy;

            // Einfache Kollision mit dem Boden
            let pX = Math.floor((player.x + player.w / 2) / bSize);
            let pY = Math.floor((player.y + player.h) / bSize);

            if (world[`${pX},${pY}`] && world[`${pX},${pY}`] !== 0) {
                player.y = pY * bSize - player.h;
                player.vy = 0;
            }
        }

        // Steuerung für iPad
        window.addEventListener('touchstart', (e) => {
            const t = e.touches[0];
            const bX = Math.floor(t.clientX / bSize);
            const bY = Math.floor(t.clientY / bSize);

            // Wenn links getippt wird: Springen
            if (t.clientX < w / 2) {
                if (player.vy === 0) player.vy = player.jump;
            } else {
                // Wenn rechts auf einen Block getippt wird: Abbauen
                world[`${bX},${bY}`] = 0;
            }
            e.preventDefault();
        }, { passive: false });

        generateWorld();
        draw();
    </script>
</body>
</html>
