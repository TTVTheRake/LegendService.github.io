<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Anti Inspect 2D Car Game with Custom Right-Click Menu</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #1c1c1c;
            overflow: hidden;
            position: relative;
        }

        canvas {
            display: block;
            margin: 0 auto;
            background-color: #333;
            border: 2px solid #fff;
        }

        .game-over {
            font-size: 30px;
            color: white;
            text-align: center;
            margin-top: 20px;
        }

        .start-button {
            display: block;
            margin: 0 auto;
            padding: 10px 20px;
            background-color: #ffdd57;
            color: #1c1c1c;
            font-size: 20px;
            border: none;
            cursor: pointer;
            border-radius: 5px;
            transition: background 0.3s;
        }

        .start-button:hover {
            background-color: #ffc107;
        }

        /* Custom right-click menu */
        #customMenu {
            position: absolute;
            display: none;
            background-color: #333;
            color: white;
            border-radius: 5px;
            box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.6);
            z-index: 1000;
            padding: 10px;
            min-width: 150px;
        }

        #customMenu a {
            display: block;
            padding: 8px;
            color: white;
            text-decoration: none;
            font-size: 14px;
            transition: background 0.3s;
        }

        #customMenu a:hover {
            background-color: #555;
        }

        /* Prevent user from inspecting the page */
        .hide-elements {
            display: none;
        }

        /* Menu on Esc */
        #escMenu {
            display: none;
            text-align: center;
            color: white;
            margin-top: 20px;
        }

    </style>
</head>
<body>

<canvas id="gameCanvas" width="800" height="600" style="display:none;"></canvas>
<div id="gameOverMessage" class="game-over" style="display: none;">Game Over! Click Start to Play Again.</div>
<button id="startButton" class="start-button">Start Game</button>

<!-- Custom Right-Click Menu -->
<div id="customMenu">
    <a href="https://discord.gg/ZskTafJpY3" target="_blank">Join Discord</a>
    <a href="https://www.twitch.tv/therakeoh_official" target="_blank">Visit My Twitch</a>
    <a href="https://www.youtube.com/c/therakeoh" target="_blank">Visit My YouTube</a>
</div>

<!-- Escape Menu -->
<div id="escMenu">
    <h2>Game Paused</h2>
    <button id="resumeButton" class="start-button">Resume Game</button>
    <button id="removeMenuButton" class="start-button">Remove Menu</button>
</div>

<script>
    // Disable right-click context menu and show custom menu
    document.addEventListener('contextmenu', (event) => {
        event.preventDefault();
        const customMenu = document.getElementById('customMenu');
        customMenu.style.left = `${event.pageX}px`;
        customMenu.style.top = `${event.pageY}px`;
        customMenu.style.display = 'block';
    });

    // Close the custom menu when clicking anywhere on the page
    document.addEventListener('click', () => {
        const customMenu = document.getElementById('customMenu');
        customMenu.style.display = 'none';
    });

    // Disable F12 (dev tools) and Ctrl+Shift+I
    document.addEventListener('keydown', (event) => {
        if ((event.key === 'F12') || 
            (event.ctrlKey && (event.key === 'i' || event.key === 'I')) || 
            (event.ctrlKey && event.shiftKey && event.key === 'I') || 
            (event.ctrlKey && event.shiftKey && event.key === 'C') || 
            (event.ctrlKey && event.shiftKey && event.key === 'U')) {
            event.preventDefault();
            alert('Developer tools are disabled!');
        }
    });

    // Handle the Esc key
    document.addEventListener('keydown', (event) => {
        if (event.key === 'Escape' && isGameRunning) {
            pauseGame();
        }
    });

    // Car Game code
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const startButton = document.getElementById("startButton");
    const gameOverMessage = document.getElementById("gameOverMessage");
    const escMenu = document.getElementById("escMenu");
    const resumeButton = document.getElementById("resumeButton");
    const removeMenuButton = document.getElementById("removeMenuButton");

    const carWidth = 50;
    const carHeight = 80;
    let carX = canvas.width / 2 - carWidth / 2;
    let carY = canvas.height - carHeight - 20;
    let carSpeed = 5;
    let obstacles = [];
    let gameInterval;
    let score = 0;
    let isGameRunning = false;

    startButton.addEventListener("click", startGame);

    // Car control
    let leftKeyPressed = false;
    let rightKeyPressed = false;

    document.addEventListener("keydown", (e) => {
        if (e.key === "ArrowLeft" || e.key === "a") {
            leftKeyPressed = true;
        }
        if (e.key === "ArrowRight" || e.key === "d") {
            rightKeyPressed = true;
    }});

    document.addEventListener("keyup", (e) => {
        if (e.key === "ArrowLeft" || e.key === "a") {
            leftKeyPressed = false;
        }
        if (e.key === "ArrowRight" || e.key === "d") {
            rightKeyPressed = false;
        }
    });

    function startGame() {
        isGameRunning = true;
        score = 0;
        obstacles = [];
        carX = canvas.width / 2 - carWidth / 2;
        carY = canvas.height - carHeight - 20;
        gameOverMessage.style.display = "none";
        startButton.style.display = "none";
        canvas.style.display = "block";
        gameInterval = setInterval(gameLoop, 1000 / 60);
    }

    function gameLoop() {
        if (!isGameRunning) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        updateCarPosition();
        spawnObstacles();
        updateObstacles();
        drawObstacles();
        checkCollisions();
        drawCar();
        updateScore();
    }

    function updateCarPosition() {
        if (leftKeyPressed && carX > 0) {
            carX -= carSpeed;
        }
        if (rightKeyPressed && carX + carWidth < canvas.width) {
            carX += carSpeed;
        }
    }

    function drawCar() {
        ctx.fillStyle = "#ff0000";
        ctx.fillRect(carX, carY, carWidth, carHeight);
    }

    function spawnObstacles() {
        if (Math.random() < 0.02) {
            const obstacleWidth = 50 + Math.random() * 100;
            const obstacleX = Math.random() * (canvas.width - obstacleWidth);
            obstacles.push({
                x: obstacleX,
                y: -50,
                width: obstacleWidth,
                height: 30 + Math.random() * 50,
                speed: 3 + Math.random() * 2
            });
        }
    }

    function updateObstacles() {
        for (let i = 0; i < obstacles.length; i++) {
            const obstacle = obstacles[i];
            obstacle.y += obstacle.speed;
            if (obstacle.y > canvas.height) {
                obstacles.splice(i, 1);
                i--;
            }
        }
    }

    function drawObstacles() {
        ctx.fillStyle = "#00ff00";
        for (let obstacle of obstacles) {
            ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
        }
    }

    function checkCollisions() {
        for (let obstacle of obstacles) {
            if (
                carX < obstacle.x + obstacle.width &&
                carX + carWidth > obstacle.x &&
                carY < obstacle.y + obstacle.height &&
                carY + carHeight > obstacle.y
            ) {
                endGame();
            }
        }
    }

    function updateScore() {
        score++;
        ctx.fillStyle = "#fff";
        ctx.font = "20px Arial";
        ctx.fillText("Score: " + score, 20, 30);
    }

    function endGame() {
        clearInterval(gameInterval);
        isGameRunning = false;
        gameOverMessage.style.display = "block";
        startButton.style.display = "block";
        startButton.innerText = "Restart Game";
        canvas.style.display = "none";
    }

    // Pause game when Escape is pressed
    function pauseGame() {
        clearInterval(gameInterval);
        isGameRunning = false;
        escMenu.style.display = 'block';
    }

    // Resume game when Resume button is clicked
    resumeButton.addEventListener('click', () => {
        escMenu.style.display = 'none';
        startGame();
    });

    // Remove the menu when Remove Menu button is clicked
    removeMenuButton.addEventListener('click', () => {
        escMenu.style.display = 'none';
    });
</script>

</body>
</html>
