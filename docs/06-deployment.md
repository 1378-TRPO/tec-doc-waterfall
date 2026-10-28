# Deployment Doc — «Календарик»

Локальный запуск MVP для разработки и демо.

## Требования

- Windows / Linux / macOS
- Docker Desktop (или Docker Engine)
- .NET SDK 10
- Node.js 18+ и npm

## 1. Подготовка базы данных

Запустить PostgreSQL 16 в Docker:

```bash
docker compose up -d db
```

Контейнер: `calendar-db`  
Порт на хосте: `5433`  
Параметры подключения задаются в `docker-compose.yml`.

## 2. Настройка backend

Создать локальный конфиг `backend/Calendar.Api/appsettings.Development.json` (файл в `.gitignore`):

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Port=5433;Database=calendar;Username=calendar;Password=calendar"
  },
  "Jwt": {
    "Key": "<случайная строка не менее 32 символов>",
    "Issuer": "Calendar",
    "Audience": "Calendar"
  }
}
```

Восстановить зависимости и применить миграции:

```bash
cd backend
dotnet restore
dotnet tool restore
dotnet ef database update --project Calendar.Api
```

Seed-пользователи добавляются в базу автоматически при первом запуске.

## 3. Запуск backend

```bash
cd backend
dotnet run --project Calendar.Api
```

После старта:
- API: http://localhost:5067
- Swagger UI: http://localhost:5067/swagger

## 4. Запуск frontend

```bash
cd frontend
npm install
npm run dev
```

После старта:
- Приложение: http://localhost:5173

## 5. Проверка

1. Открыть http://localhost:5173 — должна загрузиться страница календаря.
2. Войти одним из seed-пользователей:

| Email | Пароль |
|---|---|
| alice@local | alice123 |
| bob@local | bob123 |
| carol@local | carol123 |

3. Проверить, что API через frontend-прокси отвечает: попробовать создать событие или встречу.

## 6. Остановка

Остановить backend и frontend — нажать `Ctrl+C` в соответствующих терминалах.

Остановить базу:

```bash
docker compose down
```

## Известные ограничения

- Запуск только локально, production-конфигурация не описывается в рамках учебного MVP.
- Секреты (`Jwt:Key`, пароль БД) хранятся только в локальном `appsettings.Development.json` и не коммитятся.
- Мобильная вёрстка, email/push-уведомления и интеграции с внешними календарями — вне scope.
