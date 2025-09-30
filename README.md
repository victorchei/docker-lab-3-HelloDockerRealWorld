# Docker Compose з MongoDB - Лабораторна робота 3

## Мета роботи

Створити конфігурацію Docker Compose з трьома сервісами: серверний застосунок `api`, сервіс для баз даних `api_db` та фронтенд.

## Передумови

- Встановлений Docker Compose
- Для перевірки виконайте команду: `docker compose --version`

## Архітектура проекту

Проект розділений на **3 окремі репозиторії**:

### 1. Головний репозиторій (HelloDockerRealWorld)

```
HelloDockerRealWorld/
├── docker-compose.yml       # Конфігурація всіх сервісів
├── README.md                # Інструкції та документація
├── docker-lab-3-frontend/                # Git submodule або клонований репозиторій
├── docker-lab-3-api/                     # Git submodule або клонований репозиторій
└── ...
```

### 2. Frontend репозиторій (docker-lab-3-frontend)

```
docker-lab-3-frontend/
├── index.html               # Головна сторінка
├── README.md                # Документація frontend
├── css/
│   └── styles.css           # Стилі інтерфейсу
└── js/                      # Простий функціональний підхід
    ├── app.js               # Головний файл застосунку
    ├── utils.js             # Утилітарні функції
    └── constants.js         # Константи
```

### 3. API репозиторій (docker-lab-3-api)

```
docker-lab-3-api/
├── Dockerfile               # Docker конфігурація для Node.js
├── package.json             # Залежності Node.js
├── README.md                # Документація API
├── src/
│   ├── app.js               # Головний файл сервера
│   ├── configuration/
│   │   └── index.js         # Конфігурація
│   ├── models/
│   │   └── user.js          # Модель користувача
│   └── routers/
│       └── user.js          # Маршрути для користувачів
```

## Архітектура системи

Наша система складається з **3 основних сервісів**:

1. **Frontend** (порт 8080) - Веб-інтерфейс користувача

   - npx serve веб-сервер для статичних файлів
   - HTML/CSS/JavaScript інтерфейс
   - Проксування API запитів до backend

2. **API** (порт 3001) - Серверна логіка

   - Node.js з Express.js
   - REST API для роботи з користувачами
   - Підключення до MongoDB

3. **Database** - База даних
   - MongoDB для збереження даних користувачів
   - Автоматична ініціалізація та збереження даних

## Frontend архітектура

Фронтенд побудований за простим функціональним підходом:

- **📦 ES6 Modules** - мінімальна модульна структура
- **🔒 Security** - XSS захист
- **⚡ Performance** - без зайвих абстракцій
- **🛠️ Error Handling** - обробка помилок
- **📱 Modern JS** - async/await, template literals

## Хід роботи

### 1. Практична робота

#### Створення простого docker-compose.yml

У кореневій папці проекту створено файл `docker-compose.yml` з таким вмістом:

```yaml
version: '3'
services:
  api:
    build: ./api
    command: npm start
    ports:
      - '3001:3001'
    environment:
      - PORT=3001
      - HOST=localhost
```

#### Основні команди Docker Compose

**Побудова і запуск застосунку:**

```bash
docker compose up -d
```

**Перебудова застосунку (після змін в API коді):**

```bash
docker compose up --build -d
```

**Швидкий перезапуск після змін в frontend (без перебудови):**

```bash
docker compose restart frontend
```

> **Примітка**: Завдяки volume монтуванню, зміни в frontend файлах відразу відображаються без перебудови контейнера.

**Перегляд працюючих контейнерів:**

```bash
docker ps -a
```

**Зупинка контейнерів:**

```bash
docker compose stop
```

**Зупинка і видалення контейнерів:**

```bash
docker compose down
```

### 2. Лабораторна робота 3 - Docker Compose з базою даних

#### Клонування репозиторіїв

**Опція 1: Клонування всіх репозиторіїв окремо**

```bash
# 1. Клонуємо головний репозиторій з docker-compose.yml
git clone https://github.com/victorchei/docker-lab-3-HelloDockerRealWorld.git
cd docker-lab-3-HelloDockerRealWorld

# 2. Клонуємо frontend репозиторій
git clone https://github.com/victorchei/docker-lab-3-frontend.git

# 3. Клонуємо API репозиторій
git clone https://github.com/victorchei/docker-lab-3-api.git
```

**Опція 2: Використання Git submodules (рекомендовано)**

```bash
# 1. Клонуємо головний репозиторій
git clone --recurse-submodules https://github.com/victorchei/docker-lab-3-HelloDockerRealWorld.git
cd docker-lab-3-HelloDockerRealWorld

# Якщо submodules не завантажилися автоматично:
git submodule update --init --recursive
```

**Перевірка структури проекту:**

```bash
ls -la
# Повинно бути:
# - docker-compose.yml
# - README.md
# - frontend/ (папка з frontend кодом)
# - api/ (папка з API кодом)
```

#### Створення docker-compose.yml з трьома сервісами

```yaml
version: '3'
services:
  # Frontend сервіс (Node.js serve для статичних файлів)
  frontend:
    image: node:alpine
    working_dir: /app
    volumes:
      - ./docker-lab-3-frontend:/app
    command: npx serve -s . -l 8080
    ports:
      - '8080:8080'
    depends_on:
      - api

  # API сервіс (Node.js + Express)
  api:
    build: ./docker-lab-3-api
    command: npm start
    ports:
      - '3001:3001'
    environment:
      - PORT=3001
      - HOST=0.0.0.0
      - MONGO_URL=mongodb://api_db:27017/api
    depends_on:
      - api_db

  # База даних (MongoDB)
  api_db:
    image: mongo:latest
```

### 3. Запуск та тестування

#### Запуск системи

```bash
docker-compose up --build -d
```

#### Тестування API

**Додавання нового користувача (POST-запит):**

```bash
curl -X POST http://localhost:3001/users \
  -H "Content-Type: application/json" \
  -d '{"firstName": "Jane", "lastName": "Smith", "email": "jane@gmail.com"}'
```

**Отримання списку користувачів (GET-запит):**

```bash
curl http://localhost:3001/users
```

#### Використання веб-інтерфейсу

**Відкрийте в браузері головну сторінку:**

```
http://localhost:8080
```

На цій сторінці ви можете:

- ✅ Додавати нових користувачів через форму
- 📋 Переглядати список всіх користувачів
- 🔄 Оновлювати список користувачів
- 🎨 Користуватися зручним інтерфейсом

**Або працюйте напряму з API:**

```
http://localhost:3001/users
```

### 4. Змінні середовища

У коді застосунку використовуються змінні оточення:

```javascript
const PORT = process.env.PORT;
const HOST = process.env.HOST;
const MONGO_URL = process.env.MONGO_URL;
```

### 5. Приклади тестових запитів

**Додавання користувачів через curl:**

```bash
# Користувач 1
curl -X POST http://localhost:3001/users \
  -H "Content-Type: application/json" \
  -d '{"firstName": "John", "lastName": "Doe", "email": "john@gmail.com"}'

# Користувач 2
curl -X POST http://localhost:3001/users \
  -H "Content-Type: application/json" \
  -d '{"firstName": "Jane", "lastName": "Smith", "email": "jane@gmail.com"}'

# Користувач 3
curl -X POST http://localhost:3001/users \
  -H "Content-Type: application/json" \
  -d '{"firstName": "Bob", "lastName": "Johnson", "email": "bob@gmail.com"}'
```

### 6. Очікуваний результат

Після додавання користувачів та переходу на `http://localhost:3001/users`, ви побачите JSON-відповідь з масивом користувачів:

```json
[
  {
    "_id": "64862ee6f0d4ebd8f3c1ec7f",
    "firstName": "Jane",
    "lastName": "Smith",
    "email": "jane@gmail.com",
    "__v": 0
  },
  {
    "_id": "648633d1a300791bf151244e",
    "firstName": "John",
    "lastName": "Smith",
    "email": "john@gmail.com",
    "__v": 0
  }
]
```

## Пояснення конфігурації

### docker-compose.yml

- **frontend** - веб-сервіс на базі Node.js Alpine з `serve` пакетом для статичних файлів
- **api** - веб-сервіс Node.js з Express.js для обробки API запитів (з Dockerfile)
- **api_db** - сервіс MongoDB для збереження даних користувачів (без персистентних volumes)
- **depends_on** - забезпечує правильний порядок запуску сервісів

### Frontend конфігурація (Node.js + serve)

- Використовує **node:alpine** - легкий Node.js образ
- Монтує папку `./frontend` до `/app` в контейнері
- Використовує `npx serve` для обслуговування статичних файлів
- Запускається на порту 8080 (як всередині контейнера, так і ззовні)
- Не потребує окремого Dockerfile - все налаштовується через docker-compose

### API конфігурація (Node.js + Express)

- Будується з окремого Dockerfile в папці `./api`
- Запускається командою `npm start`
- Підключається до MongoDB через змінну середовища `MONGO_URL`

### Змінні середовища

- **PORT=3001** - порт для запуску API сервера
- **HOST=0.0.0.0** - хост для прив'язки (дозволяє підключення з інших контейнерів)
- **MONGO_URL** - URL для підключення до MongoDB
- **MONGO_INITDB_DATABASE=api** - ім'я бази даних для ініціалізації

### Доступ до сервісів

| Сервіс           | URL                         | Опис                                               |
| ---------------- | --------------------------- | -------------------------------------------------- |
| 🌐 **Frontend**  | http://localhost:8080       | Веб-інтерфейс користувача                          |
| 🔗 **API**       | http://localhost:3001       | REST API для роботи з користувачами                |
| 📊 **API Users** | http://localhost:3001/users | Список користувачів (JSON)                         |
| 🗄️ **MongoDB**   | mongodb://localhost:27017   | База даних (доступна лише всередині Docker мережі) |

## Корисні команди

```bash
# Перегляд логів
docker-compose logs api
docker-compose logs api_db

# Перегляд логів в реальному часі
docker-compose logs -f

# Перезапуск окремого сервісу
docker compose restart api

# Видалення всіх контейнерів та образів
docker compose down --rmi all

# Повне очищення (контейнери та образи)
docker compose down --rmi all
```

## Troubleshooting

### Проблема з портом

Якщо порт 3001 зайнятий, змініть його в `docker-compose.yml`:

```yaml
ports:
  - '3002:3001' # Використати порт 3002 замість 3001
```

### Проблема з підключенням до MongoDB

Переконайтеся, що:

1. Сервіс `api_db` запущений: `docker-compose ps`
2. MongoDB контейнер працює: `docker-compose logs api_db`
3. Змінна `MONGO_URL` правильно налаштована

### Очистка системи

```bash
# Видалити всі зупинені контейнери
docker container prune

# Видалити невикористовувані образи
docker image prune

# Видалити невикористовувані томи
docker volume prune
```

# docker-lab-3-HelloDockerRealWorld
