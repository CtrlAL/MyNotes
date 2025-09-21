
# 🚀 Пошаговая инструкция: как запустить и настроить Kafka в Docker на Ubuntu Server (с сетевым мостом)

---
## ✅ ШАГ: 0

```bash
sudo groupadd docker 2>/dev/null || true  # на случай, если группа уже есть
sudo usermod -aG docker $USER
newgrp docker                             # активировать без перелогина
docker ps                                 # проверить
```

---

## ✅ ШАГ 1: Убедись, что сетевой мост включён

### VirtualBox:
- Выключи ВМ.
- `Настройки → Сеть → Тип подключения: Сетевой мост`.
- Включи ВМ.

### VMware:
- `VM → Settings → Network Adapter → Bridged`.

---

## ✅ ШАГ 2: Узнай IP-адрес виртуальной машины

В терминале Ubuntu Server выполни:

```bash
hostname -I
```

Пример вывода:
```
192.168.1.100
```

> 💡 Запомни этот IP — он тебе понадобится везде!

---

## ✅ ШАГ 3: Установи Docker (если ещё не установлен)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker  # чтобы не перезаходить
```

---

## ✅ ШАГ 4: Запусти Kafka в Docker

Скопируй и выполни эту команду — **только замени `192.168.1.100` на IP своей ВМ**:

```bash
docker run -d \
  --name kafka-broker \
  --hostname kafka-broker \
  -p 9092:9092 \
  -p 9093:9093 \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.100:9092 \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@192.168.1.100:9093 \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1 \
  -e KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0 \
  -e KAFKA_NUM_PARTITIONS=3 \
  apache/kafka:latest
```

> ❗ **Обязательно замени `192.168.1.100` на свой IP!**


Если ты тестируешь Kafka **на своей машине (локально)** — замени `192.168.1.100` на `localhost`.

### 🔧 Пример команды:

```bash
docker run -d \
  --name kafka-broker \
  --hostname kafka-broker \
  -p 9092:9092 \
  -p 9093:9093 \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9093 \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1 \
  -e KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0 \
  -e KAFKA_NUM_PARTITIONS=3 \
  apache/kafka:latest
```



> ✅ Теперь:
> 
> - Kafka слушает внутри контейнера на всех интерфейсах (`:9092`)
> - Клиентам говорит: «Подключайтесь к `localhost:9092`»
> - Ты с хоста можешь стучаться на `localhost:9092` — и это будет работать


---

## ✅ ШАГ 5: Проверь, что Kafka запущен

```bash
docker ps
```

→ Должен быть контейнер `kafka-broker` со статусом `Up`.

Проверь логи (если что-то не так):

```bash
docker logs kafka-broker
```

---

## ✅ ШАГ 6: Открой порт в фаерволе (если включён)

```bash
sudo ufw allow 9092
sudo ufw allow 9093
sudo ufw reload
```

> Если `ufw` не установлен или неактивен — пропусти.

---

## ✅ ШАГ 7: Создай тестовый топик

Подключись внутрь контейнера и создай топик:

```bash
docker exec -it kafka-broker /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --create --topic test-topic \
  --partitions 1 --replication-factor 1
```

Проверь список топиков:

```bash
docker exec -it kafka-broker /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 --list
```

→ Должен вывести: `test-topic`

---

## ✅ ШАГ 8: Проверь доступность с Windows

Открой **PowerShell** на Windows и выполни:

```powershell
Test-NetConnection 192.168.1.100 -Port 9092
```

→ Должно быть: `TcpTestSucceeded: True`

---

## ✅ ШАГ 9: Подключись из приложения (C#, Python и т.д.)

### Пример на C# (.NET):

```csharp
using Confluent.Kafka;

var config = new ProducerConfig
{
    BootstrapServers = "192.168.1.100:9092"
};

using var producer = new ProducerBuilder<Null, string>(config).Build();
await producer.ProduceAsync("test-topic", new Message<Null, string> { Value = "Hello from Windows!" });
Console.WriteLine("✅ Сообщение отправлено в Kafka!");
```

---

## ✅ ШАГ 10 (опционально): Тест через консольные утилиты

### Запусти продюсер:

```bash
docker exec -it kafka-broker /opt/kafka/bin/kafka-console-producer.sh \
  --topic test-topic --bootstrap-server localhost:9092
```

Вводи сообщения:
```
Привет из консоли!
Это тест Kafka
```

(Нажми `Ctrl+C`, чтобы выйти.)

### Запусти консьюмер:

```bash
docker exec -it kafka-broker /opt/kafka/bin/kafka-console-consumer.sh \
  --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

→ Должен увидеть свои сообщения.

---

## 💡 Советы на будущее:

- Сохрани эту команду запуска — она тебе ещё пригодится.
- Если IP ВМ изменится (например, после перезагрузки) — пересоздай контейнер с новым IP.
- Для постоянного IP настрой **статический IP** на ВМ или в роутере (DHCP резервирование).
- Хочешь GUI? Поставь **Kafdrop** или **Offset Explorer** — удобно смотреть топики и сообщения.

---

## 🧩 Бонус: `docker-compose.yml` (если хочешь)

Создай файл:

```bash
nano docker-compose.yml
```

Вставь:

```yaml
version: '3.8'
services:
  kafka:
    image: apache/kafka:latest
    container_name: kafka-broker
    hostname: kafka-broker
    ports:
      - "9092:9092"
      - "9093:9093"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.100:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@192.168.1.100:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_NUM_PARTITIONS: 3
    volumes:
      - ./kafka-data:/tmp/kafka-logs
```

Запусти:

```bash
docker-compose up -d
```

---
# [Документация Kafka] [https://kafka.apache.org/documentation/]
