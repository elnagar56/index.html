<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pi Catch Pro with Sounds</title>
    <style>
        body { margin: 0; background: radial-gradient(circle, #6a1b9a, #2a0845); overflow: hidden; touch-action: none; font-family: 'Segoe UI', sans-serif; }
        canvas { display: block; }
        #ui { position: absolute; top: 20px; width: 100%; display: flex; justify-content: space-around; color: gold; font-size: 20px; font-weight: bold; pointer-events: none; text-shadow: 2px 2px 4px rgba(0,0,0,0.5); }
        #msg { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); text-align: center; color: white; width: 80%; }
        button { padding: 15px 40px; font-size: 22px; background: linear-gradient(to bottom, #ffd700, #ff8c00); border: none; border-radius: 30px; cursor: pointer; font-weight: bold; color: #2a0845; box-shadow: 0 4px 15px rgba(0,0,0,0.4); }
    </style>
</head>
<body>
    <div id="ui">
        <div>النقاط: <span id="score">0</span></div>
        <div>أعلى نتيجة: <span id="highScore">0</span></div>
    </div>
    <div id="msg">
        <h1 id="title">تحدي عملة باي المطور</h1>
        <p>اجمع العملات الذهبية!</p>
        <button id="startBtn">ابدأ اللعب</button>
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startBtn = document.getElementById('startBtn');
        const scoreEl = document.getElementById('score');
        const highEl = document.getElementById('highScore');

        // تعريف الأصوات
        const collectSound = new Audio('https://assets.mixkit.co/active_storage/sfx/2019/2019-preview.mp3');
        const gameOverSound = new Audio('https://assets.mixkit.co/active_storage/sfx/2018/2018-preview.mp3');

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let score = 0;
        let highScore = localStorage.getItem('piHighScore') || 0;
        highEl.innerText = highScore;
        let gameActive = false;
        let speedMultiplier = 1;
        let basket = { x: canvas.width / 2 - 50, y: canvas.height - 120, w: 100, h: 18 };
        let coins = [];

        window.addEventListener('touchmove', (e) => {
            if (gameActive) {
                let touchX = e.touches[0].clientX;
                basket.x = touchX - basket.w / 2;
                if (basket.x < 0) basket.x = 0;
                if (basket.x > canvas.width - basket.w) basket.x = canvas.width - basket.w;
            }
        }, { passive: false });

        function spawnCoin() {
            if (!gameActive) return;
            speedMultiplier = 1 + (score / 300); 
            coins.push({
                x: Math.random() * (canvas.width - 40) + 20,
                y: -30,
                radius: 18,
                speed: (3.5 + Math.random() * 3) * speedMultiplier
            });
            setTimeout(spawnCoin, Math.max(350, 1000 - (score * 1.5)));
        }

        function update() {
            if (!gameActive) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // رسم السلة
            ctx.shadowBlur = 10; ctx.shadowColor = "gold";
            ctx.fillStyle = "gold";
            ctx.fillRect(basket.x, basket.y, basket.w, basket.h);
            ctx.shadowBlur = 0;

            coins.forEach((coin, i) => {
                coin.y += coin.speed;
                
                // رسم العملة
                ctx.beginPath();
                ctx.arc(coin.x, coin.y, coin.radius, 0, Math.PI * 2);
                ctx.fillStyle = "#FFD700";
                ctx.fill();
                ctx.strokeStyle = "white";
                ctx.lineWidth = 2;
                ctx.stroke();
                ctx.fillStyle = "#4a148c";
                ctx.font = "bold 18px Arial";
                ctx.textAlign = "center";
                ctx.fillText("π", coin.x, coin.y + 7);

                // التقاط العملة
                if (coin.y + coin.radius > basket.y && coin.x > basket.x && coin.x < basket.x + basket.w) {
                    coins.splice(i, 1);
                    score += 10;
                    scoreEl.innerText = score;
                    collectSound.play().catch(()=>{}); // تشغيل صوت الالتقاط
                }

                if (coin.y > canvas.height) {
                    gameOver();
                }
            });
            requestAnimationFrame(update);
        }

        function gameOver() {
            gameActive = false;
            gameOverSound.play().catch(()=>{}); // تشغيل صوت الخسارة
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('piHighScore', highScore);
                highEl.innerText = highScore;
            }
            document.getElementById('msg').style.display = 'block';
            document.getElementById('title').innerText = "انتهت اللعبة!";
            document.getElementById('title').innerHTML += `<br><span style="font-size:24px; color:gold;">نقاطك: ${score}</span>`;
            startBtn.innerText = "إعادة المحاولة";
            coins = [];
        }

        startBtn.onclick = () => {
            // تفعيل الصوت عند أول ضغطة (مطلب تقني للمتصفحات)
            collectSound.play().then(() => collectSound.pause()); 
            
            document.getElementById('msg').style.display = 'none';
            score = 0;
            scoreEl.innerText = 0;
            gameActive = true;
            spawnCoin();
            update();
        };
    </script>
</body>
</html>
