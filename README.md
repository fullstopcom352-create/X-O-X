# X-O-X
<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>X O X — Tic Tac Toe</title>
  <style>
    :root{
      --bg:#0f1724;
      --card:#0b1220;
      --accent:#06b6d4;
      --muted:#94a3b8;
      --win: #10b981;
      --lose: #ef4444;
      color-scheme: dark;
    }
    *{box-sizing:border-box;font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,"Helvetica Neue",Arial;}
    body{margin:0;min-height:100vh;display:flex;align-items:center;justify-content:center;background:linear-gradient(180deg,#071021 0%, #0f1724 100%);color:#e6eef8;padding:28px}
    .app{width:100%;max-width:760px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border-radius:12px;padding:20px;box-shadow:0 10px 30px rgba(2,6,23,0.6);}
    header{display:flex;gap:12px;align-items:center;justify-content:space-between;margin-bottom:12px}
    h1{font-size:20px;margin:0}
    .controls{display:flex;gap:8px;align-items:center}
    .btn{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 10px;border-radius:8px;color:var(--muted);cursor:pointer}
    .btn.active{border-color:var(--accent);color:var(--accent)}
    .board-wrap{display:grid;grid-template-columns:1fr 260px;gap:18px}
    .board{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;background:transparent;padding:16px;border-radius:10px}
    .cell{aspect-ratio:1/1;border-radius:8px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));display:flex;align-items:center;justify-content:center;font-size:44px;cursor:pointer;user-select:none;border:1px solid rgba(255,255,255,0.03)}
    .cell.win{background:linear-gradient(180deg, rgba(16,185,129,0.12), rgba(16,185,129,0.06));box-shadow:0 6px 16px rgba(16,185,129,0.06)}
    .cell.tie{opacity:0.65}
    .panel{background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));padding:14px;border-radius:10px;border:1px solid rgba(255,255,255,0.03)}
    .status{font-size:14px;color:var(--muted);margin-top:6px}
    .score{display:flex;gap:12px;margin-top:10px}
    .score .item{padding:8px;border-radius:8px;background:rgba(255,255,255,0.02);font-weight:600}
    .small{font-size:12px;color:var(--muted)}
    .footer{margin-top:12px;display:flex;justify-content:space-between;align-items:center;gap:8px}
    .switch{display:flex;gap:6px}
    .select{padding:6px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--muted)}
    @media (max-width:720px){
      .board-wrap{grid-template-columns:1fr;align-items:start}
      .panel{order:2}
    }
  </style>
</head>
<body>
  <div class="app" role="application" aria-label="Tic Tac Toe">
    <header>
      <div>
        <h1>X O X — Tic Tac Toe</h1>
        <div class="small">Simple &amp; fast — Human vs Human or Human vs Computer (unbeatable).</div>
      </div>
      <div class="controls" role="toolbar" aria-label="Game controls">
        <button class="btn" id="newGameBtn">Reset Board</button>
        <button class="btn" id="fullResetBtn">Full Reset</button>
      </div>
    </header>

    <div class="board-wrap">
      <div>
        <div id="board" class="board" aria-live="polite">
          <!-- cells injected by JS -->
        </div>
      </div>

      <div class="panel" aria-label="Game panel">
        <div>
          <div style="display:flex;justify-content:space-between;align-items:center;">
            <div>
              <div class="small">Mode</div>
              <div style="margin-top:6px;">
                <button class="btn active" id="modeHumanBtn">2 Players</button>
                <button class="btn" id="modeAIBtn">Vs Computer</button>
              </div>
            </div>
            <div>
              <div class="small">You are</div>
              <select id="yourMark" class="select" title="Choose your mark">
                <option value="X">X (goes first)</option>
                <option value="O">O</option>
              </select>
            </div>
          </div>

          <div class="status" id="status">Turn: <strong id="turnText">X</strong></div>

          <div class="score" style="margin-top:12px">
            <div class="item">X: <span id="scoreX">0</span></div>
            <div class="item">O: <span id="scoreO">0</span></div>
            <div class="item">Ties: <span id="scoreTie">0</span></div>
          </div>

        </div>

        <div class="footer">
          <div class="small">Click a cell to play</div>
          <div>
            <button class="btn" id="undoBtn">Undo</button>
          </div>
        </div>
      </div>
    </div>
  </div>

<script>
(() => {
  // Game state
  const boardEl = document.getElementById('board');
  const statusEl = document.getElementById('status');
  const turnText = document.getElementById('turnText');
  const modeHumanBtn = document.getElementById('modeHumanBtn');
  const modeAIBtn = document.getElementById('modeAIBtn');
  const yourMarkSelect = document.getElementById('yourMark');
  const newGameBtn = document.getElementById('newGameBtn');
  const fullResetBtn = document.getElementById('fullResetBtn');
  const undoBtn = document.getElementById('undoBtn');
  const scoreXEl = document.getElementById('scoreX');
  const scoreOEl = document.getElementById('scoreO');
  const scoreTieEl = document.getElementById('scoreTie');

  let board = Array(9).fill(null);
  let currentPlayer = 'X';
  let mode = '2p'; // '2p' or 'ai'
  let yourMark = 'X';
  let scores = { X:0, O:0, T:0 };
  let history = []; // for undo
  let gameOver = false;
  let aiThinking = false;

  const winningLines = [
    [0,1,2],[3,4,5],[6,7,8],
    [0,3,6],[1,4,7],[2,5,8],
    [0,4,8],[2,4,6]
  ];

  function createBoardUI(){
    boardEl.innerHTML = '';
    for(let i=0;i<9;i++){
      const c = document.createElement('div');
      c.className = 'cell';
      c.dataset.index = i;
      c.setAttribute('role','button');
      c.setAttribute('aria-label','Cell '+(i+1));
      c.addEventListener('click', onCellClick);
      boardEl.appendChild(c);
    }
    render();
  }

  function render(){
    const cells = boardEl.children;
    for(let i=0;i<9;i++){
      cells[i].textContent = board[i] || '';
      cells[i].classList.remove('win','tie');
    }
    turnText.textContent = currentPlayer;
    statusEl.querySelector('.small');
  }

  function onCellClick(e){
    if(gameOver || aiThinking) return;
    const idx = Number(e.currentTarget.dataset.index);
    if(board[idx]) return;
    makeMove(idx, currentPlayer);
    history.push({ board: board.slice(), player: currentPlayer });
    const result = checkGameEnd();
    if(result) return handleResult(result);

    // switch
    currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
    render();

    // If vs AI and it's AI's turn -> play
    if(mode === 'ai' && currentPlayer !== yourMark){
      aiMove();
    }
  }

  function makeMove(idx, player){
    board[idx] = player;
    render();
  }

  function checkGameEnd(){
    for(const line of winningLines){
      const [a,b,c] = line;
      if(board[a] && board[a] === board[b] && board[a] === board[c]){
        return { winner: board[a], line };
      }
    }
    if(board.every(Boolean)) return { tie: true };
    return null;
  }

  function handleResult(res){
    gameOver = true;
    if(res.tie){
      scores.T++;
      scoreTieEl.textContent = scores.T;
      highlightTie();
      statusEl.textContent = 'Result: Tie!';
    } else {
      const w = res.winner;
      scores[w]++;
      if(w === 'X') scoreXEl.textContent = scores.X;
      else scoreOEl.textContent = scores.O;
      highlightWin(res.line);
      statusEl.textContent = `Winner: ${w}!`;
    }
  }

  function highlightWin(line){
    line.forEach(i => boardEl.children[i].classList.add('win'));
  }
  function highlightTie(){
    for(let i=0;i<9;i++){
      boardEl.children[i].classList.add('tie');
    }
  }

  // Minimax AI (unbeatable)
  function aiMove(){
    aiThinking = true;
    setTimeout(() => {
      const ai = currentPlayer;
      const human = yourMark;
      // best move by minimax
      const move = findBestMove(board.slice(), ai, human);
      if(move !== null && !gameOver) {
        makeMove(move, ai);
        history.push({ board: board.slice(), player: ai });
        const result = checkGameEnd();
        if(result) {
          handleResult(result);
        } else {
          currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
          render();
        }
      }
      aiThinking = false;
    }, 220); // small delay for UX
  }

  function findBestMove(b, ai, human){
    // If board empty, pick center for speed
    if(b.every(x => !x)) return 4;
    let bestScore = -Infinity;
    let bestMove = null;
    for(let i=0;i<9;i++){
      if(!b[i]){
        b[i] = ai;
        const score = minimax(b, 0, false, ai, human);
        b[i] = null;
        if(score > bestScore){
          bestScore = score;
          bestMove = i;
        }
      }
    }
    return bestMove;
  }

  function minimax(b, depth, isMaximizing, ai, human){
    const res = evaluateBoard(b, ai, human);
    if(res !== null) return res;

    if(isMaximizing){
      let best = -Infinity;
      for(let i=0;i<9;i++){
        if(!b[i]){
          b[i] = ai;
          const val = minimax(b, depth+1, false, ai, human);
          b[i] = null;
          best = Math.max(best, val);
        }
      }
      return best;
    } else {
      let best = Infinity;
      for(let i=0;i<9;i++){
        if(!b[i]){
          b[i] = human;
          const val = minimax(b, depth+1, true, ai, human);
          b[i] = null;
          best = Math.min(best, val);
        }
      }
      return best;
    }
  }

  function evaluateBoard(b, ai, human){
    for(const line of winningLines){
      const [a,b1,c] = line;
      if(b[a] && b[a] === b[b1] && b[a] === b[c]){
        return (b[a] === ai) ? 10 : -10;
      }
    }
    if(b.every(Boolean)) return 0;
    return null;
  }

  // Controls
  modeHumanBtn.addEventListener('click', () => {
    mode = '2p';
    modeHumanBtn.classList.add('active');
    modeAIBtn.classList.remove('active');
    resetBoard(false);
  });

  modeAIBtn.addEventListener('click', () => {
    mode = 'ai';
    modeAIBtn.classList.add('active');
    modeHumanBtn.classList.remove('active');
    resetBoard(false);
  });

  yourMarkSelect.addEventListener('change', (e) => {
    yourMark = e.target.value;
    // if AI mode and AI is X, AI should start
    resetBoard(false);
    if(mode==='ai' && currentPlayer !== yourMark){
      aiMove();
    }
  });

  newGameBtn.addEventListener('click', () => resetBoard(false));
  fullResetBtn.addEventListener('click', () => {
    scores = { X:0, O:0, T:0 };
    scoreXEl.textContent = '0';
    scoreOEl.textContent = '0';
    scoreTieEl.textContent = '0';
    resetBoard(true);
  });

  undoBtn.addEventListener('click', () => {
    if(history.length < 2 || aiThinking || gameOver) {
      // If vs AI, undo last two moves to return to player's turn
      if(history.length >= 1 && mode === '2p') {
        history.pop();
        const prev = history.pop();
        if(prev) board = prev.board.slice();
        else board = Array(9).fill(null);
        currentPlayer = prev ? (prev.player === 'X' ? 'O' : 'X') : 'X';
        gameOver = false;
        render();
      }
      return;
    }
    // For vs AI: remove last two entries (AI move + human move)
    if(mode === 'ai'){
      history.pop(); // AI
      const last = history.pop(); // human
      board = last ? last.board.slice() : Array(9).fill(null);
      currentPlayer = last ? (last.player === 'X' ? 'O' : 'X') : 'X';
      gameOver = false;
      render();
    } else {
      // 2p: undo last move
      history.pop();
      const last = history.pop();
      board = last ? last.board.slice() : Array(9).fill(null);
      currentPlayer = last ? (last.player === 'X' ? 'O' : 'X') : 'X';
      gameOver = false;
      render();
    }
  });

  function resetBoard(fullReset){
    board = Array(9).fill(null);
    history = [];
    gameOver = false;
    // Set starter: X always starts
    currentPlayer = 'X';
    render();
    statusEl.textContent = `Turn: ${currentPlayer}`;
    if(mode === 'ai' && yourMark !== 'X'){
      // If player chose O, AI (X) should go first
      aiMove();
    }
    if(fullReset){
      scores = { X:0, O:0, T:0 };
      scoreXEl.textContent = '0';
      scoreOEl.textContent = '0';
      scoreTieEl.textContent = '0';
    }
  }

  // init
  createBoardUI();
  resetBoard(false);

})();
</script>
</body>
</html>
