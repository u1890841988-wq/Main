<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Pulse - GitHub Edition</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Segoe UI', sans-serif; touch-action: none; }
        canvas { display: block; }
        #ui {
            position: absolute;
            top: env(safe-area-inset-top, 20px);
            left: 50%;
            transform: translateX(-50%);
            color: #0ff;
            text-align: center;
            pointer-events: none;
            z-index: 10;
        }
        #score { font-size: 48px; font-weight: bold; text-shadow: 0 0 20px #0ff; }
        #best { font-size: 14px; opacity: 0.7; letter-spacing: 2px; }
    </style>
</head>
<body>

    <div id="ui">
        <div id="score">0</div>
        <div id="best">BEST: 0</div>
    </div>
    <canvas id="gameCanvas"></canvas>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreElement = document.getElementById('score');
    const bestElement = document.getElementById('best');

    function resize() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    // Game State
    let gravity = 0.8;
    let gameSpeed = 8;
    let score = 0;
    let bestScore = localStorage.getItem('neonPulseBest') || 0;
    bestElement.innerText = "BEST: " + bestScore;
    let gameActive = true;
    let frameCount = 0;

    const player = {
        x: 120, y: 0, size: 42, dy: 0, 
        jumpForce: -15, grounded: false, rotation: 0
    };

    let obstacles = [];

    function spawn() {
        const groundY = canvas.height - 100;
        const rand = Math.random();
        
        if (rand < 0.6) {
            // PLATTFORM
            const h = 130 + Math.random() * 100;
            const w = 180 + Math.random() * 100;
            obstacles.push({ x: canvas.width, y: groundY - h, w: w, h: 35, type: 'platform' });
        } else {
            // SPIKE
            obstacles.push({ x: canvas.width, y: groundY, w: 45, h: 45, type: 'spike' });
        }
    }

    function reset() {
        if (score > bestScore) {
            bestScore = score;
            localStorage.setItem('neonPulseBest', bestScore);
            bestElement.innerText = "BEST: " + bestScore;
        }
        score = 0; obstacles = []; player.y = 100; player.dy = 0;
        gameSpeed = 8; gameActive = true; scoreElement.innerText = "0";
    }

    function update() {
        if (!gameActive) return;

        player.dy += gravity;
        player.y += player.dy;
        
        let onAnything = false;
        const groundY = canvas.height - 100;

        // Boden Check
        if (player.y + player.size > groundY) {
            player.y = groundY - player.size;
            player.dy = 0;
            onAnything = true;
        }

        // Hindernisse
        frameCount++;
        if (frameCount % 65 === 0) spawn();

        for (let i = obstacles.length - 1; i >= 0; i--) {
            let o = obstacles[i];
            o.x -= gameSpeed;

            if (o.type === 'platform') {
                // Präzises Landen
                const isAbove = player.y + player.size <= o.y + 20;
                const isFalling = player.dy >= 0;
                const isWithinX = player.x + player.size > o.x + 10 && player.x < o.x + o.w - 10;

                if (isFalling && isAbove && isWithinX && player.y + player.size + player.dy >= o.y) {
