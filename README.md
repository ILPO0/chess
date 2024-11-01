<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Шахматы</title>
    <style>
        table {
            border-collapse: collapse;
        }
        td {
            width: 50px;
            height: 50px;
            text-align: center;
            vertical-align: middle;
            font-size: 24px;
        }
        .black {
            background-color: #3b3b3b;
        }
        .white {
            background-color: #f0d9b5;
        }
    </style>
</head>
<body>
    <h1>Шахматы</h1>
    <table id="chessboard"></table>
    <p>Введите ход (например, "e2 e4" или "e3 e7" для стрельбы):</p>
    <input type="text" id="moveInput" onkeypress="handleKeyPress(event)">

    <script>
        const SIZE = 8;
        const EMPTY = ' ';
        const PAWN_WHITE = '♙';
        const PAWN_BLACK = '♟';
        const ROOK_WHITE = '♖';
        const ROOK_BLACK = '♜';
        const KNIGHT_WHITE = '♘';
        const KNIGHT_BLACK = '♞';
        const BISHOP_WHITE = '♗';
        const BISHOP_BLACK = '♝';
        const QUEEN_WHITE = '♕';
        const QUEEN_BLACK = '♛';
        const KING_WHITE = '♔';
        const KING_BLACK = '♚';
        const STRELOK_WHITE = '🏹'; // Стрелок для белых
        const STRELOK_BLACK = '🏹'; // Стрелок для черных

        let board = [
            [ROOK_BLACK, KNIGHT_BLACK, STRELOK_BLACK, QUEEN_BLACK, KING_BLACK, BISHOP_BLACK, KNIGHT_BLACK, ROOK_BLACK],
            [PAWN_BLACK, PAWN_BLACK, PAWN_BLACK, PAWN_BLACK, PAWN_BLACK, PAWN_BLACK, PAWN_BLACK, PAWN_BLACK],
            [' ', ' ', ' ', ' ', ' ', ' ', ' ', ' '],
            [' ', ' ', ' ', ' ', ' ', ' ', ' ', ' '],
            [' ', ' ', ' ', ' ', ' ', ' ', ' ', ' '],
            [' ', ' ', ' ', ' ', ' ', ' ', ' ', ' '],
            [PAWN_WHITE, PAWN_WHITE, PAWN_WHITE, PAWN_WHITE, PAWN_WHITE, PAWN_WHITE, PAWN_WHITE, PAWN_WHITE],
            [ROOK_WHITE, KNIGHT_WHITE, STRELOK_WHITE, QUEEN_WHITE, KING_WHITE, BISHOP_WHITE, KNIGHT_WHITE, ROOK_WHITE],
        ];

        let strelokCanShoot = {
            white: true,
            black: true
        };

        function drawBoard() {
            const table = document.getElementById('chessboard');
            table.innerHTML = '';
            for (let row = 0; row < SIZE; row++) {
                const tr = document.createElement('tr');
                for (let col = 0; col < SIZE; col++) {
                    const td = document.createElement('td');
                    td.className = (row + col) % 2 === 0 ? 'white' : 'black';
                    td.innerText = board[row][col];
                    tr.appendChild(td);
                }
                table.appendChild(tr);
            }
        }

        function handleKeyPress(event) {
            if (event.key === 'Enter') {
                makeMove();
                event.target.value = ''; // Очистка поля ввода после хода
            }
        }

        function makeMove() {
            const input = document.getElementById('moveInput').value;
            const [start, end] = input.split(' ');

            if (start.length !== 2 || end.length !== 2) {
                alert('Неверный формат. Используйте "e2 e4".');
                return;
            }

            const startRow = 8 - parseInt(start[1]);
            const startCol = start[0].charCodeAt(0) - 'a'.charCodeAt(0);
            const endRow = 8 - parseInt(end[1]);
            const endCol = end[0].charCodeAt(0) - 'a'.charCodeAt(0);

            if (isMoveValid(startRow, startCol, endRow, endCol)) {
                // Если это стрелок и он стреляет
                if (board[startRow][startCol] === STRELOK_WHITE || board[startRow][startCol] === STRELOK_BLACK) {
                    if (startRow !== endRow && startCol !== endCol && Math.abs(startRow - endRow) > 1 && Math.abs(startCol - endCol) > 1) {
                        // Проверяем, есть ли фигура для уничтожения
                        if (board[endRow][endCol] !== EMPTY) {
                            board[endRow][endCol] = EMPTY; // Уничтожаем фигуру
                            strelokCanShoot[board[startRow][startCol] === STRELOK_WHITE ? 'white' : 'black'] = false; // Стрелок не может стрелять снова
                        }
                    } else {
                        alert('Стрелок не может стрелять в радиусе одной клетки.');
                        return;
                    }
                }

                // Если стрелок перезаряжается
                if (!strelokCanShoot[board[startRow][startCol] === STRELOK_WHITE ? 'white' : 'black']) {
                    board[startRow][startCol] = EMPTY; // Убираем стрелка с предыдущей позиции
                    board[endRow][endCol] = board[startRow][startCol]; // Перемещаем стрелка
                    strelokCanShoot[board[startRow][startCol] === STRELOK_WHITE ? 'white' : 'black'] = true; // Теперь стрелок может стрелять снова
                } else {
                    board[endRow][endCol] = board[startRow][startCol]; // Перемещаем стрелка
                    board[startRow][startCol] = EMPTY; // Убираем стрелка с предыдущей позиции
                }

                drawBoard();
            } else {
                alert('Неверный ход.');
            }
        }

        function isMoveValid(startRow, startCol, endRow, endCol) {
            const piece = board[startRow][startCol];
            if (piece === EMPTY) {
                return false; // Нет фигуры на старой позиции
            }

            // Логика для стрелка
            if (piece === STRELOK_WHITE || piece === STRELOK_BLACK) {
                const isShooting = board[endRow][endCol] !== EMPTY && startRow !== endRow && startCol !== endCol;
                const isMove = Math.abs(startRow - endRow) <= 1 && Math.abs(startCol - endCol) <= 1;
                return isShooting || isMove; // Стрелок может стрелять или двигаться
            }

            // Логика для пешки
            if (piece === PAWN_WHITE) {
                if (startCol === endCol) {
                    if (startRow === 6 && endRow === 4) return board[5][startCol] === EMPTY && board[endRow][endCol] === EMPTY; // Двойной шаг
                    return endRow === startRow - 1 && board[endRow][endCol] === EMPTY; // Обычный шаг
                }
                return (endRow === startRow - 1 && Math.abs(endCol - startCol) === 1 && board[endRow][endCol] !== EMPTY); // Удар
            }
            if (piece === PAWN_BLACK) {
                if (startCol === endCol) {
                    if (startRow === 1 && endRow === 3) return board[2][startCol] === EMPTY && board[endRow][endCol] === EMPTY; // Двойной шаг
                    return endRow === startRow + 1 && board[endRow][endCol] === EMPTY; // Обычный шаг
                }
                return (endRow === startRow + 1 && Math.abs(endCol - startCol) === 1 && board[endRow][endCol] !== EMPTY); // Удар
            }

            // Логика для ладьи
            if (piece === ROOK_WHITE || piece === ROOK_BLACK) {
                if (startRow === endRow) {
                    for (let i = Math.min(startCol, endCol) + 1; i < Math.max(startCol, endCol); i++) {
                        if (board[startRow][i] !== EMPTY) return false; // Проверка на препятствия
                    }
                    return true;
                }
                if (startCol === endCol) {
                    for (let i = Math.min(startRow, endRow) + 1; i < Math.max(startRow, endRow); i++) {
                        if (board[i][startCol] !== EMPTY) return false; // Проверка на препятствия
                    }
                    return true;
                }
                return false;
            }

            // Логика для коня
            if (piece === KNIGHT_WHITE || piece === KNIGHT_BLACK) {
                return (Math.abs(startRow - endRow) === 2 && Math.abs(startCol - endCol) === 1) || 
                       (Math.abs(startRow - endRow) === 1 && Math.abs(startCol - endCol) === 2);
            }

            // Логика для слона
            if (piece === BISHOP_WHITE || piece === BISHOP_BLACK) {
                if (Math.abs(startRow - endRow) === Math.abs(startCol - endCol)) {
                    const rowIncrement = (endRow > startRow) ? 1 : -1;
                    const colIncrement = (endCol > startCol) ? 1 : -1;
                    let r = startRow + rowIncrement;
                    let c = startCol + colIncrement;
                    while (r !== endRow && c !== endCol) {
                        if (board[r][c] !== EMPTY) return false; // Проверка на препятствия
                        r += rowIncrement;
                        c += colIncrement;
                    }
                    return true;
                }
                return false;
            }

            // Логика для ферзя
            if (piece === QUEEN_WHITE || piece === QUEEN_BLACK) {
                if (startRow === endRow || startCol === endCol) {
                    // Логика как у ладьи
                    return isMoveValid(startRow, startCol, endRow, endCol); // Используем логику для ладьи
                } else if (Math.abs(startRow - endRow) === Math.abs(startCol - endCol)) {
                    // Логика как у слона
                    return isMoveValid(startRow, startCol, endRow, endCol); // Используем логику для слона
                }
                return false;
            }

            // Логика для короля
            if (piece === KING_WHITE || piece === KING_BLACK) {
                return Math.abs(startRow - endRow) <= 1 && Math.abs(startCol - endCol) <= 1; // Король ходит на 1 клетку
            }

            return false; // Не допустимый ход
        }

        drawBoard();
    </script>
</body>
</html>
