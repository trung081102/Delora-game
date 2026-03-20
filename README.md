<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Delora Caster - Memory Card</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            padding: 20px;
        }

        .header {
            display: flex;
            align-items: center;
            gap: 15px;
            margin-bottom: 20px;
            background: rgba(255,255,255,0.2);
            padding: 15px 30px;
            border-radius: 50px;
            backdrop-filter: blur(10px);
        }

        .logo {
            width: 50px;
            height: 50px;
        }

        .logo img {
            width: 100%;
            height: 100%;
            object-fit: contain;
        }

        .header h1 {
            color: white;
            font-size: 1.8rem;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .level-select {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .level-btn {
            padding: 10px 20px;
            border: none;
            border-radius: 20px;
            cursor: pointer;
            font-size: 0.9rem;
            font-weight: bold;
            transition: all 0.3s;
            background: rgba(255,255,255,0.3);
            color: white;
        }

        .level-btn:hover, .level-btn.active {
            background: white;
            color: #667eea;
            transform: translateY(-2px);
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }

        .stats {
            display: flex;
            gap: 30px;
            margin-bottom: 20px;
            color: white;
            font-size: 1.1rem;
        }

        .stats span {
            background: rgba(255,255,255,0.2);
            padding: 8px 20px;
            border-radius: 20px;
            backdrop-filter: blur(5px);
        }

        .game-board {
            display: grid;
            gap: 15px;
            max-width: 600px;
            width: 100%;
        }

        .card {
            aspect-ratio: 1;
            perspective: 1000px;
            cursor: pointer;
            position: relative;
        }

        .card-inner {
            position: relative;
            width: 100%;
            height: 100%;
            transition: transform 0.6s;
            transform-style: preserve-3d;
        }

        .card.flipped .card-inner {
            transform: rotateY(180deg);
        }

        .card.matched .card-inner {
            transform: rotateY(180deg);
        }

        .card.matched {
            cursor: default;
        }

        .card-front, .card-back {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            border-radius: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }

        .card-front {
            background: linear-gradient(145deg, #ffffff, #f0f0f0);
            transform: rotateY(180deg);
            padding: 10px;
        }

        .card-front img {
            width: 100%;
            height: 100%;
            object-fit: contain;
        }

        .card-back {
            background: linear-gradient(145deg, #ff6b6b, #ee5a5a);
            font-size: 3rem;
        }

        .card-back::after {
            content: "?";
            color: white;
            font-weight: bold;
        }

        .btn-restart {
            margin-top: 25px;
            padding: 12px 30px;
            font-size: 1.1rem;
            background: white;
            color: #667eea;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-weight: bold;
        }

        .win-message {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 40px;
            border-radius: 20px;
            text-align: center;
            z-index: 100;
        }

        .win-message.show {
            display: block;
        }

        .overlay {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 99;
        }

        .overlay.show {
            display: block;
        }
    </style>
</head>
<body>

    <div class="header">
        <div class="logo">
            <img src="/img/mediakit/emblem-white.png" alt="Delora Logo">
        </div>
        <h1>Delora Caster</h1>
    </div>

    <div class="level-select">
        <button class="level-btn active" data-pairs="3">Level 1</button>
        <button class="level-btn" data-pairs="4">Level 2</button>
        <button class="level-btn" data-pairs="6">Level 3</button>
        <button class="level-btn" data-pairs="8">Level 4</button>
        <button class="level-btn" data-pairs="12">Level 5</button>
    </div>

    <div class="stats">
        <span>Time: <b id="timer">0</b>s</span>
        <span>Moves: <b id="moves">0</b></span>
    </div>

    <div class="game-board" id="gameBoard"></div>

    <button class="btn-restart" onclick="restartGame()">Play Again</button>

    <div class="overlay" id="overlay"></div>
    <div class="win-message" id="winMessage">
        <h2>Completed</h2>
        <p>Time: <b id="finalTime">0</b>s</p>
        <p>Moves: <b id="finalMoves">0</b></p>
        <button class="btn-restart" onclick="restartGame()">Play Again</button>
    </div>

<script>
const cardImages = [
    "/img/mediakit/emblem-white.png",
    "/img/mediakit/emblem-black.png"
];

let cards = [];
let flippedCards = [];
let matchedPairs = 0;
let moves = 0;
let timer = 0;
let timerInterval = null;
let gameStarted = false;
let canFlip = true;
let currentPairs = 3;

document.querySelectorAll('.level-btn').forEach(btn => {
    btn.addEventListener('click', () => {
        document.querySelectorAll('.level-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        currentPairs = parseInt(btn.dataset.pairs);
        restartGame();
    });
});

function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
}

function createCard(image, index) {
    const card = document.createElement('div');
    card.className = 'card';
    card.dataset.image = image;

    card.innerHTML = `
        <div class="card-inner">
            <div class="card-front">
                <img src="${image}">
            </div>
            <div class="card-back"></div>
        </div>
    `;

    card.addEventListener('click', () => flipCard(card));
    return card;
}

function flipCard(card) {
    if (!canFlip || card.classList.contains('flipped') || card.classList.contains('matched')) return;

    if (!gameStarted) {
        gameStarted = true;
        startTimer();
    }

    card.classList.add('flipped');
    flippedCards.push(card);

    if (flippedCards.length === 2) {
        moves++;
        document.getElementById('moves').textContent = moves;
        checkMatch();
    }
}

function checkMatch() {
    canFlip = false;
    const [c1, c2] = flippedCards;

    if (c1.dataset.image === c2.dataset.image) {
        c1.classList.add('matched');
        c2.classList.add('matched');
        matchedPairs++;
        flippedCards = [];
        canFlip = true;

        if (matchedPairs === currentPairs) endGame();
    } else {
        setTimeout(() => {
            c1.classList.remove('flipped');
            c2.classList.remove('flipped');
            flippedCards = [];
            canFlip = true;
        }, 800);
    }
}

function startTimer() {
    timerInterval = setInterval(() => {
        timer++;
        document.getElementById('timer').textContent = timer;
    }, 1000);
}

function endGame() {
    clearInterval(timerInterval);
    document.getElementById('finalTime').textContent = timer;
    document.getElementById('finalMoves').textContent = moves;
    document.getElementById('overlay').classList.add('show');
    document.getElementById('winMessage').classList.add('show');
}

function restartGame() {
    clearInterval(timerInterval);

    cards = [];
    flippedCards = [];
    matchedPairs = 0;
    moves = 0;
    timer = 0;
    gameStarted = false;
    canFlip = true;

    document.getElementById('timer').textContent = '0';
    document.getElementById('moves').textContent = '0';
    document.getElementById('overlay').classList.remove('show');
    document.getElementById('winMessage').classList.remove('show');

    const board = document.getElementById('gameBoard');
    board.innerHTML = '';

    let selectedImages = [];
    for (let i = 0; i < currentPairs; i++) {
        selectedImages.push(cardImages[i % cardImages.length]);
    }

    const pairs = shuffle([...selectedImages, ...selectedImages]);

    pairs.forEach((img, i) => {
        const card = createCard(img, i);
        board.appendChild(card);
    });
}

restartGame();
</script>

</body>
</html>
