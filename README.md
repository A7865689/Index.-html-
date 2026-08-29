# Index.-html-
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width, initial-scale=1.0,
      maximum-scale=1.0, user-scalable=no">

<title>Runner Dash</title>

<style>
* {
    box-sizing: border-box;
    touch-action: none;
}

body {
    margin: 0;
    overflow: hidden;
    background: #111;
    font-family: Arial, sans-serif;
}

#game {
    width: 100vw;
    height: 100vh;
    position: relative;
    overflow: hidden;
    background: linear-gradient(
        #6ec6ff 0%,
        #dff6ff 42%,
        #55a630 42%,
        #55a630 100%
    );
}

/* Road */
#road {
    position: absolute;
    left: 8%;
    width: 84%;
    height: 100%;
    background: #444;
    transform: perspective(500px);
}

/* Lane lines */
.laneLine {
    position: absolute;
    width: 5px;
    height: 100%;
    background: repeating-linear-gradient(
        to bottom,
        white 0px,
        white 60px,
        transparent 60px,
        transparent 120px
    );
    opacity: .8;
}

.line1 {
    left: 33.33%;
}

.line2 {
    left: 66.66%;
}

/* Player */
#player {
    position: absolute;
    bottom: 100px;
    left: 50%;
    width: 48px;
    height: 70px;
    transform: translateX(-50%);
    background: #ff3344;
    border-radius: 20px 20px 12px 12px;
    z-index: 10;
    transition: left .15s;
}

#player::before {
    content: "";
    position: absolute;
    width: 32px;
    height: 32px;
    background: #ffd1a9;
    border-radius: 50%;
    top: -25px;
    left: 8px;
}

#player::after {
    content: "";
    position: absolute;
    width: 8px;
    height: 35px;
    background: #222;
    left: 20px;
    bottom: -25px;
}

/* Coin */
.coin {
    position: absolute;
    width: 28px;
    height: 28px;
    background: gold;
    border: 4px solid #d69e00;
    border-radius: 50%;
    z-index: 5;
}

/* Obstacle */
.obstacle {
    position: absolute;
    width: 48px;
    height: 48px;
    background: #202020;
    border-radius: 8px;
    z-index: 5;
}

/* Score */
#score {
    position: absolute;
    top: 15px;
    left: 15px;
    color: white;
    font-size: 22px;
    font-weight: bold;
    z-index: 20;
}

/* Menu */
#menu {
    position: absolute;
    inset: 0;
    background: rgba(0,0,0,.7);
    color: white;
    z-index: 50;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
}

#menu h1 {
    font-size: 42px;
    margin-bottom: 10px;
}

#menu button {
    padding: 15px 45px;
    border: 0;
    border-radius: 15px;
    font-size: 22px;
    background: orange;
}

/* Controls */
#controls {
    position: absolute;
    bottom: 20px;
    width: 100%;
    z-index: 30;
    display: flex;
    justify-content: center;
    gap: 10px;
}

.control {
    width: 70px;
    height: 55px;
    border: 0;
    border-radius: 15px;
    font-size: 25px;
    background: rgba(255,255,255,.8);
}
</style>
</head>

<body>

<div id="game">

    <div id="road">
        <div class="laneLine line1"></div>
        <div class="laneLine line2"></div>
    </div>

    <div id="score">Score: 0</div>

    <div id="player"></div>

    <div id="controls">
        <button class="control" onclick="left()">⬅️</button>
        <button class="control" onclick="jump()">⬆️</button>
        <button class="control" onclick="right()">➡️</button>
    </div>

    <div id="menu">
        <h1>🏃 Runner Dash</h1>
        <p>Coins collect karo aur obstacles se bacho!</p>
        <button onclick="startGame()">START GAME</button>
    </div>

</div>

<script>

let lane = 1;
let score = 0;
let speed = 6;
let running = false;
let jumping = false;

const player = document.getElementById("player");
const game = document.getElementById("game");

function positionPlayer() {

    const positions = ["25%", "50%", "75%"];

    player.style.left = positions[lane];
}

function left() {

    if (!running) return;

    if (lane > 0) {
        lane--;
        positionPlayer();
    }
}

function right() {

    if (!running) return;

    if (lane < 2) {
        lane++;
        positionPlayer();
    }
}

function jump() {

    if (!running || jumping) return;

    jumping = true;

    player.style.bottom = "230px";

    setTimeout(() => {

        player.style.bottom = "100px";
        jumping = false;

    }, 600);
}

function createCoin() {

    if (!running) return;

    const coin = document.createElement("div");

    coin.className = "coin";

    const positions = ["25%", "50%", "75%"];

    coin.style.left =
        positions[Math.floor(Math.random() * 3)];

    coin.style.top = "-40px";

    game.appendChild(coin);

    let y = -40;

    const timer = setInterval(() => {

        if (!running) {
            clearInterval(timer);
            coin.remove();
            return;
        }

        y += speed;

        coin.style.top = y + "px";

        if (y > innerHeight) {

            clearInterval(timer);
            coin.remove();
        }

        const p =
            player.getBoundingClientRect();

        const c =
            coin.getBoundingClientRect();

        if (
            p.left < c.right &&
            p.right > c.left &&
            p.top < c.bottom &&
            p.bottom > c.top
        ) {

            score += 10;

            document.getElementById("score")
                .innerText = "Score: " + score;

            clearInterval(timer);
            coin.remove();
        }

    }, 30);
}

function createObstacle() {

    if (!running) return;

    const obstacle =
        document.createElement("div");

    obstacle.className = "obstacle";

    const positions =
        ["25%", "50%", "75%"];

    obstacle.style.left =
        positions[Math.floor(Math.random() * 3)];

    obstacle.style.top = "-60px";

    game.appendChild(obstacle);

    let y = -60;

    const timer = setInterval(() => {

        if (!running) {

            clearInterval(timer);
            obstacle.remove();
            return;
        }

        y += speed;

        obstacle.style.top = y + "px";

        if (y > innerHeight) {

            clearInterval(timer);
            obstacle.remove();
        }

        const p =
            player.getBoundingClientRect();

        const o =
            obstacle.getBoundingClientRect();

        if (
            p.left < o.right &&
            p.right > o.left &&
            p.top < o.bottom &&
            p.bottom > o.top &&
            !jumping
        ) {

            gameOver();

            clearInterval(timer);
        }

    }, 30);
}

function startGame() {

    running = true;

    score = 0;
    lane = 1;
    speed = 6;

    positionPlayer();

    document.getElementById("score")
        .innerText = "Score: 0";

    document.getElementById("menu")
        .style.display = "none";

    setInterval(createCoin, 700);
    setInterval(createObstacle, 1100);

    setInterval(() => {

        if (running && speed < 14) {
            speed += .2;
        }

    }, 3000);
}

function gameOver() {

    running = false;

    document.getElementById("menu")
        .style.display = "flex";

    document.querySelector("#menu h1")
        .innerText = "💥 Game Over";

    document.querySelector("#menu p")
        .innerText =
        "Your Score: " + score;

    document.querySelector("#menu button")
        .innerText = "PLAY AGAIN";
}

/* Keyboard */

document.addEventListener("keydown", e => {

    if (e.key === "ArrowLeft") left();

    if (e.key === "ArrowRight") right();

    if (e.key === "ArrowUp" ||
        e.key === " ") jump();

});

/* Swipe */

let startX = 0;
let startY = 0;

document.addEventListener("touchstart", e => {

    startX = e.touches[0].clientX;
    startY = e.touches[0].clientY;

});

document.addEventListener("touchend", e => {

    const endX = e.changedTouches[0].clientX;
    const endY = e.changedTouches[0].clientY;

    const dx = endX - startX;
    const dy = endY - startY;

    if (Math.abs(dx) > Math.abs(dy)) {

        if (dx > 40) right();
        if (dx < -40) left();

    } else {

        if (dy < -40) jump();

    }

});

positionPlayer();

</script>

</body>
</html>
