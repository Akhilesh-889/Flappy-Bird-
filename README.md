# Flappy-Bird-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Bird</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #87CEEB, #98D8E8);
            font-family: Arial, sans-serif;
            overflow: hidden;
        }
        canvas {
            border: 3px solid #333;
            background: linear-gradient(to bottom, #87CEEB 0%, #98D8E8 70%, #90EE90 100%);
            box-shadow: 0 0 20px rgba(0,0,0,0.3);
        }
        .score {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 32px;
            font-weight: bold;
            color: white;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            z-index: 10;
        }
        .game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            color: white;
            font-size: 24px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.8);
            display: none;
        }
        .restart-btn {
            margin-top: 20px;
            padding: 12px 24px;
            font-size: 18px;
            background: #FF6B6B;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 4px 8px rgba(0,0,0,0.3);
        }
        .restart-btn:hover {
            background: #FF5252;
        }
    </style>
</head>
<body>
    <div class="score" id="score">0</div>
    <canvas id="gameCanvas" width="400" height="600"></canvas>
    <div class="game-over" id="gameOver">
        <div>Game Over!</div>
        <div style="font-size: 18px; margin-top: 10px;">Final Score: <span id="finalScore">0</span></div>
        <button class="restart-btn" onclick="restartGame()">Play Again</button>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const gameOverElement = document.getElementById('gameOver');
        const finalScoreElement = document.getElementById('finalScore');

        // Game variables
        let gameRunning = true;
        let score = 0;
        let gravity = 0.5;
        let jumpPower = -10;
        let pipes = [];
        let pipeWidth = 60;
        let pipeGap = 180;
        let pipeSpeed = 2;

        // Bird object
        const bird = {
            x: 100,
            y: canvas.height / 2,
            width: 34,
            height: 24,
            velocity: 0,
            color: '#FFD700'
        };

        // Input handling
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' && gameRunning) {
                bird.velocity = jumpPower;
            }
        });

        canvas.addEventListener('click', () => {
            if (gameRunning) {
                bird.velocity = jumpPower;
            }
        });

        // Create pipe
        function createPipe() {
            const minHeight = 50;
            const maxHeight = canvas.height - pipeGap - minHeight;
            const topHeight = Math.random() * (maxHeight - minHeight) + minHeight;
            
            pipes.push({
                x: canvas.width,
                topHeight: topHeight,
                bottomY: topHeight + pipeGap,
                bottomHeight: canvas.height - (topHeight + pipeGap),
                passed: false
            });
        }

        // Update game logic
        function update() {
            if (!gameRunning) return;

            // Update bird
            bird.velocity += gravity;
            bird.y += bird.velocity;

            // Create pipes
            if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 200) {
                createPipe();
            }

            // Update pipes
            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].x -= pipeSpeed;

                // Score
                if (!pipes[i].passed && pipes[i].x + pipeWidth < bird.x) {
                    pipes[i].passed = true;
                    score++;
                    scoreElement.textContent = score;
                }

                // Remove off-screen pipes
                if (pipes[i].x + pipeWidth < 0) {
                    pipes.splice(i, 1);
                }
            }

            // Collision detection
            if (bird.y + bird.height > canvas.height || bird.y < 0) {
                gameOver();
            }

            for (let pipe of pipes) {
                if (bird.x < pipe.x + pipeWidth &&
                    bird.x + bird.width > pipe.x &&
                    (bird.y < pipe.topHeight || bird.y + bird.height > pipe.bottomY)) {
                    gameOver();
                }
            }
        }

        // Render game
        function render() {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw clouds (background)
            ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
            for (let i = 0; i < 5; i++) {
                const x = (Date.now() * 0.02 + i * 100) % (canvas.width + 100) - 50;
                const y = 50 + i * 30;
                ctx.beginPath();
                ctx.arc(x, y, 25, 0, Math.PI * 2);
                ctx.arc(x + 25, y, 30, 0, Math.PI * 2);
                ctx.arc(x + 50, y, 25, 0, Math.PI * 2);
                ctx.fill();
            }

            // Draw bird
            ctx.save();
            ctx.translate(bird.x + bird.width/2, bird.y + bird.height/2);
            ctx.rotate(Math.min(Math.max(bird.velocity * 0.1, -0.5), 0.5));
            ctx.fillStyle = bird.color;
            ctx.beginPath();
            ctx.arc(0, 0, bird.width/2, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = '#FFA500';
            ctx.fillRect(-bird.width/4, -bird.height/4, bird.width/2, bird.height/2);
            ctx.restore();

            // Draw pipes
            ctx.fillStyle = '#228B22';
            ctx.strokeStyle = '#006400';
            ctx.lineWidth = 4;
            for (let pipe of pipes) {
                // Top pipe
                ctx.fillRect(pipe.x, 0, pipeWidth, pipe.topHeight);
                ctx.strokeRect(pipe.x, 0, pipeWidth, pipe.topHeight);
                
                // Bottom pipe
                ctx.fillRect(pipe.x, pipe.bottomY, pipeWidth, pipe.bottomHeight);
                ctx.strokeRect(pipe.x, pipe.bottomY, pipeWidth, pipe.bottomHeight);
                
                // Pipe caps
                ctx.fillStyle = '#32CD32';
                ctx.fillRect(pipe.x - 5, pipe.topHeight - 25, pipeWidth + 10, 25);
                ctx.fillRect(pipe.x - 5, pipe.bottomY, pipeWidth + 10, 25);
            }
        }

        // Game loop
        function gameLoop() {
            update();
            render();
            requestAnimationFrame(gameLoop);
        }

        // Game over
        function gameOver() {
            gameRunning = false;
            finalScoreElement.textContent = score;
            gameOverElement.style.display = 'block';
        }

        // Restart game
        function restartGame() {
            gameRunning = true;
            score = 0;
            bird.y = canvas.height / 2;
            bird.velocity = 0;
            pipes = [];
            scoreElement.textContent = '0';
            gameOverElement.style.display = 'none';
        }

        // Start game
        gameLoop();
    </script>
</body>
</html>
