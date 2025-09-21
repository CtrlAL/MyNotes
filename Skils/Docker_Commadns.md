
# 🐳 Docker Команды

## 🧱 1. Управление образами (Images)

### `docker images`
> Показывает список всех локальных образов.

```bash
docker images
```

📌 **Опции:**
- `-a` — показать все образы (включая промежуточные)
- `-q` — только ID образов
- `--filter "dangling=true"` — показать "висячие" образы (не привязаны к тегам)

---

### `docker pull <image>[:tag]`
> Скачивает образ из Docker Hub (или другого реестра).

```bash
docker pull ubuntu:22.04
docker pull mongo:7.0
```

📌 По умолчанию тег — `latest`.

---

### `docker build -t <name>:<tag> <context>`
> Собирает образ из `Dockerfile`.

```bash
docker build -t myapp:1.0 .
```

📌 `.` — путь к директории с `Dockerfile`.

---

### `docker rmi <image_id или name>`
> Удаляет локальный образ.

```bash
docker rmi ubuntu:22.04
docker rmi $(docker images -q)  # удалить все образы
```

📌 `-f` — принудительное удаление.

---

### `docker tag <image> <new_name>:<tag>`
> Присваивает образу новый тег или имя (например, для публикации).

```bash
docker tag myapp:1.0 myusername/myapp:latest
```

---

### `docker push <image>`
> Отправляет образ в реестр (например, Docker Hub).

```bash
docker push myusername/myapp:latest
```

📌 Предварительно нужно залогиниться: `docker login`.

---

## 🚢 2. Управление контейнерами (Containers)

### `docker ps`
> Показывает запущенные контейнеры.

```bash
docker ps
```

📌 **Опции:**
- `-a` — все контейнеры (включая остановленные)
- `-q` — только ID
- `--format` — кастомный вывод (например, `"{{.Names}}\t{{.Status}}"`)

---

### `docker run [OPTIONS] <image> [COMMAND]`
> Запускает новый контейнер из образа.

```bash
docker run -d --name mynginx -p 8080:80 nginx
```

📌 **Популярные опции:**
- `-d` — запуск в фоне (detached mode)
- `-it` — интерактивный режим с TTY (для shell)
- `--name` — задать имя контейнеру
- `-p HOST:CONTAINER` — проброс портов
- `-v HOST_PATH:CONTAINER_PATH` — монтирование томов
- `--rm` — удалить контейнер после остановки
- `-e KEY=VALUE` — переменные окружения
- `--network` — указать сеть

---

### `docker start <container>`
> Запускает остановленный контейнер.

```bash
docker start mynginx
```

---

### `docker stop <container>`
> Корректно останавливает контейнер (SIGTERM → SIGKILL через 10 сек).

```bash
docker stop mynginx
```

📌 `docker kill <container>` — принудительная остановка (SIGKILL).

---

### `docker restart <container>`
> Перезапускает контейнер.

```bash
docker restart mynginx
```

---

### `docker rm <container>`
> Удаляет остановленный контейнер.

```bash
docker rm mynginx
docker rm $(docker ps -aq)  # удалить все контейнеры
```

📌 `-f` — удалить даже запущенный контейнер.

---

### `docker exec -it <container> <command>`
> Выполняет команду внутри запущенного контейнера.

```bash
docker exec -it mynginx bash
docker exec mynginx ls /app
```

📌 `-it` — для интерактивного доступа (например, к shell).

---

### `docker logs <container>`
> Показывает логи контейнера.

```bash
docker logs mynginx
docker logs -f mynginx          # следить в реальном времени
docker logs --tail 100 mynginx  # последние 100 строк
```

---

### `docker inspect <container|image>`
> Показывает подробную информацию в JSON.

```bash
docker inspect mynginx
docker inspect --format='{{.NetworkSettings.IPAddress}}' mynginx
```

📌 Полезно для отладки и получения IP, путей, переменных и т.д.

---

### `docker cp <container>:<path> <host_path>`
> Копирует файлы между контейнером и хостом.

```bash
docker cp mynginx:/etc/nginx/nginx.conf ./nginx.conf
docker cp ./app.py mypython:/app/
```

---

## 📦 3. Управление томами (Volumes)

### `docker volume ls`
> Список томов.

```bash
docker volume ls
```

---

### `docker volume create <name>`
> Создает новый том.

```bash
docker volume create myapp_data
```

---

### `docker volume rm <name>`
> Удаляет том.

```bash
docker volume rm myapp_data
```

📌 ⚠️ Удаление тома = потеря данных!

---

### `docker volume inspect <name>`
> Детали тома.

```bash
docker volume inspect myapp_data
```

---

## 🌐 4. Управление сетями (Networks)

### `docker network ls`
> Список сетей.

```bash
docker network ls
```

---

### `docker network create <name>`
> Создает пользовательскую сеть.

```bash
docker network create mynet
```

📌 Контейнеры в одной сети могут общаться по имени.

---

### `docker network connect <network> <container>`
> Подключает контейнер к сети.

```bash
docker network connect mynet myapp
```

---

### `docker network disconnect <network> <container>`
> Отключает контейнер от сети.

```bash
docker network disconnect mynet myapp
```

---

### `docker network rm <name>`
> Удаляет сеть.

```bash
docker network rm mynet
```

---

## 🧹 5. Очистка системы

### `docker system prune`
> Удаляет:
- все остановленные контейнеры
- все неиспользуемые сети
- dangling-образы
- build cache

```bash
docker system prune
```

📌 `-a` — также удалить **все неиспользуемые образы** (не только dangling).

---

### `docker image prune`
> Удаляет "висячие" образы.

```bash
docker image prune
```

---

### `docker container prune`
> Удаляет все остановленные контейнеры.

```bash
docker container prune
```

---

### `docker volume prune`
> Удаляет неиспользуемые тома.

```bash
docker volume prune
```

---

## 🧩 6. Docker Compose (если установлен)

### `docker-compose up`
> Запускает сервисы из `docker-compose.yml`.

```bash
docker-compose up -d  # в фоне
```

---

### `docker-compose down`
> Останавливает и удаляет контейнеры, сети, тома (если не external).

```bash
docker-compose down -v  # также удалить тома
```

---

### `docker-compose ps`
> Показывает статус сервисов.

---

### `docker-compose logs [service]`
> Показывает логи сервиса.

```bash
docker-compose logs -f web
```

---

### `docker-compose exec <service> <command>`
> Выполняет команду в контейнере сервиса.

```bash
docker-compose exec web bash
```

---

### `docker-compose build`
> Пересобирает образы сервисов.

```bash
docker-compose build --no-cache
```

---

## 🧪 7. Полезные комбинации и трюки

### Удалить все остановленные контейнеры:
```bash
docker rm $(docker ps -aq)
```

### Удалить все неиспользуемые образы:
```bash
docker rmi $(docker images -q)
```

### Удалить все "висячие" образы:
```bash
docker image prune
```

### Посмотреть использование диска Docker:
```bash
docker system df
```

### Подключиться к контейнеру и сразу в shell:
```bash
docker exec -it <container> /bin/bash
# или
docker exec -it <container> sh
```

### Перезапустить Docker на Linux:
```bash
sudo systemctl restart docker
```

---

## 📋 Шпаргалка — самые частые команды

| Действие | Команда |
|----------|---------|
| Запустить контейнер | `docker run -d --name имя -p хост:конт образ` |
| Посмотреть контейнеры | `docker ps -a` |
| Войти внутрь | `docker exec -it имя bash` |
| Посмотреть логи | `docker logs -f имя` |
| Остановить | `docker stop имя` |
| Удалить контейнер | `docker rm имя` |
| Удалить образ | `docker rmi образ` |
| Собрать образ | `docker build -t имя:тег .` |
| Очистить систему | `docker system prune -a` |

---

## 📚 Дополнительно

- 📘 Официальная документация: https://docs.docker.com/engine/reference/commandline/cli/
- 🎯 Интерактивный туториал: https://www.docker.com/101-tutorial

---

Если нужно — могу подготовить **PDF-шпаргалку**, **alias-ы для bash/zsh**, или **cheat sheet в картинке** 🖼️.

Готов(а) ответить на любые вопросы по Docker — просто спросите! 🐳💪