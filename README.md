# webapp01
テトリスを作ろう
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tetris</title>
  <style>
    body {
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
      background-color: #f0f0f0;
    }

    #game-board {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(20, 30px);
      gap: 1px;
      border: 1px solid #ccc;
      background-color: #fff;
    }

    .cell {
      width: 30px;
      height: 30px;
      border: 1px solid #ccc;
      box-sizing: border-box;
    }
  </style>
</head>
<body>
  <div id="game-board"></div>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const gameBoard = document.getElementById('game-board');
      const boardWidth = 10;
      const boardHeight = 20;
      const blockSize = 30;
      const board = Array.from({ length: boardHeight }, () => Array(boardWidth).fill(0));

      let currentPiece;
      let intervalId;

      function drawBoard() {
        gameBoard.innerHTML = '';

        for (let row = 0; row < boardHeight; row++) {
          for (let col = 0; col < boardWidth; col++) {
            const cell = document.createElement('div');
            cell.className = `cell ${board[row][col] ? 'occupied' : ''}`;
            cell.style.width = `${blockSize}px`;
            cell.style.height = `${blockSize}px`;
            gameBoard.appendChild(cell);
          }
        }
      }

      function drawPiece() {
        currentPiece.shape.forEach((row, rowIndex) => {
          row.forEach((value, colIndex) => {
            if (value) {
              const cell = document.createElement('div');
              cell.className = 'cell active';
              cell.style.width = `${blockSize}px`;
              cell.style.height = `${blockSize}px`;
              cell.style.top = `${(currentPiece.y + rowIndex) * blockSize}px`;
              cell.style.left = `${(currentPiece.x + colIndex) * blockSize}px`;
              gameBoard.appendChild(cell);
            }
          });
        });
      }

      function movePiece() {
        currentPiece.y++;

        if (collision()) {
          currentPiece.y--;
          mergePiece();
          spawnPiece();
        }
      }

      function collision() {
        return currentPiece.shape.some((row, rowIndex) => {
          return row.some((value, colIndex) => {
            const newY = currentPiece.y + rowIndex;
            const newX = currentPiece.x + colIndex;

            return (
              value &&
              (newY >= boardHeight ||
              newX < 0 ||
              newX >= boardWidth ||
              (newY >= 0 && board[newY][newX]))
            );
          });
        });
      }

      function mergePiece() {
        currentPiece.shape.forEach((row, rowIndex) => {
          row.forEach((value, colIndex) => {
            if (value) {
              board[currentPiece.y + rowIndex][currentPiece.x + colIndex] = 1;
            }
          });
        });

        checkRows();
        drawBoard();
      }

      function checkRows() {
        for (let row = boardHeight - 1; row >= 0; row--) {
          if (board[row].every((value) => value)) {
            board.splice(row, 1);
            board.unshift(Array(boardWidth).fill(0));
          }
        }
      }

      function spawnPiece() {
        const pieces = [
          { shape: [[1, 1, 1, 1]], color: 'cyan' },
          { shape: [[1, 1, 1], [1, 0, 0]], color: 'blue' },
          { shape: [[1, 1, 1], [0, 0, 1]], color: 'orange' },
          { shape: [[1, 1, 0], [0, 1, 1]], color: 'yellow' },
          { shape: [[0, 1, 1], [1, 1, 0]], color: 'green' },
          { shape: [[1, 1], [1, 1]], color: 'red' },
          { shape: [[1, 1, 1], [0, 1, 0]], color: 'purple' },
        ];

        const randomPiece = pieces[Math.floor(Math.random() * pieces.length)];

        currentPiece = {
          shape: randomPiece.shape,
          color: randomPiece.color,
          x: Math.floor((boardWidth - randomPiece.shape[0].length) / 2),
          y: 0,
        };

        if (collision()) {
          // Game over
          clearInterval(intervalId);
          alert('Game Over!');
        }
      }

      function handleKeyPress(event) {
        switch (event.code) {
          case 'ArrowLeft':
            currentPiece.x--;
            if (collision()) currentPiece.x++;
            break;
          case 'ArrowRight':
            currentPiece.x++;
            if (collision()) currentPiece.x--;
            break;
          case 'ArrowDown':
            currentPiece.y++;
            if (collision()) currentPiece.y--;
            break;
          case 'ArrowUp':
            rotatePiece();
            break;
        }

        drawBoard();
        drawPiece();
      }

      function rotatePiece() {
        const tempPiece = currentPiece.shape;
        currentPiece.shape = currentPiece.shape[0].map((_, i) => currentPiece.shape.map(row => row[i])).reverse();

        if (collision()) {
          currentPiece.shape = tempPiece;
        }
      }

      function startGame() {
        intervalId = setInterval(() => {
          movePiece();
          drawBoard();
          drawPiece();
        }, 500);

        document.addEventListener('keydown', handleKeyPress);
      }

      // Start the game
      startGame();
    });
  </script>
</body>
</html>

