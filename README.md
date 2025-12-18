<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pi Catch Extreme</title>
    <style>
        body { margin: 0; background: #2a0845; overflow: hidden; touch-action: none; font-family: 'Arial', sans-serif; }
        #game-ui { position: absolute; top: 10px; width: 100%; display: flex; justify-content: space-around; color: #fff; font-size: 18px; z-index: 10; pointer-events: none; }
        .stat { background: rgba(0,0,0,0.5); padding: 5px 15px; border-radius: 20px; border: 1px solid gold; }
        #msg-screen { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; background: rgba(42, 8, 69, 0.9); z-index: 20; color: white; text-align: center; }
        button { padding: 15px 40px; font-size: 20px; background: gold; border: none; border-radius: 50px; cursor: pointer; font-weight: bold; color: #2a0845; transition: 0.3s; }
        button:active { transform: scale(0.9); }
        .heart { color: red; font-size: 25px; }
    </style>
</head>
<body>
    <div id="game-ui">
        <div class="stat">النقاط: <span id="score">0</span></div>
        <div class="stat"><span id="lives">❤️❤️❤️</span></div>
    </div>

    <div id="msg-screen">
        <h1 id="title">Pi Catch Extreme</h1>
        <p id="desc">التقط عملات باي وكن الأفضل في الشبكة!</p>
        <button id="startBtn">ابدأ التحدي</button>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('score');
        const livesEl = document.getElementById('lives');
        const startBtn = document.getElementById('startBtn');
        const msgScreen = document.getElementById('msg-screen');

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let score = 0;
        let lives = 3;
        let gameActive = false;
        let coins = [];
        let basket = { x: canvas.width / 2 - 50, y: canvas.height - 100, w: 100, h: 15 };
        
        // أصوات (تأكد من تشغيل الصوت في هاتفك)
        const sndCollect = new Audio('https://assets.mixkit.co/active_storage/sfx/2019/2019-preview.mp3');
        const sndLose = new Audio('https://assets.mixkit.co/active_storage/sfx/2018/2018-preview.mp3');

        window.addEventListener('touchmove', (e) => {
            if (gameActive) basket.x = e.touches[0].clientX - basket.w / 2;
        });

        function createCoin() {
            if (!gameActive) return;
            let isBonus = Math.random() > 0.9; // 10% احتمال عملة بونص
            coins.push({
                x: Math.random() * (canvas.width - 40) + 20,
                y: -30,
                speed: (3 + Math.random() * 4) + (score / 100), // تزداد السرعة مع النقاط
                type: isBonus ? 'bonus' : 'normal',
                color: isBonus ? '#00ffcc' : 'gold'
            });
            let nextSpawn = Math.max(300, 1000 - (score * 2)); // يزداد معدل الظهور
            setTimeout(createCoin, nextSpawn);
        }

        function draw() {
            if (!gameActive) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // رسم السلة (تتغير حسب النقاط)
            ctx.fillStyle = "gold";
            ctx.shadowBlur = 15; ctx.shadowColor = "gold";
            ctx.fillRect(basket.x, basket.y, basket.w, basket.h);
            ctx.shadowBlur = 0;

            coins.forEach((coin, i) => {
                coin.y += coin.speed;
                
                // رسم العملة
                ctx.beginPath();
                ctx.arc(coin.x, coin.y, 20, 0, Math.PI * 2);
                ctx.fillStyle = coin.color;
                ctx.fill();
                ctx.fillStyle = "#2a0845";
                ctx.font = "bold 16px Arial";
                ctx.textAlign = "center";
                ctx.fillText("π", coin.x, coin.y + 6);

                // فحص الاصطدام
                if (coin.y + 20 > basket.y && coin.x > basket.x && coin.x < basket.x + basket.w) {
                    sndCollect.play().catch(()=>{});
                    score += coin.type === 'bonus' ? 50 : 10;
                    scoreEl.innerText = score;
                    coins.splice(i, 1);
                }

                // فحص السقوط
                if (coin.y > canvas.height) {
                    coins.splice(i, 1);
                    if (coin.type === 'normal') { // البونص لا ينقص الأرواح
                        lives--;
                        livesEl.innerText = "❤️".repeat(lives);
                        sndLose.play().catch(()=>{});
                        if (lives <= 0) endGame();
                    }
                }
            });
            requestAnimationFrame(draw);
        }

        function endGame() {
            gameActive = false;
            msgScreen.style.display = 'flex';
            document.getElementById('title').innerText = "Game Over!";
            document.getElementById('desc').innerText = "لقد جمعت " + score + " من نقاط باي";
            startBtn.innerText = "حاول مرة أخرى";
        }

        startBtn.onclick = () => {
            score = 0; lives = 3; coins = [];
            scoreEl.innerText = score; livesEl.innerText = "❤️❤️❤️";
            gameActive = true;
            msgScreen.style.display = 'none';
            createCoin();
            draw();
        };
    </script>
</body>
</html>
