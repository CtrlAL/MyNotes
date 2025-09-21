Отличный вопрос! 👏  
Ты хочешь **задеплоить SonarQube** — либо для локальной разработки, либо для корпоративного использования.  
Я покажу тебе **все основные способы**, от самого простого до production-ready.

---

# 🧩 Что такое SonarQube?

**SonarQube** — это open-source платформа для **непрерывного анализа и инспекции кода**. Она находит:

- 🐛 Баги
- 🚨 Уязвимости
- 💩 Code Smells
- 📊 Покрытие тестами
- 📈 Дублирование кода

> ⚠️ Важно: SonarQube ≠ SonarCloud.  
> - **SonarCloud** — это SaaS (облако, бесплатно для open-source).  
> - **SonarQube** — self-hosted (деплоишь сам, контроль полный).

---

# ✅ Способ 1: Локально через Docker (самый быстрый для старта)

## 🐳 Шаг 1: Запусти SonarQube + PostgreSQL через `docker-compose.yml`

```yaml
# docker-compose.yml
version: '3.8'

services:
  sonarqube:
    image: sonarqube:community
    depends_on:
      - db
    ports:
      - "9000:9000"
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_extensions:/opt/sonarqube/extensions
    networks:
      - sonarnet

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
    volumes:
      - postgresql_data:/var/lib/postgresql/data
    networks:
      - sonarnet

volumes:
  sonarqube_data:
  sonarqube_logs:
  sonarqube_extensions:
  postgresql_data:

networks:
  sonarnet:
    driver: bridge
```

## 🚀 Шаг 2: Запусти

```bash
docker-compose up -d
```

## 🌐 Шаг 3: Открой в браузере

```
http://localhost:9000
```

Логин: `admin`  
Пароль: `admin` (при первом входе попросит сменить)

---

# ✅ Способ 2: На сервере (Production) — через Docker + Nginx + HTTPS

## 📁 Структура

```
/srv/sonarqube/
├── docker-compose.yml
├── data/
├── logs/
├── extensions/
└── nginx/
    └── sonar.conf
```

## 🐳 `docker-compose.yml` (тот же, что выше, но с volume-путями)

```yaml
version: '3.8'

services:
  sonarqube:
    image: sonarqube:community
    depends_on:
      - db
    ports:
      - "127.0.0.1:9000:9000"  # только localhost
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - ./data:/opt/sonarqube/data
      - ./logs:/opt/sonarqube/logs
      - ./extensions:/opt/sonarqube/extensions
    networks:
      - sonarnet
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    networks:
      - sonarnet
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  sonarnet:
    driver: bridge
```

## 🌐 Nginx reverse proxy + HTTPS (через Let’s Encrypt)

### 📄 `/etc/nginx/sites-available/sonar.conf`

```nginx
server {
    server_name sonar.yourcompany.com;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/sonar.yourcompany.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sonar.yourcompany.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = sonar.yourcompany.com) {
        return 301 https://$host$request_uri;
    }
    listen 80;
    server_name sonar.yourcompany.com;
    return 404;
}
```

## 🔄 Получи SSL-сертификат

```bash
sudo certbot --nginx -d sonar.yourcompany.com
```

## 🚀 Перезапусти

```bash
docker-compose up -d
sudo systemctl reload nginx
```

---

# ✅ Способ 3: Kubernetes (Helm Chart)

Если у тебя кластер Kubernetes — используй **официальный Helm Chart**.

## 📥 Добавь репозиторий

```bash
helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
helm repo update
```

## 🚀 Установи

```bash
helm upgrade --install sonarqube sonarqube/sonarqube \
  --set service.type=LoadBalancer \
  --set postgresql.enabled=true \
  --namespace sonarqube \
  --create-namespace
```

> 💡 Для production — настрой PersistentVolume, Ingress, ресурсы, секреты.

---

# ✅ Способ 4: Вручную (без Docker)

## 📥 Скачай

https://www.sonarqube.org/downloads/

## 📁 Распакуй

```bash
tar -xzf sonarqube-*.zip -C /opt
ln -s /opt/sonarqube-*/ /opt/sonarqube
```

## 🐘 Настрой PostgreSQL

```sql
CREATE DATABASE sonar;
CREATE USER sonar WITH ENCRYPTED PASSWORD 'secret';
GRANT ALL PRIVILEGES ON DATABASE sonar TO sonar;
```

## 📄 `/opt/sonarqube/conf/sonar.properties`

```properties
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonar
sonar.jdbc.username=sonar
sonar.jdbc.password=secret

# Для production
sonar.web.host=127.0.0.1
sonar.web.port=9000
```

## 🐱‍👤 Создай пользователя

```bash
sudo useradd -r -s /bin/false sonar
sudo chown -R sonar:sonar /opt/sonarqube
```

## 🔄 Systemd service

```ini
# /etc/systemd/system/sonar.service
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar
Group=sonar
Restart=always

[Install]
WantedBy=multi-user.target
```

## 🚀 Запусти

```bash
sudo systemctl daemon-reload
sudo systemctl start sonar
sudo systemctl enable sonar
```

---

# ✅ Способ 5: Облако (AWS, Azure, GCP)

## ☁️ AWS: через EC2 + RDS

- Запусти EC2 с Ubuntu.
- Установи Docker или вручную.
- Создай RDS PostgreSQL.
- Настрой Security Group (открыть 9000 порт или через Nginx + 443).

## ☁️ Azure: через App Service + Azure Database for PostgreSQL

- Создай Web App for Containers.
- Используй образ `sonarqube:community`.
- Привяжи Azure Database for PostgreSQL.
- Настрой Application Settings (переменные окружения).

## ☁️ GCP: через Compute Engine + Cloud SQL

- ВМ с Ubuntu + Docker.
- Cloud SQL PostgreSQL.
- Настрой фаервол и HTTPS через Load Balancer.

---

# 🧩 Какой способ выбрать?

| Сценарий | Рекомендуемый способ |
|----------|-----------------------|
| 🧪 Локальная разработка, тестирование | ✅ Docker Compose (Способ 1) |
| 🏢 Production on-premise | ✅ Docker + Nginx + HTTPS (Способ 2) |
| ☸️ Kubernetes | ✅ Helm Chart (Способ 3) |
| 🖥️ Старая инфраструктура, без Docker | ✅ Вручную (Способ 4) |
| ☁️ Облако | ✅ EC2/VM + RDS/Cloud SQL (Способ 5) |

---

# ✅ После установки — первые шаги

1. **Зайди в веб-интерфейс** → смени пароль admin.
2. **Создай токен** → `User > My Account > Security > Generate Tokens`.
3. **Создай проект** → или он создастся автоматически при первом `sonar-scanner`.
4. **Настрой Quality Gate** → `Quality Gates > Create`.
5. **Установи плагины** → `Marketplace` (например, C#, Python, etc.).

---

# 🧠 Важные замечания

- ❗ **SonarQube Community Edition не поддерживает Branch Analysis и PR Decoration** — для этого нужна **Developer Edition+** (платная).
- ✅ Для open-source проектов — используй **SonarCloud** (бесплатно, с PR Decoration).
- 🐘 **Обязательно используй PostgreSQL** — H2 (встроенный) только для тестов.
- 📈 Для production — настрой **резервное копирование** (PostgreSQL + volumes SonarQube).

---

# 🚀 Бонус: Бэкап и восстановление

## Бэкап PostgreSQL

```bash
pg_dump -h localhost -U sonar -W sonar > sonar_backup.sql
```

## Бэкап volumes

```bash
tar -czf sonar_data_backup.tar.gz ./data ./logs ./extensions
```

## Восстановление

- Разверни SonarQube.
- Восстанови PostgreSQL: `psql -U sonar -d sonar -f sonar_backup.sql`
- Распакуй volumes.

---

# Конфигурация [[SonarQube]]