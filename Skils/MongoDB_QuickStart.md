Конечно! Вот **быстрый старт с MongoDB**, включая запуск через **Docker** — самый простой и популярный способ для разработки.

---

## 🐳 MongoDB Quick Start через Docker (рекомендуемый способ)

### ✅ Шаг 1: Установите Docker

Если Docker ещё не установлен — скачайте и установите:

- [Docker Desktop (Windows/macOS)](https://www.docker.com/products/docker-desktop)
- Linux: `sudo apt install docker.io docker-compose` (для Ubuntu/Debian)

Проверьте установку:

```bash
docker --version
docker-compose --version
```

---

### ✅ Шаг 2: Запустите MongoDB одной командой

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v mongodb_data:/data/db \
  mongo:latest
```

> 💡 Используется официальный образ `mongo:7.0` (актуальная стабильная версия на 2025).  
> `-v mongodb_data:/data/db` — сохраняет данные даже после перезапуска контейнера.  
> `-p 27017:27017` — пробрасывает порт MongoDB на хост.

---

### ✅ Шаг 3: Подключитесь к MongoDB

#### Вариант A: Через `mongosh` (MongoDB Shell) внутри контейнера

```bash
docker exec -it mongodb mongosh
```

Вы окажетесь в интерактивной оболочке MongoDB:

```js
> show dbs
> use mydb
> db.mycollection.insertOne({ name: "Alice", age: 30 })
> db.mycollection.find()
```

#### Вариант B: Установите `mongosh` локально и подключитесь с хоста

Скачайте `mongosh`:

👉 https://www.mongodb.com/try/download/shell

После установки:

```bash
mongosh "mongodb://localhost:27017"
```

---

## 🗃️ (Опционально) Запуск с Docker Compose

Создайте файл `docker-compose.yml`:

```yaml
version: '3.8'
services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    restart: unless-stopped

volumes:
  mongodb_data:
```

Запустите:

```bash
docker-compose up -d
```

Остановить:

```bash
docker-compose down
```

> 💡 Данные сохраняются в volume `mongodb_data`, их можно посмотреть:  
> `docker volume inspect mongodb_data`

---

## 🔐 (Опционально) Запуск с аутентификацией

Создайте пользователя при первом запуске:

```bash
docker run -d \
  --name mongodb-auth \
  -p 27017:27017 \
  -v mongodb_auth_data:/data/db \
  mongo:4.4 \
  --auth
```

Затем создайте администратора:

```bash
docker exec -it mongodb-auth mongosh admin
```

Внутри `mongosh`:

```js
db.createUser({
  user: "admin",
  pwd: "123",  // ← замените на свой пароль
  roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
})
```

Теперь подключайтесь с аутентификацией:

```bash
mongosh "mongodb://admin:password123@localhost:27017/admin?authSource=admin"
```

---

## 🧪 GUI-клиенты (по желанию)

Для удобной работы с MongoDB используйте:

- **MongoDB Compass** (официальный GUI) — https://www.mongodb.com/try/download/compass
- **Studio 3T** — мощный коммерческий клиент
- **VS Code + MongoDB Extension**

---

## 🧹 Очистка (если нужно)

Остановить и удалить контейнер:

```bash
docker stop mongodb
docker rm mongodb
```

Удалить volume (все данные!):

```bash
docker volume rm mongodb_data
```

---

## 📌 Полезные команды

| Команда                               | Описание                       |
| ------------------------------------- | ------------------------------ |
| `docker ps`                           | Показать работающие контейнеры |
| `docker logs mongodb`                 | Посмотреть логи MongoDB        |
| `docker exec -it mongodb mongosh`     | Зайти в оболочку MongoDB       |
| `mongosh "mongodb://localhost:27017"` | Подключиться с хоста           |

---