Конечно! Ниже — обновлённая и расширенная версия руководства с **новой секцией про Docker**, чтобы у тебя был выбор: либо классическая установка Redis, либо быстрый запуск через контейнер.

---

# 🚀 Quick Start: Установка и запуск Redis на VM (NAT) для доступа из Windows  
*(с поддержкой Docker-образа)*

---

## ✅ Шаг 1: Выбери способ установки Redis

У тебя есть **два варианта** — выбери тот, что удобнее:

---

## 📦 Вариант A: Классическая установка (через `apt`)

Выполни в терминале Ubuntu VM:

```bash
sudo apt update
sudo apt install -y redis-server
```

> 💡 Если `redis-server` не найден — включи universe-репозиторий:
> ```bash
> sudo add-apt-repository universe
> sudo apt update
> sudo apt install redis-server
> ```

---

## 🐳 Вариант B: Запуск Redis через Docker (рекомендуется для быстрого старта)

Если ты хочешь **изолированную, чистую и легко управляемую среду** — используй Docker.

### 🔧 Установи Docker (если ещё не установлен):

```bash
# Установка Docker Engine
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### ✅ Добавь себя в группу `docker` (чтобы не использовать `sudo`):

```bash
sudo usermod -aG docker $USER
```

> ⚠️ **Выйди и зайди заново в сессию Ubuntu (или перезапусти VM)**, чтобы изменения вступили в силу.

### 🚀 Запусти Redis в контейнере:

```bash
docker run -d \
  --name redis-server \
  -p 6379:6379 \
  --restart unless-stopped \
  redis:latest \
  redis-server --bind 0.0.0.0 --protected-mode no
```

> 💡 Объяснение:
> - `-d` — запуск в фоне.
> - `-p 6379:6379` — пробрасывает порт контейнера на хост (VM).
> - `--bind 0.0.0.0` — разрешает внешние подключения.
> - `--protected-mode no` — отключает защиту (для теста; в продакшене используй пароль).
> - `--restart unless-stopped` — автоматический перезапуск после сбоя или перезагрузки VM.

### ✅ Проверь, что контейнер работает:

```bash
docker ps
```

→ Должен быть контейнер `redis-server` в статусе `Up`.

Проверь локально внутри VM:

```bash
redis-cli -h 127.0.0.1 ping
# → PONG
```

> 💡 Если `redis-cli` не установлен — установи: `sudo apt install redis-tools`

---

## ✅ Шаг 2: Настрой проброс портов в VirtualBox (или VMware)

Ты в режиме **NAT** → значит, хост не может просто так подключиться к ВМ.

### В VirtualBox:

1. Выключи ВМ (не выключай ОС — останови через VirtualBox).
2. Открой → **Настройки → Сеть → NAT → Дополнительно → Проброс портов**.
3. Добавь правило:

| Имя   | Протокол | Порт хоста | Порт гостя | IP гостя        |
| ----- | -------- | ---------- | ---------- | --------------- |
| Redis | TCP      | `6379`     | `6379`     | (оставь пустым) |

> 💡 IP гостя можно не указывать — если в ВМ только один сетевой интерфейс.

4. Запусти ВМ.

---

## ✅ Шаг 3: Проверь доступ к Redis с хоста (Windows)

Открой PowerShell на **хосте (Windows)**:

```powershell
Test-NetConnection localhost -Port 6379
```

→ Должно быть:

```
TcpTestSucceeded : True
```

> 💡 Почему `localhost`? Потому что ты пробросил порт **с хоста 6379 → в гостя 6379**.

Если `False` — проверь:
- Проброс портов в VirtualBox
- Если используешь Docker — убедись, что контейнер запущен (`docker ps`) и пробрасывает порт (`-p 6379:6379`)
- Если используешь классический Redis — проверь `bind 0.0.0.0` в `/etc/redis/redis.conf`
- Фаервол внутри ВМ: `sudo ufw allow 6379` (если включён)

---

## ✅ Шаг 4: Подключись из C# (.NET)

Создай проект:

```bash
dotnet new console -n RedisTest
cd RedisTest
dotnet add package StackExchange.Redis
```

Замени `Program.cs`:

```csharp
using StackExchange.Redis;
using System;

class Program
{
    static async Task Main(string[] args)
    {
        // Подключаемся к localhost:6379 — потому что порт проброшен!
        var redis = await ConnectionMultiplexer.ConnectAsync("localhost:6379");

        var db = redis.GetDatabase();

        await db.StringSetAsync("test", "Hello from Windows to VM!");
        var value = await db.StringGetAsync("test");

        Console.WriteLine($"Получено: {value}");
    }
}
```

Запусти:

```bash
dotnet run
```

→ Вывод:

```
Получено: Hello from Windows to VM!
```

---
## [Документация Redis] [https://stackexchange.github.io/StackExchange.Redis/]