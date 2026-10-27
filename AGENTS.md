# Agents.md

Файл для ИИ-агентов (Claude Code, Cursor, Copilot, ChatGPT), работающих в этом репозитории.
Прочитай его целиком перед любым действием. Если что-то противоречит — приоритет у `docs/01-srs.md` и этого файла.

## 1. Проект

**«Календарик»** — учебный веб-календарь для маленькой компании (пародия на Google Calendar).
Планировщик: события, встречи с участниками, внешние ссылки на созвон, in-app уведомления, RSVP.
**Не** сервис видеозвонков, **не** интеграция с Google/Yandex API.

Дисциплина — методологии (Waterfall). Кода мало, процесс и артефакты важнее объёма фич.

## 2. Стек (не менять без CR)

- **Backend:** C# / .NET 8, ASP.NET Core Web API, EF Core, Npgsql
- **Frontend:** React 18 + TypeScript, Vite, React Router, axios
- **БД:** PostgreSQL 16
- **Auth:** JWT (Bearer) или cookie — решение в `docs/02-design-doc.md`; пароли через `BCrypt.Net`
- **Календарь:** FullCalendar (react-обёртка) через npm
- **Стили:** Bootstrap или минимальный CSS
- **Без:** Docker в проде, микросервисов, GraphQL, Redis, очередей, WebSocket, внешних API

## 3. Структура репозитория
/
├── Agents.md
├── README.md
├── docs/
│ ├── 00-team-charter.md
│ ├── 01-srs.md
│ ├── 02-design-doc.md
│ ├── 03-test-plan.md
│ ├── 04-test-report.md
│ ├── 05-change-log.md
│ ├── 06-deployment.md
│ ├── 07-support.md
│ ├── demo-script.md
│ ├── report.md
│ └── prompts/
│ └── team-lead-agent.md
├── backend/
│ ├── Calendar.sln
│ └── Calendar.Api/
│ ├── Controllers/
│ ├── Models/
│ ├── Data/
│ ├── Services/
│ ├── Program.cs
│ └── Calendar.Api.csproj
├── frontend/
│ ├── src/
│ │ ├── api/
│ │ ├── components/
│ │ ├── pages/
│ │ ├── types/
│ │ └── main.tsx
│ ├── package.json
│ └── vite.config.ts
├── docker-compose.yml
└── .gitignore

## 4. Команды

```bash
# БД
docker compose up -d db

# Backend
cd backend && dotnet restore
dotnet ef database update --project Calendar.Api
dotnet run --project Calendar.Api

# Frontend
cd frontend && npm install && npm run dev

# Тесты backend (опционально)
cd backend && dotnet test
5. Конвенции C#
.NET 8, nullable включён, async/await везде, где I/O.

Контроллеры тонкие, логика — в Services/.

DTO для запросов/ответов, не отдавать EF-сущности наружу.

Валидация через DataAnnotations + ручные проверки в сервисе.

Ошибки — ProblemDetails, коды 400/401/403/404/409.

Имена: PascalCase для public, _camelCase для private полей.

Никаких Console.WriteLine — ILogger<T>.

6. Конвенции TypeScript / React
strict: true, никаких any без причины.

Функциональные компоненты + хуки, без классов.

Типы API — в src/types/, зеркалят DTO backend.

Запросы — через src/api/, не размазывать fetch по компонентам.

Состояние — useState/useReducer; без Redux/MobX.

Стили — CSS-модули или Bootstrap-классы, без styled-components.

7. Правила для ИИ-агентов
Scope фиксирован в docs/01-srs.md. Не добавляй фичи вне MVP. Идея вне scope → CR-кандидат в docs/05-change-log.md, не реализуй без подтверждения.

Сначала читай артефакты. Перед кодом — 01-srs.md и 02-design-doc.md. Противоречие — спроси, не догадывайся.

Не меняй стек и архитектуру без CR: без автомапперов, MediatR, CQRS, микросервисов, GraphQL, Redis.

Один PR — одна задача. Не смешивай фичу, рефакторинг и форматирование.

Артефакты важнее кода. Просят «добавь фичу» → проверь SRS. Нет в SRS → CR.

Code freeze уважай. После объявления — только критичные багфиксы.

Тесты — ручные. Автотестов не требуется; если пишешь — только xUnit для сервисов, не для UI.

Не коммить секреты. Строки подключения — в appsettings.Development.json (в .gitignore), .env для фронта.

Обновляй README и docs, если меняешь запуск или поведение.

8. Definition of Done (DoD)
работает по acceptance criteria из 01-srs.md;

код собирается (dotnet build / npm run build) без ошибок;

ручной тест-кейс из 03-test-plan.md проходит;

изменения отражены в README или docs;

нет TODO в критичном пути.

9. Чего НЕ делать
Микросервисы, CQRS, DDD-слои «на будущее».

Email/push/telegram/WebSocket уведомления.

Google/Yandex Calendar API.

Свой сервис видеозвонков.

Повторяющиеся события, часовые пояса, приватные календари.

Автотесты на UI (Selenium/Playwright).

Менять структуру docs/ без согласования с PM.

10. Ссылки
SRS (ТЗ): docs/01-srs.md — источник истины по фичам

Design Doc: docs/02-design-doc.md — источник истины по архитектуре

Change Log: docs/05-change-log.md — все изменения scope

Test Plan: docs/03-test-plan.md — что и как проверяем

text

---

## `docs/00-team-charter.md`

```markdown
# Team Charter — «Календарик»

## 1. Цель проекта

Собрать MVP «Календарика» (веб-календарь для маленькой компании) за один вечер и показать демо на 5 минут на паре. Дисциплина — методологии (Waterfall), не объём кода.

## 2. Команда и роли

| # | Человек | Роли | Артефакты (владелец) | FR |
|---|---|---|---|---|
| 1 | [Имя] | PM / лид | Team Charter, SRS, Change Log, Demo Script, Report | — (процесс) |
| 2 | [Имя] | Backend + Интегратор | схема БД, `Models/`, `Data/`, `Services/`, `AuthController`, `EventsController`, `NotificationsController` | FR-1…FR-13 (роуты) |
| 3 | [Имя] | Frontend | React-страницы, FullCalendar, формы, карточка, колокольчик | UI для FR-2…FR-11 |
| 4 | [Имя] | QA + документация | Test Plan, Test Report, README, seed-данные | тесты на FR-1…FR-13 |
| 5 | [Имя] | Design Doc owner + ревью + помощь | Design Doc, ревью PR, помощь C и D, второй голос на демо | дизайн FR-3…FR-11 |

**Правила склейки ролей:**
- У каждого артефакта и каждого FR — один владелец, даже если ролей у человека две.
- QA не тестирует свой код — код продукта не пишет.
- PM не кодит.
- Design Doc собирает пятый из решений backend и frontend; PM принимает на gate 2.

## 3. Каналы связи

- Один общий чат в мессенджере — туда пишут все, никаких «мне в личку».
- Все решения фиксируются в `docs/` (Project Book), а не «где-то в чате».
- Личные сообщения — только для личного, не для рабочих решений.

## 4. Definition of Done (DoD)

Задача «готова», если:
- работает по acceptance criteria из `docs/01-srs.md`;
- код собирается (`dotnet build` / `npm run build`) без ошибок;
- ручной тест-кейс из `docs/03-test-plan.md` проходит;
- изменения отражены в README или `docs/`;
- нет `TODO` в критичном пути.

## 5. Правила работы

- Scope меняется **только** через Change Request в `docs/05-change-log.md`.
- После baseline SRS молча ничего не меняется.
- Code freeze — по расписанию; после него только критичные багфиксы.
- Вопросы — в общий чат.
- PM не кодит. QA не тестирует свой код.
- Конфликт двух мнений: 2 минуты на аргументы → решает владелец артефакта → PM фиксирует решение.

## 6. Дедлайны и gate-точки

| Gate | Этап | Артефакт | Критерий |
|---|---|---|---|
| 0 | Инициация | Team Charter | Роли и план согласованы |
| 1 | Требования | SRS v1.0 | Baseline зафиксирован |
| 2 | Проектирование | Design Doc + Test Plan | Дизайн и тест-план одобрены |
| 3 | Реализация | Код, черновик README | Code freeze объявлен |
| 4 | Верификация | Test Report | Acceptance criteria выполнены |
| 5 | Внедрение | Deployment Doc | Инструкция по запуску есть |
| 6 | Поддержка | Support Doc | 2–3 строчки о сопровождении есть |
| 7 | Демо | Demo Script, Report | Репетиция прошла, отчёт готов |

## 7. Разрешение конфликтов

Два мнения → 2 минуты аргументов → решает владелец артефакта → PM фиксирует в Project Book. Без «давайте обсудим ещё час».