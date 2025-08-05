# Digital-marketer
Expert digital marketer
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Car Racing Game</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
        }
        
        .game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 20px;
        }
        
        .score-display {
            font-size: 24px;
            font-weight: bold;
            color: #333;
        }
        
        canvas {
            border: 3px solid #333;
            border-radius: 8px;
            background-color: #fff;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }
        
        .controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }
        
        .button-row {
            display: flex;
            gap: 10px;
        }
        
        button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: #45a049;
        }
        
        button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        
        .game-over {
            position: absolute;
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px;
            border-radius: 8px;
            text-align: center;
            display: none;
        }
        
        .mobile-controls {
            display: none;
            margin-top: 20px;
        }
        
        @media (max-width: 600px) {
            .mobile-controls {
                display: grid;
                grid-template-columns: repeat(3, 1fr);
                grid-template-rows: repeat(2, 1fr);
                gap: 10px;
            }
            
            .mobile-controls button {
                padding: 15px;
                font-size: 20px;
            }
            
            .mobile-up {
                grid-column: 2;
                grid-row: 1;
            }
            
            .mobile-left {
                grid-column: 1;
                grid-row: 2;
            }
            
            .mobile-right {
                grid-column: 3;
                grid-row: 2;
            }
            
            .mobile-down {
                grid-column: 2;
                grid-row: 2;
            }
            
            canvas {
                width: 90%;
                height: auto;
            }
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>Car Racing Game</h1>
        <div class="score-display">Score: <span id="score">0</span></div>
        <canvas id="gameCanvas" width="400" height="400"></canvas>
        
        <div class="controls">
            <button id="startBtn">Start Game</button>
            <button id="pauseBtn" disabled>Pause</button>
            <div class="mobile-controls">
                <button class="mobile-left">←</button>
                <button class="mobile-right">→</button>
            </div>
        </div>
        
        <div class="game-over" id="gameOver">
            <h2>Game Over!</h2>
            <p>Your final score: <span id="finalScore">0</span></p>
            <button id="restartBtn">Play Again</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');
        const finalScoreDisplay = document.getElementById('finalScore');
        const startBtn = document.getElementById('startBtn');
        const pauseBtn = document.getElementById('pauseBtn');
        const restartBtn = document.getElementById('restartBtn');
        const gameOverDisplay = document.getElementById('gameOver');
        
        // Mobile controls
        const upBtn = document.querySelector('.mobile-up');
        const leftBtn = document.querySelector('.mobile-left');
        const rightBtn = document.querySelector('.mobile-right');
        const downBtn = document.querySelector('.mobile-down');
        
        const gridSize = 20;
        const tileCount = canvas.width / gridSize;
        
        let playerCar = {x: 190, y: 340, width: 30, height: 60};
        let obstacles = [];
        let roadMarkers = [];
        let roadOffset = 0;
        let score = 0;
        let gameSpeed = 100;
        let speedIncrease = 0;
        let gameRunning = false;
        let gamePaused = false;
        let score = 0;
        let gameSpeed = 150;
        let gameLoop;
        
        // Initialize game
        function initGame() {
            snake = [
                {x: 10, y: 10},
                {x: 9, y: 10},
                {x: 8, y: 10}
            ];
            
            spawnFood();
            direction = 'right';
            nextDirection = 'right';
            score = 0;
            scoreDisplay.textContent = score;
            gameOverDisplay.style.display = 'none';
        }
        
        // Spawn food at random location
        function spawnFood() {
            food = {
                x: Math.floor(Math.random() * tileCount),
                y: Math.floor(Math.random() * tileCount)
            };
            
            // Make sure food doesn't spawn on snake
            for (let i = 0; i < snake.length; i++) {
                if (snake[i].x === food.x && snake[i].y === food.y) {
                    spawnFood();
                    return;
                }
            }
        }
        
        // Draw game elements
        function drawGame() {
            // Clear canvas
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw road
            ctx.fillStyle = '#555555';
            ctx.fillRect(100, 0, 200, canvas.height);
            
            // Draw road markers
            ctx.fillStyle = '#ffffff';
            for (let i = 0; i < roadMarkers.length; i++) {
                ctx.fillRect(roadMarkers[i].x, roadMarkers[i].y, 20, 40);
            }
            
            // Draw player car
            ctx.fillStyle = '#FF0000';
            ctx.fillRect(playerCar.x, playerCar.y, playerCar.width, playerCar.height);
            
            // Draw obstacles
            ctx.fillStyle = '#0000FF';
            for (let i = 0; i < obstacles.length; i++) {
                ctx.fillRect(obstacles[i].x, obstacles[i].y, obstacles[i].width, obstacles[i].height);
            }
            
            // Draw grid (optional)
            ctx.strokeStyle = '#e0e0e0';
            ctx.lineWidth = 0.5;
            for (let i = 0; i < tileCount; i++) {
                ctx.beginPath();
                ctx.moveTo(i * gridSize, 0);
                ctx.lineTo(i * gridSize, canvas.height);
                ctx.stroke();
                
                ctx.beginPath();
                ctx.moveTo(0, i * gridSize);
                ctx.lineTo(canvas.width, i * gridSize);
                ctx.stroke();
            }
        }
        
        // Game loop
        function updateGame() {
            if (gamePaused) return;
            
            direction = nextDirection;
            
            // Calculate new head position
            const head = {x: snake[0].x, y: snake[0].y};
            
            switch (e.key) {
                case 'ArrowLeft':
                    if (playerCar.x > 100) playerCar.x -= 10;
                    e.preventDefault();
                    break;
                case 'ArrowRight':
                    if (playerCar.x < 270) playerCar.x += 10;
                    e.preventDefault();
                    break;
            }
            
            // Check collision with walls
            if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
                gameOver();
                return;
            }
            
            // Check collision with self
            for (let i = 0; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    gameOver();
                    return;
                }
            }
            
            // Check if snake eats food
            if (head.x === food.x && head.y === food.y) {
                spawnFood();
                score++;
                scoreDisplay.textContent = score;
                
                // Increase speed every 5 points
                if (score % 5 === 0 && gameSpeed > 80) {
                    gameSpeed -= 5;
                    clearInterval(gameLoop);
                    gameLoop = setInterval(updateGame, gameSpeed);
                }
            } else {
                // Remove tail if no food eaten
                snake.pop();
            }
            
            // Add new head
            snake.unshift(head);
            
            drawGame();
        }
        
        // Game over function
        function gameOver() {
            clearInterval(gameLoop);
            gameRunning = false;
            gamePaused = false;
            finalScoreDisplay.textContent = score;
            gameOverDisplay.style.display = 'block';
            startBtn.textContent = 'Start Game';
            pauseBtn.disabled = true;
            pauseBtn.textContent = 'Pause';
        }
        
        // Event listeners for keyboard controls
        document.addEventListener('keydown', function(e) {
            switch (e.key) {
                case 'ArrowUp':
                    if (direction !== 'down') nextDirection = 'up';
                    e.preventDefault();
                    break;
                case 'ArrowDown':
                    if (direction !== 'up') nextDirection = 'down';
                    e.preventDefault();
                    break;
                case 'ArrowLeft':
                    if (direction !== 'right') nextDirection = 'left';
                    e.preventDefault();
                    break;
                case 'ArrowRight':
                    if (direction !== 'left') nextDirection = 'right';
                    e.preventDefault();
                    break;
                case ' ':
                    if (gameRunning) {
                        togglePause();
                    }
                    e.preventDefault();
                    break;
            }
        });
        
        // Mobile control event listeners
        upBtn.addEventListener('click', function() {
            if (direction !== 'down') nextDirection = 'up';
        });
        
        downBtn.addEventListener('click', function() {
            if (direction !== 'up') nextDirection = 'down';
        });
        
        leftBtn.addEventListener('click', function() {
            if (direction !== 'right') nextDirection = 'left';
        });
        
        rightBtn.addEventListener('click', function() {
            if (direction !== 'left') nextDirection = 'right';
        });
        
        // Toggle pause
        function togglePause() {
            gamePaused = !gamePaused;
            pauseBtn.textContent = gamePaused ? 'Resume' : 'Pause';
        }
        
        // Start game button
        startBtn.addEventListener('click', function() {
            if (!gameRunning) {
                initGame();
                gameRunning = true;
                gamePaused = false;
                gameSpeed = 150;
                startBtn.textContent = 'Restart';
                pauseBtn.disabled = false;
                pauseBtn.textContent = 'Pause';
                gameLoop = setInterval(updateGame, gameSpeed);
                drawGame();
            } else {
                clearInterval(gameLoop);
                gameRunning = false;
                initGame();
                gameRunning = true;
                gamePaused = false;
                gameSpeed = 150;
                gameLoop = setInterval(updateGame, gameSpeed);
                drawGame();
            }
        });
        
        // Pause button
        pauseBtn.addEventListener('click', togglePause);
        
        // Restart button
        restartBtn.addEventListener('click', function() {
            gameOverDisplay.style.display = 'none';
            clearInterval(gameLoop);
            gameRunning = false;
            startBtn.click();
        });
        
        // Initial draw
        drawGame();
    </script>
</body>
</html>

