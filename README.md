# Mines predictor.github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mines Game Predictor</title>
    <style>
        :root {
            --primary-bg: linear-gradient(135deg, #1f4037, #99f2c8);
            --cell-bg: #212529;
            --safe-bg: #28a745;
            --mine-bg: #dc3545;
            --cell-hover: #6c757d;
            --text-color: #f8f9fa;
        }
        body {
            font-family: 'Arial', sans-serif;
            text-align: center;
            margin: 20px;
            color: var(--text-color);
            background: var(--primary-bg);
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        h1 {
            font-size: 2rem;
            margin-bottom: 10px;
        }
        button {
            padding: 10px 20px;
            font-size: 1rem;
            color: var(--text-color);
            background-color: #007bff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-bottom: 20px;
            transition: background-color 0.3s ease;
        }
        button:hover {
            background-color: #0056b3;
        }
        .grid {
            display: grid;
            grid-template-columns: repeat(5, 60px);
            gap: 5px;
        }
        .cell {
            width: 60px;
            height: 60px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5rem;
            border: 1px solid #343a40;
            cursor: pointer;
            background-color: var(--cell-bg);
            color: var(--text-color);
            border-radius: 5px;
            transition: background-color 0.3s ease;
        }
        .cell:hover {
            background-color: var(--cell-hover);
        }
        .cell.mine {
            background-color: var(--mine-bg);
        }
        .cell.safe {
            background-color: var(--safe-bg);
        }
        .predicted-safe {
            background-color: #ffc107 !important;
            color: #000;
        }
    </style>
</head>
<body>
    <h1>Mines Game Predictor</h1>
    <button onclick="generateGrid()">Generate New Grid</button>
    <button id="predict-button">Predict Next Safe Tiles</button>
    <div id="grid-container" class="grid"></div>

    <script>
        const gridSize = 5; // 5x5 grid
        const numMines = 3; // Number of mines
        const markedMines = [];
        let grid = [];

        function generateGrid() {
            markedMines.length = 0; // Clear previous marks
            grid = Array.from({ length: gridSize }, () => Array(gridSize).fill(null));
            const container = document.getElementById('grid-container');
            container.innerHTML = '';
            container.style.gridTemplateColumns = `repeat(${gridSize}, 60px)`;

            for (let row = 0; row < gridSize; row++) {
                for (let col = 0; col < gridSize; col++) {
                    const cell = document.createElement('div');
                    cell.classList.add('cell');
                    cell.dataset.row = row;
                    cell.dataset.col = col;

                    cell.addEventListener('click', () => markMine(cell, row, col));
                    container.appendChild(cell);
                }
            }
        }

        function markMine(cell, row, col) {
            if (markedMines.length >= numMines && !cell.classList.contains('mine')) {
                alert(`You can only mark ${numMines} mines.`);
                return;
            }

            if (cell.classList.contains('mine')) {
                cell.classList.remove('mine');
                const index = markedMines.findIndex(m => m.row === row && m.col === col);
                markedMines.splice(index, 1);
            } else {
                cell.classList.add('mine');
                markedMines.push({ row, col });
            }
        }

        function predictSafeTiles(grid, gridSize, numMines) {
            const simulations = 1000; // Number of simulations
            const safeCounts = Array.from({ length: gridSize }, () =>
                Array(gridSize).fill(0)
            );

            for (let sim = 0; sim < simulations; sim++) {
                const simGrid = Array.from({ length: gridSize }, () =>
                    Array(gridSize).fill(false)
                );
                let minesPlaced = 0;

                while (minesPlaced < numMines) {
                    const row = Math.floor(Math.random() * gridSize);
                    const col = Math.floor(Math.random() * gridSize);
                    if (!simGrid[row][col]) {
                        simGrid[row][col] = true; // Place a mine
                        minesPlaced++;
                    }
                }

                for (let row = 0; row < gridSize; row++) {
                    for (let col = 0; col < gridSize; col++) {
                        if (!simGrid[row][col]) {
                            safeCounts[row][col]++;
                        }
                    }
                }
            }

            const probabilities = safeCounts.map(row =>
                row.map(count => count / simulations)
            );

            const safeTiles = [];
            probabilities.forEach((row, rowIndex) => {
                row.forEach((prob, colIndex) => {
                    safeTiles.push({ row: rowIndex, col: colIndex, prob });
                });
            });

            safeTiles.sort((a, b) => b.prob - a.prob);
            return safeTiles.slice(0, 5);
        }

        function highlightPredictedSafeTiles(gridContainer, gridSize, safeTiles) {
            const cells = gridContainer.querySelectorAll('.cell');
            cells.forEach(cell => cell.classList.remove('predicted-safe'));

            safeTiles.forEach(tile => {
                const index = tile.row * gridSize + tile.col;
                const cell = cells[index];
                cell.classList.add('predicted-safe');
                cell.textContent = '🌟';
            });
        }

        document.getElementById('predict-button').addEventListener('click', () => {
            if (markedMines.length !== numMines) {
                alert('Please mark exactly 3 mines before predicting!');
                return;
            }

            const safeTiles = predictSafeTiles(grid, gridSize, numMines);
            highlightPred
ictedSafeTiles(document.getElementById('grid-container'), gridSize, safeTiles);
        });

        // Initialize the grid on page load
        generateGrid();
    </script>
</body>
</html>
