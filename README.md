<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CleanMant 30s Speed Challenge</title>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;900&display=swap" rel="stylesheet">
    <style>
        body { 
            text-align: center; 
            background-color: #000; 
            color: #fff;
            font-family: 'Orbitron', sans-serif;
            margin: 0; 
            overflow: hidden; 
        }

        h1.neon-title { 
            color: #fff;
            text-shadow: 0 0 10px #fff, 0 0 20px #00d2ff, 0 0 40px #00d2ff;
            font-size: 2.5em;
            margin: 10px 0 0 0;
        }

        .subtitle { color: #00d2ff; font-size: 1em; margin-top: 5px; text-transform: uppercase; }
        .instruction { color: #ccc; font-size: 0.8em; margin: 5px 0 10px 0; }

        .stats-container {
            display: flex;
            justify-content: center;
            gap: 30px;
            margin-bottom: 10px;
        }

        .score-box { color: #00ff88; text-shadow: 0 0 10px #00ff88; font-size: 1.2em; }
        .time-box { color: #ff00de; text-shadow: 0 0 10px #ff00de; font-size: 1.2em; }

        canvas { 
            background: radial-gradient(circle, #0a0a0a 0%, #000 100%);
            border: 2px solid #00d2ff; 
            box-shadow: 0 0 20px #00d2ff;
            display: block; 
            margin: 0 auto; 
            cursor: none;
        }

        #win-screen, #lose-screen { 
            display: none; 
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.95); 
            padding: 30px; border-radius: 20px; z-index: 100;
            width: 320px;
        }

        #win-screen { border: 3px solid #ff00de; box-shadow: 0 0 50px #ff00de; }
        #lose-screen { border: 3px solid #ff4444; box-shadow: 0 0 50px #ff4444; }

        .neon-win { 
            color: #fff; 
            text-shadow: 0 0 10px #fff, 0 0 20px #ff00de, 0 0 40px #ff00de;
            font-size: 2.2em; margin: 0;
        }

        .reward-text { color: #00ff88; margin: 15px 0; font-size: 0.9em; }

        button { 
            padding: 12px 25px; background: transparent; color: #ff00de; 
            border: 2px solid #ff00de; cursor: pointer; font-family: 'Orbitron';
            margin-top: 15px; text-transform: uppercase;
        }
        button:hover { background: #ff00de; color: white; }
    </style>
</head>
<body>

    <h1 class="neon-title">CLEANMANT</h1>
    <div class="subtitle">Mantenimiento preventivo y correctivo</div>
    <div class="instruction">Atrapa las reparaciones(🛠️) 	 Evita la Avería (⚠️)</div>
    
    <div class="stats-container">
        <div class="score-box">PUNTOS: <span id="score">0</span> / 200</div>
        <div class="time-box">TIEMPO: <span id="timer">30</span>s</div>
    </div>

    <canvas id="gameCanvas" width="400" height="480"></canvas>

    <div id="win-screen">
        <h2 class="neon-win">FELICIDADES</h2>
        <p class="reward-text">¡MANTENIMIENTO COMPLETADO AL 100%!</p>
        <p>Tu recompensa por habilidad extrema:</p>
        <p style="font-size: 1.8em; color: #fff; text-shadow: 0 0 10px #fff;">15% DESCUENTO</p>
        <p style="color: #666; font-size: 0.8em;">Código: EXPERT-CLEAN</p>
        <button onclick="location.reload()">REINTENTAR</button>
    </div>

    <div id="lose-screen">
        <h2 style="color: #ff4444;">TIEMPO AGOTADO</h2>
        <p>No lograste los 200 puntos.</p>
        <button onclick="location.reload()">REINTENTAR</button>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const scoreElement = document.getElementById("score");
        const timerElement = document.getElementById("timer");

        let score = 0;
        let timeLeft = 30; // Tiempo reducido a 30s
        let gameActive = true;
        let items = [];
        const player = { x: 175, y: 410, w: 60, h: 60 };

        const countdown = setInterval(() => {
            if (gameActive) {
                timeLeft--;
                timerElement.innerText = timeLeft;
                if (timeLeft <= 0) endGame(false);
            }
        }, 1000);

        canvas.addEventListener("mousemove", (e) => {
            const rect = canvas.getBoundingClientRect();
            player.x = (e.clientX - rect.left) - player.w / 2;
            if (player.x < 0) player.x = 0;
            if (player.x > canvas.width - player.w) player.x = canvas.width - player.w;
        });

        function createItem() {
            if (!gameActive) return;
            const rand = Math.random();
            let type, color, icon;

            if (rand > 0.35) { 
                type = "tool"; color = "#00ff88"; icon = "🛠️";
            } else if (rand > 0.15) { 
                type = "dust"; color = "#ffcc00"; icon = "⚠️";
            } else { 
                type = "fire"; color = "#ff4444"; icon = "🔥";
            }

            items.push({
                x: Math.random() * (canvas.width - 40),
                y: -50,
                type: type,
                icon: icon,
                color: color,
                speed: 5 + (score / 15) // Empiezan más rápido por el poco tiempo
            });
        }

        function endGame(victory) {
            gameActive = false;
            clearInterval(countdown);
            if (victory) {
                document.getElementById("win-screen").style.display = "block";
            } else {
                document.getElementById("lose-screen").style.display = "block";
            }
        }

        function draw() {
            if (!gameActive) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            ctx.shadowBlur = 15; ctx.shadowColor = "#00d2ff";
            ctx.font = "50px Arial";
            ctx.fillText("🔧", player.x, player.y + 45);
            ctx.shadowBlur = 0;

            for (let i = items.length - 1; i >= 0; i--) {
                let it = items[i];
                it.y += it.speed;

                ctx.shadowBlur = 10; ctx.shadowColor = it.color;
                ctx.font = "35px Arial";
                ctx.fillText(it.icon, it.x, it.y);

                if (it.y > player.y && it.y < player.y + player.h &&
                    it.x > player.x - 20 && it.x < player.x + player.w) {
                    
                    if (it.type === "tool") score += 10;
                    else if (it.type === "dust") score = Math.max(0, score - 10);
                    else if (it.type === "fire") score = Math.max(0, score - 30);

                    scoreElement.innerText = score;
                    items.splice(i, 1);
                }
                if (it.y > canvas.height + 50) items.splice(i, 1);
            }

            if (score >= 200) endGame(true);
            else requestAnimationFrame(draw);
        }

        setInterval(createItem, 450); // Salen mucho más seguido para poder llegar a 200 en 30s
        draw();
    </script>
</body>
</html>
