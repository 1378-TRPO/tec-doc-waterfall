# Design Doc — «Календарик»

**Основание:** `docs/01-srs.md` v1.0

## 1. Архитектура

Классическое трёхзвенное приложение: SPA на React (Vite) общается по REST/JSON через axios с монолитным ASP.NET Core Web API, который хранит данные в PostgreSQL через EF Core (Npgsql). Backend разбит на три контроллера — `AuthController`, `EventsController`, `NotificationsController`, — тонкие контроллеры вызывают сервисы из `Services/`, наружу отдаются только DTO. Уведомления in-app: сервис создаёт запись в таблице `Notifications` при приглашении, пинге, изменении и отмене встречи, а фронтенд опрашивает счётчик непрочитанных (polling раз в 30 с, без WebSocket).

```
React + FullCalendar ──axios (JSON, Bearer JWT)──> ASP.NET Core Web API ──EF Core──> PostgreSQL 16
                                                   Auth / Events / Notifications
```

## 2. Аутентификация: JWT или cookie

**Решение: JWT (Bearer-токен).**

| Критерий | JWT (Bearer) | Cookie |
|---|---|---|
| Фронт и API на разных портах (5173 / 5xxx) | работает с обычным CORS | нужен `AllowCredentials`, `SameSite`, настройка CORS |
| CSRF | не применим | нужна защита |
| Проверка через Swagger / Postman | просто вставить токен | неудобно |
| Состояние на сервере | нет (stateless) | нет, но больше настроек |
| Отзыв токена | нельзя до истечения | можно |

Почему JWT: фронтенд и backend живут на разных портах, и Bearer-токен избавляет от настройки CORS с credentials и защиты от CSRF; его легко проверять в Swagger. Недостаток (нельзя отозвать токен) для учебного MVP некритичен.

Детали:
- `POST /api/auth/login` проверяет пароль через `BCrypt.Net` и возвращает токен; `POST /api/auth/register` создаёт пользователя.
- Токен подписан HS256, срок жизни 8 часов, refresh-токенов нет. В claims: `sub` (UserId), `email`, `name`.
- Ключ подписи — в `appsettings.Development.json` (не коммитится).
- Фронт хранит токен в `localStorage`, axios-интерсептор добавляет `Authorization: Bearer <token>`; на ответ `401` — выход на страницу логина.
- Для демо (FR-1) в БД сидятся пользователи `alice@local`, `bob@local`, `carol@local` с известными паролями.

## 3. Схема БД

Одна таблица для событий и встреч (поле `Type`), участники и уведомления — в отдельных таблицах. Все даты хранятся в UTC (`timestamptz`), часовые пояса вне scope.

```
Users 1 ──< CalendarEvents (OrganizerId)
Users 1 ──< EventParticipants >── 1 CalendarEvents
Users 1 ──< Notifications >── 0..1 CalendarEvents
```

### Users
| Поле | Тип | Ограничения |
|---|---|---|
| Id | int | PK, identity |
| Email | varchar(256) | NOT NULL, UNIQUE |
| PasswordHash | varchar(100) | NOT NULL (BCrypt) |
| DisplayName | varchar(100) | NOT NULL |
| CreatedAt | timestamptz | NOT NULL |

### CalendarEvents
| Поле | Тип | Ограничения |
|---|---|---|
| Id | int | PK, identity |
| Type | varchar(16) | NOT NULL, `event` / `meeting` |
| Title | varchar(200) | NOT NULL, не пустое (FR-5) |
| Description | text | NULL |
| StartAt | timestamptz | NOT NULL |
| EndAt | timestamptz | NOT NULL, CHECK `EndAt > StartAt` (FR-5) |
| MeetingUrl | varchar(500) | NULL; для `meeting` начинается с `http://` или `https://` (FR-5) |
| OrganizerId | int | FK → Users.Id, NOT NULL |
| IsCancelled | bool | NOT NULL, default false (FR-12) |
| CreatedAt / UpdatedAt | timestamptz | NOT NULL |

Индексы: `(OrganizerId, StartAt)`, `(StartAt)`.

### EventParticipants
| Поле | Тип | Ограничения |
|---|---|---|
| EventId | int | PK (составной), FK → CalendarEvents.Id, ON DELETE CASCADE |
| UserId | int | PK (составной), FK → Users.Id |
| RsvpStatus | varchar(16) | NOT NULL, `pending` / `accepted` / `declined`, default `pending` |
| RespondedAt | timestamptz | NULL |

Индекс: `(UserId)` — для календаря участника и «Ближайших встреч».

### Notifications
| Поле | Тип | Ограничения |
|---|---|---|
| Id | int | PK, identity |
| UserId | int | FK → Users.Id, NOT NULL (получатель) |
| EventId | int | FK → CalendarEvents.Id, NULL |
| Type | varchar(16) | NOT NULL, `invited` / `pinged` / `changed` / `cancelled` |
| Message | varchar(300) | NOT NULL |
| IsRead | bool | NOT NULL, default false |
| CreatedAt | timestamptz | NOT NULL |

Индекс: `(UserId, IsRead)` — для счётчика колокольчика.

Енамы (`Type`, `RsvpStatus`) в C# — `enum`, в БД хранятся строкой через `HasConversion<string>()`.

## 4. API

| Метод и путь | Назначение | FR |
|---|---|---|
| `POST /api/auth/register`, `POST /api/auth/login` | регистрация, вход, выдача JWT | FR-1 |
| `GET /api/auth/me` | текущий пользователь | FR-1 |
| `GET /api/users` | список пользователей для выбора участников | FR-4 |
| `GET /api/events?from=&to=` | мои события и встречи (организатор или участник) за период | FR-2 |
| `GET /api/events/{id}` | карточка с участниками и RSVP | FR-7 |
| `POST /api/events` | создать событие / встречу | FR-3, FR-4, FR-5 |
| `PUT /api/events/{id}` | изменить (только организатор) → `changed` участникам | FR-12 |
| `POST /api/events/{id}/cancel` | отменить (только организатор) → `cancelled` участникам | FR-12 |
| `PUT /api/events/{id}/rsvp` | принять / отклонить (только участник) | FR-6 |
| `POST /api/events/{id}/participants/{userId}/ping` | пингнуть участника → `pinged` | FR-8 |
| `GET /api/events/upcoming` | ближайшие встречи по времени | FR-10 |
| `GET /api/notifications`, `GET /api/notifications/unread-count` | список и счётчик | FR-9 |
| `POST /api/notifications/{id}/read` | отметить прочитанным | FR-9 |

Ошибки — `ProblemDetails`: 400 (валидация), 401 (нет токена), 403 (не организатор / не участник), 404, 409 (email занят).

**FR-11** («Сгенерировать демо-ссылку») — на фронте, без запроса к API: `https://meet.local/` + случайная строка из 8 символов подставляется в поле ссылки.

## 5. Frontend

- Страницы (React Router): `/login`, `/register`, `/` (календарь + «Ближайшие встречи»), `/notifications`.
- Календарь — FullCalendar (`dayGridMonth`, `timeGridWeek`); при смене диапазона загружается `GET /api/events?from=&to=`. Отменённые встречи показываются зачёркнутыми, drag-and-drop выключен.
- Компоненты: форма события/встречи (с клиентской валидацией FR-5, дублирующей серверную), карточка встречи (участники, RSVP, кнопки «Пингнуть», «Изменить», «Отменить»), колокольчик в шапке со счётчиком.
- Запросы — в `src/api/` через axios, кеш и повторные запросы — TanStack Query; типы DTO — в `src/types/`.

## 6. Ключевые правила бизнес-логики

- Пользователь видит запись, если он организатор или участник.
- При создании встречи всем участникам ставится `pending` и создаётся уведомление `invited`.
- Изменять, отменять и пинговать может только организатор; отвечать на RSVP — только участник.
- Отменённую встречу нельзя изменить или пингнуть.
