<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Челлендж чтения</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 20px;
        }
        h1 {
            margin-bottom: 20px;
        }
        .circle {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            max-width: 600px;
            margin: 0 auto;
        }
        .cell {
            width: 40px;
            height: 40px;
            margin: 5px;
            border: 2px solid black;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            user-select: none;
            transition: background-color 0.3s ease;
        }
        .form-section {
            margin-bottom: 20px;
        }
        .form-section input[type="text"],
        .form-section input[type="color"] {
            margin: 5px;
            padding: 5px;
            font-size: 16px;
        }
        .book-list {
            margin-top: 20px;
        }
        .book-item {
            margin: 5px 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .book-item input {
            margin-left: 10px;
        }
        button {
            margin: 10px;
            padding: 10px 15px;
            font-size: 16px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Челлендж чтения</h1>

    <div class="form-section">
        <input type="text" id="bookName" placeholder="Название книги">
        <input type="color" id="bookColor" value="#ff0000">
        <button onclick="addBook()">Добавить книгу</button>
        <button onclick="clearProgress()">Очистить прогресс</button>
        <button onclick="toggleChallenge()">Переключить челлендж</button>
    </div>

    <div class="book-list" id="bookList"></div>

    <div class="circle" id="circle">
        <!-- Генерация ячеек -->
    </div>

    <script>
        let currentChallenge = "month"; // month или year
        const daysInMonth = 30; // Количество дней в месячном челлендже
        const daysInYear = 365; // Количество дней в годовом челлендже
        const circle = document.getElementById('circle');
        const bookList = document.getElementById('bookList');
        const bookNameInput = document.getElementById('bookName');
        const bookColorInput = document.getElementById('bookColor');

        let books = JSON.parse(localStorage.getItem('books')) || {};
        let progress = JSON.parse(localStorage.getItem('progress')) || {};

        function renderBooks() {
            bookList.innerHTML = '';
            for (const [name, color] of Object.entries(books)) {
                const bookItem = document.createElement('div');
                bookItem.classList.add('book-item');

                const bookText = document.createElement('span');
                bookText.textContent = `${name}`;

                const colorInput = document.createElement('input');
                colorInput.type = 'color';
                colorInput.value = color;
                colorInput.addEventListener('change', (e) => {
                    books[name] = e.target.value;
                    updateCellColors(name, e.target.value);
                    saveData();
                });

                const deleteButton = document.createElement('button');
                deleteButton.textContent = "Удалить";
                deleteButton.style.marginLeft = "10px";
                deleteButton.addEventListener('click', () => {
                    delete books[name];
                    saveData();
                    renderBooks();
                });

                bookItem.appendChild(bookText);
                bookItem.appendChild(colorInput);
                bookItem.appendChild(deleteButton);
                bookList.appendChild(bookItem);
            }
        }

        function addBook() {
            const name = bookNameInput.value.trim();
            const color = bookColorInput.value;
            if (name) {
                books[name] = color;
                bookNameInput.value = '';
                saveData();
                renderBooks();
            }
        }

        function updateCellColors(bookName, newColor) {
            for (let i = 1; i <= (currentChallenge === "month" ? daysInMonth : daysInYear); i++) {
                if (progress[i] === books[bookName]) {
                    const cell = document.querySelector(`.cell[data-day="${i}"]`);
                    if (cell) {
                        cell.style.backgroundColor = newColor;
                    }
                }
            }
        }

        function clearProgress() {
            progress = {};
            saveData();
            loadData();
        }

        function saveData() {
            localStorage.setItem('books', JSON.stringify(books));
            localStorage.setItem('progress', JSON.stringify(progress));
        }

        function loadData() {
            circle.innerHTML = ''; // Очистка текущих ячеек
            const days = currentChallenge === "month" ? daysInMonth : daysInYear;
            for (let i = 1; i <= days; i++) {
                const cell = document.createElement('div');
                cell.classList.add('cell');
                cell.textContent = i;
                cell.dataset.day = i;

                if (progress[i]) {
                    cell.style.backgroundColor = progress[i];
                }

                cell.addEventListener('click', () => {
                    const neutralColor = "lightgreen";
                    const bookName = prompt('Введите название книги из списка или оставьте пустым для нейтрального цвета');
                    if (bookName === "") {
                        cell.style.backgroundColor = neutralColor;
                        progress[i] = neutralColor;
                        saveData();
                    } else if (bookName && books[bookName]) {
                        const color = books[bookName];
                        cell.style.backgroundColor = color;
                        progress[i] = color;
                        saveData();
                    }
                });

                circle.appendChild(cell);
            }
        }

        function toggleChallenge() {
            currentChallenge = currentChallenge === "month" ? "year" : "month";
            loadData();
        }

        // Инициализация
        renderBooks();
        loadData();
    </script>
</body>
</html>
