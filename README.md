# Tic-tac-toe-game-Nice — let’s build an unbeatable Tic-Tac-Toe AI. I’ll give you a concise explanation, then two copy-pasteable implementations you can run right away:

1. a simple **Python (console)** version using **Minimax + alpha-beta pruning** (unbeatable), and
2. a neat **HTML + JavaScript** browser version (clickable board) also using minimax.

---

## Quick plan / idea

* Tic-Tac-Toe is a small finite game — minimax can search the full game tree.
* The AI evaluates terminal states: +1 for AI win, −1 for human win, 0 for draw.
* Use **alpha-beta** to speed up but for 3×3 it's optional.
* Optionally add difficulty levels by limiting minimax depth or adding randomness.

---

# 1) Python (console) — copy, paste, run

Save as `tictactoe_ai.py` and run `python tictactoe_ai.py`.

```python
# tictactoe_ai.py
import math
import random

EMPTY = " "
HUMAN = "X"
AI = "O"

def new_board():
    return [EMPTY]*9

def print_board(b):
    for r in range(3):
        print(" " + " | ".join(b[3*r:3*r+3]))
        if r < 2:
            print("---+---+---")

def available_moves(b):
    return [i for i, v in enumerate(b) if v == EMPTY]

def winner(b):
    wins = [(0,1,2),(3,4,5),(6,7,8),
            (0,3,6),(1,4,7),(2,5,8),
            (0,4,8),(2,4,6)]
    for a,b_,c in wins:
        if b[a] == b[b_] == b[c] != EMPTY:
            return b[a]
    if EMPTY not in b:
        return "Draw"
    return None

# Minimax with alpha-beta
def minimax(board, maximizing, alpha=-math.inf, beta=math.inf):
    w = winner(board)
    if w == AI:
        return  1, None
    if w == HUMAN:
        return -1, None
    if w == "Draw":
        return 0, None

    if maximizing:
        best_val = -math.inf
        best_move = None
        for m in available_moves(board):
            board[m] = AI
            val, _ = minimax(board, False, alpha, beta)
            board[m] = EMPTY
            if val > best_val:
                best_val = val
                best_move = m
            alpha = max(alpha, val)
            if beta <= alpha:
                break
        return best_val, best_move
    else:
        best_val = math.inf
        best_move = None
        for m in available_moves(board):
            board[m] = HUMAN
            val, _ = minimax(board, True, alpha, beta)
            board[m] = EMPTY
            if val < best_val:
                best_val = val
                best_move = m
            beta = min(beta, val)
            if beta <= alpha:
                break
        return best_val, best_move

def ai_move(board):
    _, move = minimax(board, True)
    return move

def human_turn(board):
    while True:
        try:
            move = int(input("Your move (0-8): ").strip())
            if move in available_moves(board):
                return move
            else:
                print("Invalid move. Choose an empty cell 0-8.")
        except ValueError:
            print("Enter a number between 0 and 8.")

def play():
    board = new_board()
    print("Cells numbered 0..8 left-to-right, top-to-bottom.")
    print_board(board)
    # Let human go first; swap if you prefer AI first
    while True:
        # Human
        mv = human_turn(board)
        board[mv] = HUMAN
        print_board(board)
        w = winner(board)
        if w:
            break

        # AI
        mv = ai_move(board)
        if mv is None:
            break
        board[mv] = AI
        print("AI moves to", mv)
        print_board(board)
        w = winner(board)
        if w:
            break

    if w == HUMAN:
        print("You win! (unexpected vs perfect AI)")
    elif w == AI:
        print("AI wins.")
    else:
        print("Draw.")

if __name__ == "__main__":
    play()
```

How it works: the AI chooses the minimax-optimal move and is therefore unbeatable. To make easier difficulties, pick a random move sometimes.

---

# 2) Browser version — HTML + JavaScript (clickable)

Create `tictactoe.html` and open in browser.

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Tic Tac Toe — Minimax AI</title>
  <style>
    body{font-family:system-ui,sans-serif;display:flex;flex-direction:column;align-items:center;padding:20px;}
    #board{display:grid;grid-template-columns:repeat(3,100px);grid-gap:5px;margin:15px;}
    .cell{width:100px;height:100px;display:flex;align-items:center;justify-content:center;font-size:40px;border:2px solid #333;cursor:pointer;user-select:none}
    .info{margin-top:10px}
    button{margin-left:8px}
  </style>
</head>
<body>
  <h2>Tic Tac Toe — Unbeatable AI</h2>
  <div id="board"></div>
  <div class="info">
    You are X. <span id="status"></span>
    <button id="reset">Reset</button>
  </div>

<script>
const HUMAN = 'X', AI = 'O', EMPTY = '';
let board = Array(9).fill(EMPTY);
const boardEl = document.getElementById('board');
const statusEl = document.getElementById('status');
document.getElementById('reset').onclick = reset;

function reset(){
  board.fill(EMPTY);
  render();
  statusEl.textContent = '';
}
function render(){
  boardEl.innerHTML = '';
  board.forEach((v,i)=>{
    const cell = document.createElement('div');
    cell.className = 'cell';
    cell.textContent = v;
    cell.onclick = ()=> onCellClick(i);
    boardEl.appendChild(cell);
  });
}

function winner(b){
  const wins = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];
  for (const [a,b1,c] of wins){
    if (b[a] && b[a] === b[b1] && b[a] === b[c]) return b[a];
  }
  if (b.every(x=>x!==EMPTY)) return 'Draw';
  return null;
}

function availableMoves(b){ return b.map((v,i)=> v===EMPTY ? i : null).filter(x=>x!==null); }

// Minimax w/ alpha-beta, returns {score, move}
function minimax(b, maximizing, alpha=-Infinity, beta=Infinity){
  const w = winner(b);
  if (w === AI) return {score: 1, move: null};
  if (w === HUMAN) return {score: -1, move: null};
  if (w === 'Draw') return {score: 0, move: null};

  if (maximizing){
    let best = {score: -Infinity, move: null};
    for (const m of availableMoves(b)){
      b[m] = AI;
      const res = minimax(b, false, alpha, beta);
      b[m] = EMPTY;
      if (res.score > best.score){ best = {score: res.score, move: m}; }
      alpha = Math.max(alpha, res.score);
      if (beta <= alpha) break;
    }
    return best;
  } else {
    let best = {score: Infinity, move: null};
    for (const m of availableMoves(b)){
      b[m] = HUMAN;
      const res = minimax(b, true, alpha, beta);
      b[m] = EMPTY;
      if (res.score < best.score){ best = {score: res.score, move: m}; }
      beta = Math.min(beta, res.score);
      if (beta <= alpha) break;
    }
    return best;
  }
}

function aiMove(){
  const {move} = minimax(board, true);
  return move;
}

function onCellClick(i){
  if (board[i] !== EMPTY) return;
  if (winner(board)) return;
  board[i] = HUMAN;
  render();
  const w = winner(board);
  if (w){
    showResult(w); return;
  }
  // AI's turn (instant)
  const mv = aiMove();
  if (mv !== null && mv !== undefined) board[mv] = AI;
  render();
  const w2 = winner(board);
  if (w2) showResult(w2);
}

function showResult(w){
  if (w === HUMAN) statusEl.textContent = 'You win (very unlikely vs perfect AI)!';
  else if (w === AI) statusEl.textContent = 'AI wins.';
  else statusEl.textContent = 'Draw.';
}

reset();
render();
</script>
</body>
</html>
```

Open that file and play. The AI will never lose.

---

## Making it less perfect (difficulty)

* **Easy:** 40% random move, else minimax.
* **Medium:** limit minimax depth to 2–4 and use heuristic (center > corners > edges).
* **Hard:** full minimax (perfect).

Example (JS): before calling minimax, do `if (Math.random() < 0.4) return random available move`.

---

## Tips & improvements

* Let AI choose to play first to see a forced draw/win pattern.
* Add UI highlights for winning line, score tracking.
* For large games (4×4, Connect Four) use heuristics + iterative deepening, or Monte-Carlo Tree Search (MCTS).
* Add a training mode: let two AIs play millions of games (for learning-based approach).

---

If you want, I can:

* give you a **difficulty variant** with randomness,
* add **win-line highlight + CSS polish** to the browser version, or
* port the AI to **React** or to a **mobile-friendly UI**.

Which would you like next? 🎮
