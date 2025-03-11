<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minesweeper</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background: #f0f0f0;
            margin: 0;
            min-height: 100vh;
        }

        .game-header {
            display: flex;
            justify-content: space-between;
            width: 400px;
            margin: 20px 0;
            padding: 15px;
            background: #fff;
            border-radius: 10px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        .stats {
            font-size: 1.2em;
            color: #333;
        }

        .grid {
            display: grid;
            grid-template-columns: repeat(10, 40px);
            gap: 2px;
            background: #ddd;
            padding: 5px;
            border-radius: 5px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        .cell {
            width: 40px;
            height: 40px;
            background: #bbb;
            border: none;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
        }

        .cell:hover {
            background: #999;
        }

        .revealed {
            background: #fff;
        }

        .mine {
            background: #ff4444 !important;
        }

        .flag {
            background: #44ff44;
        }

        .number-1 { color: #0000ff; }
        .number-2 { color: #008000; }
        .number-3 { color: #ff0000; }
        .number-4 { color: #000080; }
        .number-5 { color: #800000; }
        .number-6 { color: #008080; }
        .number-7 { color: #000000; }
        .number-8 { color: #808080; }
    </style>
</head>
<body>
    <div class="game-header">
        <div class="stats">🚩 <span id="mines-left">10</span></div>
        <div class="stats">⏱ <span id="timer">0</span></div>
    </div>
    <div id="grid" class="grid"></div>

    <script>
        class Minesweeper {
            constructor(rows = 10, cols = 10, mines = 10) {
                this.rows = rows;
                this.cols = cols;
                this.mines = mines;
                this.flagsLeft = mines;
                this.gameOver = false;
                this.startTime = null;
                this.timerInterval = null;
                this.grid = [];
                this.initGrid();
                this.createGrid();
            }

            initGrid() {
                // Initialize empty grid
                for (let i = 0; i < this.rows; i++) {
                    this.grid[i] = new Array(this.cols).fill(0);
                }
                
                // Place mines
                let minesPlaced = 0;
                while (minesPlaced < this.mines) {
                    const row = Math.floor(Math.random() * this.rows);
                    const col = Math.floor(Math.random() * this.cols);
                    if (this.grid[row][col] !== 'X') {
                        this.grid[row][col] = 'X';
                        minesPlaced++;
                    }
                }

                // Calculate numbers
                for (let row = 0; row < this.rows; row++) {
                    for (let col = 0; col < this.cols; col++) {
                        if (this.grid[row][col] === 'X') continue;
                        let count = 0;
                        for (let i = -1; i <= 1; i++) {
                            for (let j = -1; j <= 1; j++) {
                                if (row+i >= 0 && row+i < this.rows && 
                                    col+j >= 0 && col+j < this.cols &&
                                    this.grid[row+i][col+j] === 'X') {
                                    count++;
                                }
                            }
                        }
                        this.grid[row][col] = count;
                    }
                }
            }

            createGrid() {
                const gridElement = document.getElementById('grid');
                gridElement.innerHTML = '';
                
                for (let row = 0; row < this.rows; row++) {
                    for (let col = 0; col < this.cols; col++) {
                        const cell = document.createElement('div');
                        cell.className = 'cell';
                        cell.dataset.row = row;
                        cell.dataset.col = col;
                        
                        cell.addEventListener('click', (e) => this.handleLeftClick(e));
                        cell.addEventListener('contextmenu', (e) => this.handleRightClick(e));
                        
                        gridElement.appendChild(cell);
                    }
                }
            }

            handleLeftClick(event) {
                if (this.gameOver) return;
                if (!this.startTime) this.startTimer();
                
                const cell = event.target;
                const row = parseInt(cell.dataset.row);
                const col = parseInt(cell.dataset.col);
                
                if (cell.classList.contains('flag')) return;
                
                if (this.grid[row][col] === 'X') {
                    this.gameOver = true;
                    cell.classList.add('mine');
                    this.revealAllMines();
                    this.stopTimer();
                    alert('Game Over! You hit a mine!');
                } else {
                    this.revealCell(row, col);
                    if (this.checkWin()) {
                        this.gameOver = true;
                        this.stopTimer();
                        alert('Congratulations! You won!');
                    }
                }
            }

            handleRightClick(event) {
                event.preventDefault();
                if (this.gameOver) return;
                
                const cell = event.target;
                if (cell.classList.contains('revealed')) return;
                
                if (cell.classList.contains('flag')) {
                    cell.classList.remove('flag');
                    this.flagsLeft++;
                } else {
                    if (this.flagsLeft === 0) return;
                    cell.classList.add('flag');
                    this.flagsLeft--;
                }
                
                document.getElementById('mines-left').textContent = this.flagsLeft;
            }

            revealCell(row, col) {
                const cell = this.getCell(row, col);
                if (cell.classList.contains('revealed')) return;
                
                cell.classList.add('revealed');
                const value = this.grid[row][col];
                
                if (value > 0) {
                    cell.textContent = value;
                    cell.classList.add(`number-${value}`);
                }
                
                if (value === 0) {
                    for (let i = -1; i <= 1; i++) {
                        for (let j = -1; j <= 1; j++) {
                            if (row+i >= 0 && row+i < this.rows && 
                                col+j >= 0 && col+j < this.cols) {
                                this.revealCell(row+i, col+j);
                            }
                        }
                    }
                }
            }

            getCell(row, col) {
                return document.querySelector(`[data-row="${row}"][data-col="${col}"]`);
            }

            revealAllMines() {
                for (let row = 0; row < this.rows; row++) {
                    for (let col = 0; col < this.cols; col++) {
                        if (this.grid[row][col] === 'X') {
                            const cell = this.getCell(row, col);
                            cell.classList.add('mine');
                        }
                    }
                }
            }

            startTimer() {
                this.startTime = Date.now();
                this.timerInterval = setInterval(() => {
                    const seconds = Math.floor((Date.now() - this.startTime) / 1000);
                    document.getElementById('timer').textContent = seconds;
                }, 1000);
            }

            stopTimer() {
                clearInterval(this.timerInterval);
            }

            checkWin() {
                for (let row = 0; row < this.rows; row++) {
                    for (let col = 0; col < this.cols; col++) {
                        const cell = this.getCell(row, col);
                        if (this.grid[row][col] !== 'X' && !cell.classList.contains('revealed')) {
                            return false;
                        }
                    }
                }
                return true;
            }
        }

        // Start the game
        const game = new Minesweeper();
    </script>
</body>
</html>
