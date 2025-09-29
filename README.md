# Docker Compose з MongoDB - Лабораторна робота 3

## Мета роботи

Створити конфігурацію Docker Compose з двома сервісами: серверний застосунок `api` та сервіс для баз даних `api_db`.

## Передумови

- Встановлений Docker Compose
- Для перевірки виконайте команду: `docker compose --version`

## Архітектура проекту

```
HelloDockerRealWorld/
├── frontend/                 # Frontend сервіс (HTML+CSS+JS)
│   ├── Dockerfile           # Docker конфігурація для Nginx
│   ├── nginx.conf           # Конфігурація Nginx веб-сервера
│   ├── index.html           # Головна сторінка

│   ├── css/
│   │   └── styles.css       # Стилі інтерфейсу
│   └── js/                  # Простий функціональний підхід
│       ├── app.js           # Головний файл застосунку
│       ├── utils.js         # Утилітарні функції
│       └── constants.js     # Константи
├── api/                     # Backend API сервіс (Node.js)
│   ├── Dockerfile           # Docker конфігурація для Node.js
│   ├── package.json         # Залежності Node.js
│   ├── app.js               # Головний файл сервера
│   ├── models/              # Моделі MongoDB
│   │   └── User.js          # Модель користувача
│   └── routes/              # API маршрути
│       └── users.js         # Маршрути для користувачів
├── docker-compose.yml       # Конфігурація всіх сервісів
└── README.md                # Інструкції та документація
```

## Архітектура системи

Наша система складається з **3 основних сервісів**:

1. **Frontend** (порт 8080) - Веб-інтерфейс користувача

   - Nginx веб-сервер для статичних файлів
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

### 1. Практична робота (необов'язково) - Використання Docker Compose

#### Створення простого docker-compose.yml

У кореневій папці проекту створіть файл `docker-compose.yml` з таким вмістом:

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

**Перебудова застосунку (після змін в коді):**

```bash
docker-compose up --build -d
```

**Перегляд працюючих контейнерів:**

```bash
docker ps
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

#### Клонування репозиторію

```bash
cd HelloDockerRealWorld
git clone https://gitlab.com/web-systems-docker/docker-nodejs-app-lab2 .
```

#### Створення docker-compose.yml з трьома сервісами

```yaml
version: '3'
services:
  # Frontend сервіс (Nginx + статичні файли)
  frontend:
    build: ./frontend
    ports:
      - '8080:80'
    depends_on:
      - api
    networks:
      - app-network

  # API сервіс (Node.js + Express)
  api:
    build: ./api
    command: npm start
    ports:
      - '3001:3001'
    environment:
      - PORT=3001
      - HOST=0.0.0.0
      - MONGO_URL=mongodb://api_db:27017/api
    depends_on:
      - api_db
    networks:
      - app-network

  # База даних (MongoDB)
  api_db:
    image: mongo:latest
    environment:
      - MONGO_INITDB_DATABASE=api
    volumes:
      - mongodb_data:/data/db
    networks:
      - app-network

# Мережа для зв'язку між сервісами
networks:
  app-network:
    driver: bridge

# Том для збереження даних MongoDB
volumes:
  mongodb_data:
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

- **frontend** - веб-сервіс на базі Nginx для статичних файлів (HTML/CSS/JS)
- **api** - веб-сервіс Node.js з Express.js для обробки API запитів
- **api_db** - сервіс MongoDB для збереження даних користувачів
- **networks** - створює ізольовану мережу для зв'язку між сервісами
- **volumes** - зберігає дані MongoDB між перезапусками контейнерів
- **depends_on** - забезпечує правильний порядок запуску сервісів

### Dockerfile (Frontend)

- Використовує **nginx:alpine** - легкий веб-сервер
- Копіює статичні файли до `/usr/share/nginx/html/`
- Налаштовує проксування API запитів до backend
- Відкриває порт 80 для HTTP трафіку

### Nginx конфігурація

- Обслуговує статичні файли (HTML, CSS, JS)
- Проксує `/api/` запити до Node.js сервера
- Налаштовує CORS headers для крос-доменних запитів

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
docker-compose restart api

# Видалення всіх контейнерів та образів
docker-compose down --rmi all

# Видалення всіх даних (включно з томами)
docker-compose down -v
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
