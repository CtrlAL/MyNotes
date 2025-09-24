Отлично! Вот **полный Quick Start Guide** для запуска Keycloak на виртуальной машине и подключения к нему .NET API с аутентификацией по **Access Token**.

---

## 🚀 Quick Start: Keycloak + .NET API (локально на VM)

---

### 🔹 Шаг 1: Установи Docker на виртуальную машину

Если ещё не установлен:

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER  # чтобы не использовать sudo
```

> Выйди и зайди заново в сессию, чтобы группа `docker` применилась.

---

### 🔹 Шаг 2: Запусти Keycloak в Docker

Выполни в терминале VM:

```bash
docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:24.0 start-dev
```

> ⚠️ `start-dev` — только для разработки! Не для продакшена.

Теперь Keycloak доступен по адресу:  
👉 **http://[IP_VM]:8080**

Например, если IP твоей VM — `192.168.1.100`, то:  
**http://192.168.1.100:8080**

---

### 🔹 Шаг 3: Настрой Keycloak (делай один раз)

1. Открой в браузере: `http://[IP_VM]:8080`
2. Войди как:
   - Логин: `admin`
   - Пароль: `admin`

3. Создай Realm:
   - Нажми **Add realm**
   - Имя: `dev` → **Create**

4. Создай клиент для API:
   - Слева: **Clients** → **Create client**
   - Client type: **OpenID Connect**
   - Client ID: `my-api`
   - Нажми **Next** → **Next** → **Save**

5. Настрой клиента `my-api`:
   - Вкладка **Settings**:
     - **Client authentication**: ✅ (включить!)
     - **Service accounts enabled**: ✅ (если нужно machine-to-machine)
   - Вкладка **Credentials** → скопируй **Secret** (он понадобится для получения токена)

6. Создай тестового пользователя:
   - Слева: **Users** → **Add user**
   - Username: `testuser`
   - Включи **Email verified** → **Save**
   - Перейди во вкладку **Credentials** → задай пароль (например, `password123`)

---

### 🔹 Шаг 4: Получи Access Token (для теста)

Выполни на VM или с любого компьютера в той же сети:

```bash
curl -X POST 'http://[IP_VM]:8080/realms/dev/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'client_id=my-api' \
  -d 'client_secret=СЮДА_ВСТАВЬ_SECRET' \
  -d 'username=testuser' \
  -d 'password=password123' \
  -d 'grant_type=password'
```

> В ответе будет JSON с `access_token`. Сохрани его — он нужен для вызова API.

---

### 🔹 Шаг 5: Создай .NET API

На твоей **локальной машине** (или на той же VM — как удобно):

```bash
dotnet new webapi -n MySecureApi
cd MySecureApi
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

---

### 🔹 Шаг 6: Настрой `Program.cs`

Замени всё в `Program.cs` на:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Аутентификация через JWT (Access Token от Keycloak)
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        // Укажи IP твоей VM и realm
        options.Authority = "http://[IP_VM]:8080/realms/dev";
        options.Audience = "my-api"; // ← должен совпадать с client_id
        options.RequireHttpsMetadata = false; // ← только потому что HTTP (в продакшене — убрать!)
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

// Защищённый эндпоинт
app.MapGet("/api/secure", () =>
{
    var name = app.User.Identity?.Name;
    var email = app.User.FindFirst("email")?.Value;
    return Results.Ok(new { message = "Hello, authenticated user!", name, email });
})
.RequireAuthorization();

app.Run();
```

> 🔴 **Замени `[IP_VM]` на реальный IP твоей виртуальной машины!**  
> Например: `http://192.168.1.100:8080/realms/dev`

---

### 🔹 Шаг 7: Запусти .NET API

```bash
dotnet run
```

По умолчанию запустится на:
- `http://localhost:5000`
- `https://localhost:5001`

---

### 🔹 Шаг 8: Проверь работу

1. Получи `access_token` (см. Шаг 4).
2. Выполни запрос к API:

```bash
curl -H "Authorization: Bearer ВСТАВЬ_СЮДА_ACCESS_TOKEN" http://localhost:5000/api/secure
```

✅ Если всё настроено верно — получишь JSON с данными пользователя.

❌ Если ошибка — проверь:
- Доступность Keycloak с машины, где запущен .NET
- Правильность `Authority` и `Audience`
- Что токен не просрочен

---

## 🧼 Чистка (если нужно)

Остановить Keycloak:

```bash
docker stop keycloak
docker rm keycloak
```
