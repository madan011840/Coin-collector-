# Coin-collector-
The best game for coin collector 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Coin Collector Game</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: sans-serif;
      background: #111;
      color: #fff;
    }
    #game {
      position: relative;
      width: 100vw;
      height: 100vh;
      background: linear-gradient(to bottom, #333, #111);
    }
    .player {
      position: absolute;
      bottom: 20px;
      left: 50%;
      width: 50px;
      height: 50px;
      background: gold;
      border-radius: 50%;
      transform: translateX(-50%);
    }
    .coin {
      position: absolute;
      top: 0;
      width: 20px;
      height: 20px;
      background: yellow;
      border-radius: 50%;
    }
    .score, .lives, .high-score {
      position: absolute;
      font-size: 20px;
    }
    .score { top: 10px; left: 10px; }
    .lives { top: 10px; right: 10px; color: lightcoral; }
    .high-score { top: 40px; left: 10px; color: lightgreen; }

    .game-over {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      font-size: 40px;
      color: red;
      text-align: center;
      display: none;
      background: rgba(0,0,0,0.7);
      padding: 20px;
      border-radius: 10px;
    }
    .button {
      background: gold;
      color: black;
      border: none;
      padding: 10px 20px;
      font-size: 20px;
      margin-top: 10px;
      border-radius: 8px;
      cursor: pointer;
    }
    #startScreen {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.8);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      z-index: 10;
    }
  </style>
</head>
<body>
  <div id="game">
    <div class="player" id="player"></div>
    <div class="score" id="score">Score: 0</div>
    <div class="lives" id="lives">Lives: 3</div>
    <div class="high-score" id="highScore">High Score: 0</div>
    <div id="startScreen">
      <h1>Coin Collector</h1>
      <button class="button" onclick="startGame()">Start Game</button>
    </div>
    <div class="game-over" id="gameOver">
      Game Over!<br/>
      <button class="button" onclick="restartGame()">Restart</button>
    </div>
  </div>

  <audio id="bgMusic" src="https://cdn.pixabay.com/audio/2022/03/15/audio_b6e65bdf47.mp3" loop></audio>
  <audio id="coinSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_8a55b9d7c6.mp3"></audio>
  <audio id="gameOverSound" src="https://cdn.pixabay.com/audio/2022/03/14/audio_96ba23c3e0.mp3"></audio>

  <script>
    const player = document.getElementById("player");
    const game = document.getElementById("game");
    const scoreEl = document.getElementById("score");
    const livesEl = document.getElementById("lives");
    const highScoreEl = document.getElementById("highScore");
    const gameOverEl = document.getElementById("gameOver");
    const startScreen = document.getElementById("startScreen");

    const bgMusic = document.getElementById("bgMusic");
    const coinSound = document.getElementById("coinSound");
    const gameOverSound = document.getElementById("gameOverSound");

    let score = 0;
    let lives = 3;
    let highScore = localStorage.getItem("coinHighScore") || 0;
    let playerX = window.innerWidth / 2;
    let gameActive = false;

    let coinCreationInterval;
    let coinFallIntervals = [];

    highScoreEl.textContent = "High Score: " + highScore;

    document.addEventListener("keydown", (e) => {
      if (!gameActive) return;
      if (e.key === "ArrowLeft") {
        playerX -= 20;
      } else if (e.key === "ArrowRight") {
        playerX += 20;
      }
      playerX = Math.max(0, Math.min(window.innerWidth - 50, playerX));
      player.style.left = playerX + "px";
    });

    function createCoin() {
      const coin = document.createElement("div");
      coin.classList.add("coin");
      coin.style.left = Math.random() * (window.innerWidth - 20) + "px";
      game.appendChild(coin);
      let y = 0;

      const fall = setInterval(() => {
        if (!gameActive) {
          clearInterval(fall);
          return;
        }
        y += 5;
        coin.style.top = y + "px";

        const coinX = coin.offsetLeft;
        const coinY = coin.offsetTop;
        const playerY = player.offsetTop;

        if (
          y + 20 > playerY &&
          coinX > playerX - 10 &&
          coinX < playerX + 50
        ) {
          score++;
          coinSound.play();
          scoreEl.textContent = "Score: " + score;
          if (score > highScore) {
            highScore = score;
            highScoreEl.textContent = "High Score: " + highScore;
            localStorage.setItem("coinHighScore", highScore);
          }
          game.removeChild(coin);
          clearInterval(fall);
          coinFallIntervals = coinFallIntervals.filter(i => i !== fall);
        } else if (y > window.innerHeight) {
          game.removeChild(coin);
          clearInterval(fall);
          coinFallIntervals = coinFallIntervals.filter(i => i !== fall);
          lives--;
          livesEl.textContent = "Lives: " + lives;
          if (lives <= 0) {
            endGame();
          }
        }
      }, 30);
      coinFallIntervals.push(fall);
    }

    function startGame() {
      gameActive = true;
      score = 0;
      lives = 3;
      playerX = window.innerWidth / 2;
      player.style.left = playerX + "px";
      scoreEl.textContent = "Score: 0";
      livesEl.textContent = "Lives: 3";
      highScoreEl.textContent = "High Score: " + highScore;
      gameOverEl.style.display = "none";
      startScreen.style.display = "none";
      bgMusic.play();

      coinCreationInterval = setInterval(createCoin, 1000);
    }

    function endGame() {
      gameActive = false;
      clearInterval(coinCreationInterval);
      coinFallIntervals.forEach(i => clearInterval(i));
      coinFallIntervals = [];
      document.querySelectorAll(".coin").forEach(coin => coin.remove());
      gameOverEl.style.display = "block";
      bgMusic.pause();
      gameOverSound.play();
    }

    function restartGame() {
      startGame();
    }
  </script>
</body>
</html>
