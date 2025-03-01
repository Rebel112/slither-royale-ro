# slither-royale-ro<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Slither Royale: Ro Edition 🐍</title>
    <style>
        body {
            text-align: center;
            font-family: 'Arial', sans-serif;
            background: linear-gradient(45deg, #1a1a1a, #333);
            color: white;
            overflow: hidden;
        }
        h1 {
            color: #ffcc00;
            text-shadow: 2px 2px 10px rgba(255, 204, 0, 0.7);
        }
        #game-container {
            margin: auto;
            max-width: 90vw;
            position: relative;
            border: 5px solid #ffcc00;
            box-shadow: 0px 0px 20px rgba(255, 204, 0, 0.8);
            border-radius: 10px;
        }
        canvas {
            background-color: black;
            display: block;
            margin: auto;
            width: 100%;
            height: auto;
        }
        #score {
            font-size: 22px;
            font-weight: bold;
            margin-top: 10px;
            text-shadow: 1px 1px 10px rgba(255, 255, 255, 0.6);
        }
        #leaderboard {
            margin-top: 20px;
            padding: 10px;
            background: #222;
            border-radius: 10px;
            width: 250px;
            margin: auto;
            text-align: left;
        }
        #leaderboard h2 {
            color: #ffcc00;
            text-align: center;
        }
        #leaderboard ol {
            padding-left: 20px;
        }
        #controls {
            margin-top: 10px;
        }
        button {
            padding: 12px;
            margin: 5px;
            font-size: 16px;
            cursor: pointer;
            border: none;
            border-radius: 5px;
        }
        #startBtn {
            background-color: #28a745;
            color: white;
        }
        #restartBtn {
            background-color: #dc3545;
            color: white;
            display: none;
        }
        #resetLeaderboardBtn {
            background-color: #007bff;
            color: white;
            margin-top: 10px;
        }
        input {
            padding: 8px;
            font-size: 16px;
            margin-right: 5px;
            border-radius: 5px;
            border: 1px solid #ffcc00;
            background-color: black;
            color: white;
        }
    </style>
</head>
<body>
    <h1>Slither Royale: Ro Edition 🐍</h1>
    <p>Use arrow keys or tap buttons to move. Eat the red squares to grow!</p>

   <div id="controls">
        <input type="text" id="playerName" placeholder="Enter your name">
        <button id="startBtn">Start Game</button>
        <button id="restartBtn">Restart Game</button>
    </div>

   <div id="game-container">
        <canvas id="gameCanvas"></canvas>
    </div>

  <p id="score">Score: 0</p> <!-- Leaderboard -->
    <div id="leaderboard">
        <h2>🏆 Leaderboard</h2>
        <ol id="leaderboardList"></ol>
        <button id="resetLeaderboardBtn">Reset Leaderboard</button>
    </div>

   <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const startBtn = document.getElementById("startBtn");
        const restartBtn = document.getElementById("restartBtn");
        const resetLeaderboardBtn = document.getElementById("resetLeaderboardBtn");
        const playerNameInput = document.getElementById("playerName");
        const scoreDisplay = document.getElementById("score");
        const leaderboardList = document.getElementById("leaderboardList");

        const box = 20;
        let snake, food, dx, dy, score, speed, gameInterval, playerName;
        let gameOver = false;

        function setCanvasSize() {
            canvas.width = Math.min(window.innerWidth * 0.9, 400);
            canvas.height = Math.min(window.innerWidth * 0.9, 400);
        }
        setCanvasSize();

        function initGame() {
            playerName = playerNameInput.value.trim() || "Player";
            alert(`Welcome to Slither Royale, ${playerName}! 🏆`);

            snake = [{ x: 200, y: 200 }];
            food = { x: Math.floor(Math.random() * (canvas.width / box)) * box, y: Math.floor(Math.random() * (canvas.height / box)) * box };
            dx = box;
            dy = 0;
            score = 0;
            speed = 150;
            gameOver = false;
            scoreDisplay.innerHTML = `Score: ${score}`;

            clearInterval(gameInterval);
            gameInterval = setInterval(draw, speed);
        }

        function draw() {
            if (gameOver) return;

            ctx.fillStyle = "black";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = "red";
            ctx.fillRect(food.x, food.y, box, box);

            ctx.fillStyle = "lime";
            snake.forEach(segment => ctx.fillRect(segment.x, segment.y, box, box));

            let newHead = { x: snake[0].x + dx, y: snake[0].y + dy };

            if (newHead.x < 0 || newHead.y < 0 || newHead.x >= canvas.width || newHead.y >= canvas.height) {
                endGame();
                return;
            }

            for (let segment of snake) {
                if (newHead.x === segment.x && newHead.y === segment.y) {
                    endGame();
                    return;
                }
            }

            if (newHead.x === food.x && newHead.y === food.y) {
                score += 10;
                scoreDisplay.innerHTML = `${playerName}'s Score: ${score}`;
                food = { x: Math.floor(Math.random() * (canvas.width / box)) * box, y: Math.floor(Math.random() * (canvas.height / box)) * box };
            } else {
                snake.pop();
            }

            snake.unshift(newHead);
        }

        function changeDirection(event) {
            const key = event.keyCode;
            if (key === 37 && dx === 0) { dx = -box; dy = 0; }
            else if (key === 38 && dy === 0) { dx = 0; dy = -box; }
            else if (key === 39 && dx === 0) { dx = box; dy = 0; }
            else if (key === 40 && dy === 0) { dx = 0; dy = box; }
        }

        function endGame() {
            clearInterval(gameInterval);
            gameOver = true;
            alert(`Game Over! ${playerName}'s final score: ${score}`);
            updateLeaderboard(playerName, score);
            restartBtn.style.display = "inline-block";
        }

        function updateLeaderboard(name, score) {
            let leaderboard = JSON.parse(localStorage.getItem("leaderboard")) || [];
            leaderboard.push({ name, score });
            leaderboard.sort((a, b) => b.score - a.score);
            leaderboard = leaderboard.slice(0, 5);
            localStorage.setItem("leaderboard", JSON.stringify(leaderboard));
            showLeaderboard();
        }

        function resetLeaderboard() {
            localStorage.removeItem("leaderboard");
            showLeaderboard();
        }

        function showLeaderboard() {
            let leaderboard = JSON.parse(localStorage.getItem("leaderboard")) || [];
            leaderboardList.innerHTML = leaderboard.map(entry => `<li>${entry.name}: ${entry.score}</li>`).join("");
        }

        resetLeaderboardBtn.addEventListener("click", resetLeaderboard);
        document.addEventListener("keydown", changeDirection);
        startBtn.addEventListener("click", initGame);
        restartBtn.addEventListener("click", initGame);
        showLeaderboard();
    </script>
</body>
</html>


