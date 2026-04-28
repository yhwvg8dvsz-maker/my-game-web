# my-game-web
<!DOCTYPE html>
<html>
<head>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        canvas {
            border: 2px solid #00FF9F;
            background: radial-gradient(circle, #2D0059 0%, #020205 100%);
            cursor: crosshair;
        }
    </style>
</head>
<body class="bg-[#020205] flex flex-col items-center justify-center min-h-screen overflow-hidden">

    <div class="mb-4 flex gap-10 text-[#00FF9F] font-black italic uppercase">
        <div class="text-2xl">Оноо: <span id="score">0</span></div>
        <div class="text-2xl text-[#FF4081]">Амь: <span id="lives">3</span></div>
    </div>

    <canvas id="gameCanvas" width="600" height="500"></canvas>

    <div class="mt-6 text-gray-500 text-sm animate-pulse">
        [A, D] Хөдлөх | [Space / Click] Буудах
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const scoreElement = document.getElementById("score");
        const livesElement = document.getElementById("lives");

        let score = 0;
        let lives = 3;
        let gameActive = true;

        const player = { x: 300, y: 450, w: 40, h: 40, speed: 7, color: "#00FF9F" };
        const bullets = [];
        const enemies = [];
        const keys = {};

        document.addEventListener("keydown", e => keys[e.code] = true);
        document.addEventListener("keyup", e => keys[e.code] = false);
        document.addEventListener("mousedown", () => shoot());

        function shoot() {
            if (!gameActive) return;
            bullets.push({ x: player.x + player.w/2 - 2, y: player.y, w: 4, h: 15, speed: 8 });
        }

        function spawnEnemy() {
            if (!gameActive) return;
            const size = 30 + Math.random() * 20;
            enemies.push({
                x: Math.random() * (canvas.width - size),
                y: -size,
                w: size,
                h: size,
                speed: 2 + Math.random() * 2,
                color: "#FF4081"
            });
        }
        setInterval(spawnEnemy, 1000);

        function update() {
            if (!gameActive) return;

            // Player movement
            if ((keys["ArrowLeft"] || keys["KeyA"]) && player.x > 0) player.x -= player.speed;
            if ((keys["ArrowRight"] || keys["KeyD"]) && player.x < canvas.width - player.w) player.x += player.speed;
            if (keys["Space"]) shoot();

            // Bullets
            bullets.forEach((b, i) => {
                b.y -= b.speed;
                if (b.y < 0) bullets.splice(i, 1);
            });

            // Enemies
            enemies.forEach((en, ei) => {
                en.y += en.speed;
                
                // Collision with player
                if (en.x < player.x + player.w && en.x + en.w > player.x && en.y < player.y + player.h && en.y + en.h > player.y) {
                    enemies.splice(ei, 1);
                    lives--;
                    livesElement.innerText = lives;
                    if (lives <= 0) {
                        gameActive = false;
                        alert("GAME OVER! Таны оноо: " + score);
                        location.reload();
                    }
                }

                // Collision with bullets
                bullets.forEach((b, bi) => {
                    if (b.x < en.x + en.w && b.x + b.w > en.x && b.y < en.y + en.h && b.y + b.h > en.y) {
                        enemies.splice(ei, 1);
                        bullets.splice(bi, 1);
                        score += 100;
                        scoreElement.innerText = score;
                    }
                });

                if (en.y > canvas.height) enemies.splice(ei, 1);
            });
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw Player (Neon Triangle)
            ctx.fillStyle = player.color;
            ctx.shadowBlur = 15;
            ctx.shadowColor = player.color;
            ctx.beginPath();
            ctx.moveTo(player.x + player.w/2, player.y);
            ctx.lineTo(player.x, player.y + player.h);
            ctx.lineTo(player.x + player.w, player.y + player.h);
            ctx.fill();

            // Draw Bullets
            ctx.fillStyle = "#FFFFFF";
            ctx.shadowColor = "#00FF9F";
            bullets.forEach(b => ctx.fillRect(b.x, b.y, b.w, b.h));

            // Draw Enemies
            enemies.forEach(en => {
                ctx.fillStyle = en.color;
                ctx.shadowColor = en.color;
                ctx.strokeRect(en.x, en.y, en.w, en.h);
                ctx.fillRect(en.x + 5, en.y + 5, en.w - 10, en.h - 10);
            });
            
            ctx.shadowBlur = 0;
        }

        function loop() {
            update();
            draw();
            requestAnimationFrame(loop);
        }

        loop();
    </script>
</body>
</html>
