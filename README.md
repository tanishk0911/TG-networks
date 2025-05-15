# TG-networks


Welcome


<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<meta name="google-site-verification" content="d4hASePdM2T16JpypMUESKFixwO1DjqJXn9AZGqTzqU" />
<title>Tic Tac Toe - Complete Game</title>
<style>
  :root {
    --bg: #f7f7f7;
    --text: #000;
    --cell-bg: #fff;
    --cell-border: #444;
    --cell-hover: #eee;
  }
  body {
    font-family: 'Courier New', Courier, monospace;
    background-color: var(--bg);
    color: var(--text);
    margin: 0; padding: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
  }
  #setup {
    position: fixed;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    background: white;
    border: 2px solid black;
    padding: 30px 40px;
    box-shadow: 4px 4px 0 #000;
    text-align: center;
    width: 320px;
    z-index: 10;
  }
  #setup input,
  #setup select {
    width: 90%;
    padding: 8px;
    margin: 10px 0;
    font-size: 16px;
    border: 1px solid black;
    outline: none;
    font-family: inherit;
  }
  #setup button {
    margin-top: 20px;
    padding: 10px 25px;
    font-size: 16px;
    border: 2px solid black;
    background: white;
    cursor: pointer;
    font-family: inherit;
  }
  #setup button:hover {
    background: black;
    color: white;
  }
  #game {
    margin-top: 100px;
    display: none;
    text-align: center;
  }
  #board {
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
    gap: 5px;
    margin: 20px auto;
  }
  .cell {
    background-color: var(--cell-bg);
    border: 2px solid var(--cell-border);
    font-size: 3em;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    user-select: none;
    transition: background-color 0.2s;
  }
  .cell:hover {
    background-color: var(--cell-hover);
  }
  #status {
    font-weight: bold;
    font-size: 1.2em;
  }
  #scoreboard {
    margin-top: 20px;
  }
  #scoreboard span {
    margin: 0 15px;
    font-weight: bold;
  }
  <meta name="google-site-verification" content="d4hASePdM2T16JpypMUESKFixwO1DjqJXn9AZGqTzqU" />
</style>
</head>
<body>

<div id="setup">
  <h2>Tic Tac Toe</h2>
  <input type="text" id="playerName" placeholder="Your Name (X)" />
  
  <select id="difficulty">
    <option value="easy">Easy</option>
    <option value="hard">Hard</option>
    <option value="impossible" selected>Impossible</option>
  </select>
  
  <select id="theme" onchange="applyTheme(this.value)">
    <option value="default">Default Theme</option>
    <option value="dark">Dark</option>
    <option value="neon">Neon</option>
    <option value="sky">Sky</option>
  </select>
  
  <button onclick="startGame(false)">Start Game</button>
  <button onclick="startGame(true)">Start in Test Mode</button>
</div>

<div id="game">
  <div id="scoreboard">
    <span id="player1Name">Player X</span>: <span id="player1Score">0</span> |
    Computer: <span id="player2Score">0</span> |
    Draws: <span id="drawScore">0</span>
  </div>
  <div id="board"></div>
  <div id="status"></div>
  <button onclick="resetGame()">Reset Game</button>
</div>

<audio id="clickSound" src="click.mp3"></audio>
<audio id="winSound" src="win.mp3"></audio>
<audio id="drawSound" src="draw.mp3"></audio>

<script>
  const boardEl = document.getElementById('board');
  const statusEl = document.getElementById('status');
  const player1NameEl = document.getElementById('player1Name');
  const player1ScoreEl = document.getElementById('player1Score');
  const player2ScoreEl = document.getElementById('player2Score');
  const drawScoreEl = document.getElementById('drawScore');

  let board = ['', '', '', '', '', '', '', '', ''];
  let currentPlayer = 'X'; // Player always X, Computer O
  let gameActive = true;
  let player1Score = 0;
  let player2Score = 0;
  let drawScore = 0;
  let player1Name = 'Player X';
  let difficulty = 'impossible';
  let isTestMode = false;

  // Sound effects
  const clickSound = document.getElementById('clickSound');
  const winSound = document.getElementById('winSound');
  const drawSound = document.getElementById('drawSound');

  function playSound(sound) {
    if (!sound) return;
    sound.currentTime = 0;
    sound.play();
  }

  // Winning combos
  const winCombos = [
    [0,1,2],[3,4,5],[6,7,8], // rows
    [0,3,6],[1,4,7],[2,5,8], // cols
    [0,4,8],[2,4,6]          // diagonals
  ];

  function renderBoard() {
    boardEl.innerHTML = '';
    board.forEach((cell, idx) => {
      const cellEl = document.createElement('div');
      cellEl.classList.add('cell');
      cellEl.textContent = cell;
      cellEl.addEventListener('click', () => cellClicked(idx));
      boardEl.appendChild(cellEl);
    });
  }

  function cellClicked(idx) {
    if (!gameActive) return;
    if (board[idx] !== '') return;

    if (currentPlayer === 'X') {
      board[idx] = 'X';
      playSound(clickSound);
      renderBoard();
      if (checkWin('X')) {
        endGame('X');
        return;
      }
      if (checkDraw()) {
        endGame('draw');
        return;
      }
      currentPlayer = 'O';
      if (!isTestMode) {
        setTimeout(aiMove, 300);
      }
    }
  }

  function aiMove() {
    if (!gameActive) return;

    let move;
    if (difficulty === 'easy') {
      move = getRandomMove();
    } else if (difficulty === 'hard') {
      move = getHardMove();
    } else {
      move = minimax(board, 'O').index;
    }
    if (move !== undefined) {
      board[move] = 'O';
      playSound(clickSound);
      renderBoard();
      if (checkWin('O')) {
        endGame('O');
        return;
      }
      if (checkDraw()) {
        endGame('draw');
        return;
      }
      currentPlayer = 'X';
    }
  }

  function getRandomMove() {
    const available = board.map((v,i) => v === '' ? i : null).filter(i => i !== null);
    return available[Math.floor(Math.random() * available.length)];
  }

  // Simple 'hard' AI: if can win or block, do it, else random
  function getHardMove() {
    // Win if possible
    for (let i = 0; i < board.length; i++) {
      if (board[i] === '') {
        board[i] = 'O';
        if (checkWin('O')) {
          board[i] = '';
          return i;
        }
        board[i] = '';
      }
    }
    // Block opponent win
    for (let i = 0; i < board.length; i++) {
      if (board[i] === '') {
        board[i] = 'X';
        if (checkWin('X')) {
          board[i] = '';
          return i;
        }
        board[i] = '';
      }
    }
    return getRandomMove();
  }

  // Minimax algorithm for impossible mode
  function minimax(newBoard, player) {
    const availSpots = newBoard.map((v,i) => v === '' ? i : null).filter(i => i !== null);

    if (checkWinStatic(newBoard, 'X')) {
      return {score: -10};
    } else if (checkWinStatic(newBoard, 'O')) {
      return {score: 10};
    } else if (availSpots.length === 0) {
      return {score: 0};
    }

    let moves = [];
    for (let i = 0; i < availSpots.length; i++) {
      let move = {};
      move.index = availSpots[i];
      newBoard[availSpots[i]] = player;

      if (player === 'O') {
        let result = minimax(newBoard, 'X');
        move.score = result.score;
      } else {
        let result = minimax(newBoard, 'O');
        move.score = result.score;
      }

      newBoard[availSpots[i]] = '';
      moves.push(move);
    }

    let bestMove;
    if (player === 'O') {
      let bestScore = -Infinity;
      for (let i = 0; i < moves.length; i++) {
        if (moves[i].score > bestScore) {
          bestScore = moves[i].score;
          bestMove = moves[i];
        }
      }
    } else {
      let bestScore = Infinity;
      for (let i = 0; i < moves.length; i++) {
        if (moves[i].score < bestScore) {
          bestScore = moves[i].score;
          bestMove = moves[i];
        }
      }
    }
    return bestMove;
  }

  function checkWin(player) {
    return winCombos.some(combo => 
      combo.every(i => board[i] === player)
    );
  }

  // Static check for minimax recursive calls
  function checkWinStatic(bd, player) {
    return winCombos.some(combo => 
      combo.every(i => bd[i] === player)
    );
  }

  function checkDraw() {
    return board.every(cell => cell !== '');
  }

  function endGame(winner) {
    gameActive = false;
    if (winner === 'X') {
      statusEl.textContent = `${player1Name} (X) wins!`;
      player1Score++;
      playSound(winSound);
    } else if (winner === 'O') {
      statusEl.textContent = `Computer (O) wins!`;
      player2Score++;
      playSound(winSound);
    } else {
      statusEl.textContent = "It's a draw!";
      drawScore++;
      playSound(drawSound);
    }
    updateScore();
  }

  function updateScore() {
    player1ScoreEl.textContent = player1Score;
    player2ScoreEl.textContent = player2Score;
    drawScoreEl.textContent = drawScore;
  }

  function resetGame() {
    board = ['', '', '', '', '', '', '', '', ''];
    currentPlayer = 'X';
    gameActive = true;
    statusEl.textContent = `${player1Name}'s turn (X)`;
    renderBoard();
  }

  // Apply themes
  function applyTheme(theme) {
    if (theme === 'default') {
      document.documentElement.style.setProperty('--bg', '#f7f7f7');
      document.documentElement.style.setProperty('--text', '#000');
      document.documentElement.style.setProperty('--cell-bg', '#fff');
      document.documentElement.style.setProperty('--cell-border', '#444');
      document.documentElement.style.setProperty('--cell-hover', '#eee');
    } else if (theme === 'dark') {
      document.documentElement.style.setProperty('--bg', '#121212');
      document.documentElement.style.setProperty('--text', '#eee');
      document.documentElement.style.setProperty('--cell-bg', '#333');
      document.documentElement.style.setProperty('--cell-border', '#555');
      document.documentElement.style.setProperty('--cell-hover', '#444');
    } else if (theme === 'neon') {
      document.documentElement.style.setProperty('--bg', '#0f0f0f');
      document.documentElement.style.setProperty('--text', '#0ff');
      document.documentElement.style.setProperty('--cell-bg', '#222');
      document.documentElement.style.setProperty('--cell-border', '#0ff');
      document.documentElement.style.setProperty('--cell-hover', '#044');
    } else if (theme === 'sky') {
      document.documentElement.style.setProperty('--bg', '#d0e7f9');
      document.documentElement.style.setProperty('--text', '#036');
      document.documentElement.style.setProperty('--cell-bg', '#a9d0f5');
      document.documentElement.style.setProperty('--cell-border', '#036');
      document.documentElement.style.setProperty('--cell-hover', '#79b1f7');
    }
  }

  function startGame(testMode) {
    player1Name = document.getElementById('playerName').value.trim() || 'Player X';
    difficulty = document.getElementById('difficulty').value;
    isTestMode = testMode;
    const theme = document.getElementById('theme').value;

    player1NameEl.textContent = player1Name;
    applyTheme(theme);

    document.getElementById('setup').style.display = 'none';
    document.getElementById('game').style.display = 'block';

    resetGame();
    statusEl.textContent = `${player1Name}'s turn (X)`;

    if (!isTestMode && currentPlayer === 'O') {
      aiMove();
    }
  }

</script>

</body>
</html>
