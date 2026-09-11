
Тема проєкту: Трекер води (Water Tracker) 

## 1. Планування
Мета продукту:  
Створити простий та інтуїтивний додаток «Трекер води», який допомагає користувачам контролювати свій щоденний водний баланс та легко додавати випиті склянки води протягом дня.

## 2. Аналіз вимог (User Stories)

1. Як користувач, я хочу додавати випиту склянку води в один клік, щоб швидко оновлювати свій денний прогрес. ✅ (Must have)
2. Як користувач, я хочу бачити загальну кількість випитої води та денну норму на головному екрані, щоб контролювати свій баланс. ✅ (Must have)
3. Як користувач, я хочу скасовувати останнє додавання склянки, якщо я натиснув кнопку випадково.
4. Як користувач, я хочу налаштовувати свою денну норму води (в мл або склянках), щоб вона відповідала моїм індивідуальним потребам.
5. Як користувач, я хочу переглядати історію споживання води за тиждень, щоб аналізувати свої звички.

---

## 3. Дизайн (Прототип)

![Опис картинки](images/water-tracker.png)

## 4. Реалізація (Псевдокод)

Алгоритм функціоналу додавання випитої склянки води:

<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Трекер води</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: #e0f7fa;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            background-color: #ffffff;
            width: 350px;
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            text-align: center;
            position: relative;
        }

        h1 {
            color: #0077b6;
            font-size: 24px;
            margin-bottom: 20px;
        }

        /* Екрани */
        .screen {
            display: none;
        }

        .screen.active {
            display: block;
        }

        /* Елементи головного екрана */
        .progress-box {
            position: relative;
            width: 150px;
            height: 150px;
            border-radius: 50%;
            background: conic-gradient(#00b4d8 0deg, #caf0f8 0deg);
            margin: 0 auto 20px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .progress-inner {
            width: 120px;
            height: 120px;
            background-color: white;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        .percentage {
            font-size: 24px;
            font-weight: bold;
            color: #0077b6;
        }

        .water-info {
            font-size: 18px;
            color: #333;
            margin-bottom: 20px;
        }

        .btn {
            background-color: #0077b6;
            color: white;
            border: none;
            padding: 12px 20px;
            font-size: 16px;
            border-radius: 10px;
            cursor: pointer;
            width: 100%;
            margin-bottom: 10px;
            transition: background 0.2s;
        }

        .btn:hover {
            background-color: #0096c7;
        }

        .btn-secondary {
            background-color: #90e0ef;
            color: #03045e;
        }

        .btn-secondary:hover {
            background-color: #ade8f4;
        }

        .btn-danger {
            background-color: #ef233c;
            margin-top: 10px;
        }

        /* Налаштування */
        .input-group {
            text-align: left;
            margin-bottom: 15px;
        }

        .input-group label {
            display: block;
            margin-bottom: 5px;
            color: #555;
            font-size: 14px;
        }

        .input-group input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 8px;
            font-size: 16px;
        }

        .message {
            color: #2b9348;
            font-weight: bold;
            margin-top: 10px;
            min-height: 20px;
        }
    </style>
</head>
<body>

<div class="container">
    <!-- ЕКРАН 1: Головний -->
    <div id="main-screen" class="screen active">
        <h1>Трекер води 💧</h1>
        
        <div class="progress-box" id="progress-circle">
            <div class="progress-inner">
                <span class="percentage" id="percent-text">0%</span>
            </div>
        </div>

        <div class="water-info">
            <span id="current-water">0</span> / <span id="goal-water">2000</span> мл
        </div>

        <button class="btn" id="add-glass-btn">+ Додати склянку (<span id="glass-size-text">200</span> мл)</button>
        <button class="btn btn-secondary" onclick="showScreen('settings-screen')">⚙️ Налаштування</button>
        <div class="message" id="status-message"></div>
    </div>

    <!-- ЕКРАН 2: Налаштування -->
    <div id="settings-screen" class="screen">
        <h1>Налаштування ⚙️</h1>
        
        <div class="input-group">
            <label for="goal-input">Денна норма (мл):</label>
            <input type="number" id="goal-input" value="2000">
        </div>

        <div class="input-group">
            <label for="glass-input">Об'єм однієї склянки (мл):</label>
            <input type="number" id="glass-input" value="200">
        </div>

        <button class="btn" onclick="saveSettings()">Зберегти</button>
        <button class="btn btn-secondary" onclick="showScreen('main-screen')">Назад</button>
        <button class="btn btn-danger" onclick="resetData()">Скинути прогрес за день</button>
    </div>
</div>

<script>
    // Початкові дані (стан)
    let currentWater = parseInt(localStorage.getItem('currentWater')) || 0;
    let dailyGoal = parseInt(localStorage.getItem('dailyGoal')) || 2000;
    let glassSize = parseInt(localStorage.getItem('glassSize')) || 200;

    // Елементи DOM
    const currentWaterEl = document.getElementById('current-water');
    const goalWaterEl = document.getElementById('goal-water');
    const percentTextEl = document.getElementById('percent-text');
    const progressCircleEl = document.getElementById('progress-circle');
    const glassSizeTextEl = document.getElementById('glass-size-text');
    const statusMessageEl = document.getElementById('status-message');
    
    const goalInput = document.getElementById('goal-input');
    const glassInput = document.getElementById('glass-input');

    // Оновлення інтерфейсу
    function updateUI() {
        currentWaterEl.innerText = currentWater;
        goalWaterEl.innerText = dailyGoal;
        glassSizeTextEl.innerText = glassSize;
        
        goalInput.value = dailyGoal;
        glassInput.value = glassSize;

        // Розрахунок відсотків
        let percentage = Math.round((currentWater / dailyGoal) * 100);
        percentTextEl.innerText = `${percentage}%`;

        // Оновлення кругового прогрес-бару
        let degree = Math.min((percentage / 100) * 360, 360);
        progressCircleEl.style.background = `conic-gradient(#00b4d8 ${degree}deg, #caf0f8 ${degree}deg)`;

        // Перевірка досягнення цілі
        if (currentWater >= dailyGoal && dailyGoal > 0) {
            statusMessageEl.innerText = "🎉 Вітаємо! Норму досягнуто!";
        } else {
            statusMessageEl.innerText = "";
        }

        // Збереження в LocalStorage
        localStorage.setItem('currentWater', currentWater);
        localStorage.setItem('dailyGoal', dailyGoal);
        localStorage.setItem('glassSize', glassSize);
    }

    // Додавання склянки води (Логіка реалізації з лаб. роботи)
    document.getElementById('add-glass-btn').addEventListener('click', () => {
        if (glassSize <= 0) {
            alert("Об'єм склянки має бути більше 0!");
            return;
        }
        currentWater += glassSize;
        updateUI();
    });

    // Перемикання екранів
    function showScreen(screenId) {
        document.querySelectorAll('.screen').forEach(screen => {
            screen.classList.remove('active');
        });
        document.getElementById(screenId).classList.add('active');
    }

    // Збереження налаштувань
    function saveSettings() {
        let newGoal = parseInt(goalInput.value);
        let newGlass = parseInt(glassInput.value);

        if (newGoal > 0 && newGlass > 0) {
            dailyGoal = newGoal;
            glassSize = newGlass;
            updateUI();
            showScreen('main-screen');
        } else {
            alert("Будь ласка, введіть коректні значення більше 0.");
        }
    }

    // Скидання даних
    function resetData() {
        if (confirm("Ви впевнені, що хочете скинути лічильник води за сьогодні?")) {
            currentWater = 0;
            updateUI();
            showScreen('main-screen');
        }
    }

    updateUI();
</script>

</body>
</html>

## 5. Тестування

Додавання склянки води (200 мл):
Початковий стан: 0 мл.
Дія: Натискання кнопки «+ Додати склянку».
Очікуваний результат: Лічильник збільшується до 200 мл, прогрес-бар оновлюється.
Додавання від'ємного або нульового значення:
Дія: Спроба передати об'єм склянки ≤0 мл.
Очікуваний результат: Система видає повідомлення про помилку і не змінює поточний прогрес.
Досягнення денної норми (наприклад, 2000 мл):
Початковий стан: 1800 мл при нормі 2000 мл.
Дія: Натискання «+ Додати склянку (200 мл)».
Очікуваний результат: Лічильник стає 2000 мл (100%), з'являється вітальне повідомлення про виконання норми.

## 6. Висновки

Для розробки додатку «Трекер води» найоптимальнішим є використання Agile (Scrum / Kanban).
Оскільки додаток має простий базовий функціонал (MVP), Agile дозволить дуже швидко випустити першу працюючу версію (додавання склянки та лічильник), після чого гнучко додавати нові можливості на основі відгуків користувачів.
Модель Waterfall була б занадто жорсткою для швидкого запуску, а Spiral — надмірно складною та дорогою для проєкту такого масштабу.