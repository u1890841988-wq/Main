<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body { margin: 0; background: #000d1a; overflow: hidden; touch-action: none; }
        canvas { display: block; }
        #ui { position: absolute; top: 20px; width: 100%; text-align: center; color: white; font-family: Arial; font-size: 30px; pointer-events: none; }
    </style>
</head>
<body>
    <div id="ui">Punkte: <span id="s">0</span></div>
    <canvas id="c"></canvas>
    <script>
        const canvas = document.getElementById('c');
        const ctx = canvas.getContext('2d');
        let w = canvas.width = window.innerWidth;
        let h = canvas.height = window.innerHeight;
        let score = 0, speed = 8, active = true, obstacles = [];
        let player = { x: 80, y: h-150, s: 40, dy: 0, g: 0.8, jump: -15, grounded: false };

        function reset() { score = 0; obstacles = []; player.dy = 0; active = true; document.getElementById('s').innerText = "0"; }

        function loop() {
            if (!active) return requestAnimationFrame(loop);
            ctx.fillStyle = "#000d1a"; ctx.fillRect(0,0,w,h);
            
            // Boden
            ctx.fillStyle = "#0ff"; ctx.fillRect(0, h-100, w, 2);

            // Spieler
            player.dy += player.g; player.y += player.dy;
            if (player.y > h-140) { player.y = h-140; player.dy = 0; player.grounded = true; }
            ctx.fillStyle = "#0ff"; ctx.fillRect(player.x, player.y, player.s, player.s);

            // Hindernisse
            if (Math.random() < 0.02) obstacles.push({x: w, y: h-140, w: 40, h: 40});
            for (let i = obstacles.length-1; i >= 0; i--) {
                let o = obstacles[i]; o.x -= speed;
                ctx.fillStyle = "#f0f"; ctx.fillRect(o.x, o.y, o.w, o.h);
                
                // Kollision
                if (player.x < o.x + o.w && player.x + player.s > o.x && player.y + player.s > o.y) {
                    active = false; setTimeout(reset, 1000);
                }
                if (o.x + o.w < 0) { obstacles.splice(i, 1); score++; document.getElementById('s').innerText = score; }
            }
            requestAnimationFrame(loop);
        }

        window.addEventListener('touchstart', (e) => { 
            if (player.grounded) { player.dy = player.jump; player.grounded = false; }
            e.preventDefault();
        }, {passive: false});

        loop();
    </script>
</body>
</html>

