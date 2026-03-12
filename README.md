<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dire Straits of Hormuz - Definitive Edition</title>
    <link href="https://fonts.googleapis.com/css2?family=Aref+Ruqaa:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            background-color: #2c3e50;
            color: #ecf0f1;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
        }

        h1 {
            font-family: 'Aref Ruqaa', serif;
            font-size: 3.5rem;
            color: #f1c40f;
            text-shadow: 2px 2px 4px #000;
            margin-bottom: 5px;
            text-align: center;
        }

        p.subtitle {
            margin-top: 0;
            margin-bottom: 20px;
            font-style: italic;
            color: #bdc3c7;
        }

        .game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .status-bar {
            display: flex;
            justify-content: space-between;
            width: 100%;
            margin-bottom: 10px;
            font-size: 1.2rem;
            font-weight: bold;
            background-color: #34495e;
            padding: 10px 15px;
            border-radius: 5px;
            box-sizing: border-box;
            border: 2px solid #f1c40f;
        }

        #board {
            display: grid;
            gap: 1px;
            background-color: #34495e;
            padding: 5px;
            border: 3px solid #f1c40f;
            border-radius: 5px;
            box-shadow: 0px 10px 30px rgba(0,0,0,0.5);
        }

        .cell {
            width: 22px;
            height: 22px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            font-size: 13px;
            cursor: pointer;
            user-select: none;
            box-sizing: border-box;
        }

        .land {
            background-color: #8B5A2B;
            cursor: not-allowed;
            border: 1px solid #6b441f;
        }

        .water {
            background-color: #2980b9;
            border: 2px outset #3498db;
        }

        .water:hover {
            background-color: #3498db;
        }

        .revealed {
            background-color: #1abc9c;
            border: 1px solid #16a085;
            cursor: default;
        }

        .mine {
            background-color: #e74c3c !important;
            font-size: 16px;
        }

        /* Modal / Game Over Message */
        #modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background-color: rgba(0,0,0,0.8);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }

        .modal-content {
            background-color: #ecf0f1;
            color: #2c3e50;
            padding: 30px;
            border-radius: 10px;
            max-width: 500px;
            text-align: center;
            border: 4px solid #c0392b;
        }

        .modal-content h2 {
            font-family: 'Aref Ruqaa', serif;
            color: #c0392b;
            font-size: 2rem;
        }

        button {
            background-color: #f1c40f;
            color: #2c3e50;
            font-weight: bold;
            border: none;
            padding: 8px 15px;
            font-size: 1rem;
            cursor: pointer;
            border-radius: 5px;
        }

        button:hover {
            background-color: #f39c12;
        }
    </style>
</head>
<body>

    <h1>Dire Straits of Hormuz</h1>
    <p class="subtitle">Klik voor Sonar. Rechtermuisklik voor een 🇺🇸 vlag.</p>

    <div class="game-container">
        <div class="status-bar">
            <span>Mijnen: <span id="mines-count">0</span></span>
            <button onclick="initGame()">Herstart Patrouille</button>
        </div>
        <div id="board"></div>
    </div>

    <div id="modal">
        <div class="modal-content">
            <h2>Bismillah!</h2>
            <p><strong>Bericht van de Hoogste Leider:</strong></p>
            <p><em>"De Grote Satan heeft wederom gefaald! Jullie roekeloze arrogantie was geen partij voor onze onzichtbare verdediging. Jullie imperialistische schepen liggen nu op de bodem van onze heilige wateren. De Straat van Hormuz behoort toe aan de rechtvaardigen! Game Over."</em></p>
            <button onclick="document.getElementById('modal').style.display='none'; initGame();" style="margin-top:15px;">Probeer Opnieuw te Infiltreren</button>
        </div>
    </div>

    <script>
        // Visuele map: '.' is land, '1' is water (gebaseerd op je screenshot)
        const rawMap = [
            "1..........................11111111...................",
            "11.................11111111111111111..................",
            "111......1........1111111111111111111.................",
            "11111...11111111111111111111111111111.................",
            "111111.111111111111111111111111111111.................",
            "1111111111111111111111111111111111111.................",
            "1111111111111111111111111111111111111.................",
            "11111111111111111111111111111111111111................",
            "1111111111111111111111...1111111111111................",
            "11111111111111111111.......111111111111...............",
            "1111111111111111111.........1111111111111.............",
            "111111111111111111...........11111111111111...........",
            "11111111111111111.............111111111111111111111111",
            "111111111111111................11111111111111111111111",
            "1111111111111..................11111111111111111111111",
            "11111111111....................11111111111111111111111",
            "111111111......................11111111111111111111111",
            "11111..........................11111111111111111111111",
            "...............................11111111111111111111111",
            "................................1111111111111111111111",
            ".................................111111111111111111111",
            "...................................1111111111111111111",
            ".....................................11111111111111111",
            ".......................................111111111111111"
        ];

        const ROWS = rawMap.length;
        const COLS = rawMap[0].length;
        const TOTAL_MINES = 85; 

        let grid = [];
        let minesLeft = TOTAL_MINES;
        let gameOver = false;

        function initGame() {
            const boardElement = document.getElementById('board');
            boardElement.innerHTML = '';
            
            // Dynamische grid-grootte instellen op basis van de array
            boardElement.style.gridTemplateColumns = `repeat(${COLS}, 22px)`;
            boardElement.style.gridTemplateRows = `repeat(${ROWS}, 22px)`;

            grid = [];
            minesLeft = TOTAL_MINES;
            gameOver = false;
            document.getElementById('mines-count').innerText = minesLeft;

            for (let r = 0; r < ROWS; r++) {
                let row = [];
                for (let c = 0; c < COLS; c++) {
                    row.push({
                        isWater: rawMap[r][c] === '1',
                        isMine: false,
                        isRevealed: false,
                        isFlagged: false,
                        neighborMines: 0
                    });
                }
                grid.push(row);
            }

            let minesPlaced = 0;
            while (minesPlaced < TOTAL_MINES) {
                let r = Math.floor(Math.random() * ROWS);
                let c = Math.floor(Math.random() * COLS);
                
                if (grid[r][c].isWater && !grid[r][c].isMine) {
                    grid[r][c].isMine = true;
                    minesPlaced++;
                }
            }

            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    if (grid[r][c].isWater && !grid[r][c].isMine) {
                        grid[r][c].neighborMines = countNeighborMines(r, c);
                    }
                }
            }

            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    const cellEl = document.createElement('div');
                    cellEl.dataset.r = r;
                    cellEl.dataset.c = c;
                    
                    if (grid[r][c].isWater) {
                        cellEl.className = 'cell water';
                        cellEl.addEventListener('click', () => revealCell(r, c));
                        cellEl.addEventListener('contextmenu', (e) => {
                            e.preventDefault();
                            toggleFlag(r, c);
                        });
                    } else {
                        cellEl.className = 'cell land';
                    }
                    boardElement.appendChild(cellEl);
                }
            }
        }

        function countNeighborMines(r, c) {
            let count = 0;
            for (let i = -1; i <= 1; i++) {
                for (let j = -1; j <= 1; j++) {
                    let nr = r + i;
                    let nc = c + j;
                    if (nr >= 0 && nr < ROWS && nc >= 0 && nc < COLS) {
                        if (grid[nr][nc].isMine) count++;
                    }
                }
            }
            return count;
        }

        function getCellElement(r, c) {
            return document.querySelector(`.cell[data-r='${r}'][data-c='${c}']`);
        }

        function toggleFlag(r, c) {
            if (gameOver || grid[r][c].isRevealed) return;
            
            let cell = grid[r][c];
            let cellEl = getCellElement(r, c);

            if (!cell.isFlagged) {
                cell.isFlagged = true;
                cellEl.innerHTML = '🇺🇸';
                minesLeft--;
            } else {
                cell.isFlagged = false;
                cellEl.innerHTML = '';
                minesLeft++;
            }
            document.getElementById('mines-count').innerText = minesLeft;
            checkWin();
        }

        function revealCell(r, c) {
            if (gameOver) return;
            let cell = grid[r][c];
            if (cell.isRevealed || cell.isFlagged) return;

            cell.isRevealed = true;
            let cellEl = getCellElement(r, c);
            cellEl.classList.remove('water');
            cellEl.classList.add('revealed');

            if (cell.isMine) {
                cellEl.classList.add('mine');
                cellEl.innerHTML = '💥';
                triggerGameOver();
                return;
            }

            if (cell.neighborMines > 0) {
                cellEl.innerHTML = cell.neighborMines;
                const colors = ['#2980b9', '#27ae60', '#e74c3c', '#8e44ad', '#d35400', '#16a085', '#2c3e50', '#000000'];
                cellEl.style.color = colors[cell.neighborMines - 1];
            } else {
                for (let i = -1; i <= 1; i++) {
                    for (let j = -1; j <= 1; j++) {
                        let nr = r + i;
                        let nc = c + j;
                        if (nr >= 0 && nr < ROWS && nc >= 0 && nc < COLS) {
                            if (grid[nr][nc].isWater) {
                                revealCell(nr, nc);
                            }
                        }
                    }
                }
            }
            checkWin();
        }

        function triggerGameOver() {
            gameOver = true;
            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    if (grid[r][c].isMine) {
                        let el = getCellElement(r, c);
                        el.innerHTML = '💣';
                        el.style.backgroundColor = '#e74c3c';
                    }
                }
            }
            setTimeout(() => {
                document.getElementById('modal').style.display = 'flex';
            }, 500);
        }

        function checkWin() {
            let won = true;
            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    if (grid[r][c].isWater) {
                        if (!grid[r][c].isMine && !grid[r][c].isRevealed) {
                            won = false;
                        }
                    }
                }
            }
            if (won) {
                gameOver = true;
                setTimeout(() => alert("Gefeliciteerd, Commandeur! De Straat van Hormuz is veiliggesteld."), 100);
            }
        }

        window.onload = initGame;
    </script>
</body>
</html>
