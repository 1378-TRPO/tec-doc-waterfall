# Отчёт — «Календарик»

## 1. Общие сведения

- **Продукт:** «Календарик» — веб-календарь для маленькой компании.
- **Цель:** собрать MVP по методологии Waterfall и продемонстрировать работу на паре.
- **Команда:** 5 человек (роли в `docs/00-team-charter.md`).
- **Тимлид / PM:** Виноградов Андрей.
- **Репозиторий:** `tec-doc-waterfall`.

## 2. Этапы Waterfall

| Gate | Этап | Артефакт | Статус |
|---|---|---|---|
| 0 | Инициация | Team Charter | ✅ `docs/00-team-charter.md` |
| 1 | Требования | SRS v1.0 | ✅ `docs/01-srs.md` |
| 2 | Проектирование | Design Doc | ✅ `docs/02-design-doc.md` |
| 3 | Реализация | Код backend + frontend | ✅ `backend/`, `frontend/` |
| 4 | Верификация | Test Plan, Test Report | ✅ `docs/03-test-plan.md`, `docs/04-test-report.md` |
| 5 | Внедрение | Deployment Doc | ✅ `docs/06-deployment.md` |
| 6 | Поддержка | Support Doc | ✅ `docs/07-support.md` |
| 7 | Демо | Demo Script, Report | ✅ `docs/demo-script.md`, `docs/report.md` |

## 3. Scope продукта

MVP включает:
- регистрацию и вход (FR-1);
- календарь в трёх видах: месяц, неделя, день (FR-2);
- события и встречи с участниками (FR-3, FR-4);
- валидацию полей (FR-5);
- RSVP и статусы участников (FR-6, FR-7);
- in-app пинги и уведомления (FR-8, FR-9);
- панель «Ближайшие встречи» (FR-10);
- генератор демо-ссылок (FR-11);
- редактирование/отмену встречи (FR-12);
- хранение данных в PostgreSQL (FR-13).

За scope отвечает SRS v1.0; изменения фиксируются в `docs/05-change-log.md`.

## 4. Архитектура

- **Backend:** C# / .NET 10, ASP.NET Core Web API, EF Core, Npgsql, PostgreSQL 16, BCrypt.Net.
- **Frontend:** React 19, TypeScript, Vite 8, React Router 7, TanStack Query 5, Axios, Tabler Icons.
- **Auth:** JWT Bearer, токен живёт 8 часов.
- **БД:** PostgreSQL 16 в Docker, volume `calendar-db-data`.

Подробное описание архитектуры и решений — в `docs/02-design-doc.md`.

## 5. Результаты реализации

- Реализованы все FR-1…FR-13 в рамках MVP.
- Backend (`waterfall-calendar-backend`): auth, CRUD событий/встреч, RSVP, пинги, уведомления, Swagger UI в dev-режиме.
- Frontend (`waterfall-calendar-frontend`): вход, календарь в трёх видах, формы событий/встреч, карточка с RSVP и пингом, колокольчик уведомлений, панель «Ближайшие», генератор демо-ссылок, переключение тем.
- Код проходит сборку:
  - `dotnet build` — без ошибок;
  - `npm run build` — без ошибок.
- Демо проведено по `docs/demo-script.md` за 5 минут.

## 6. Тестирование

- Ручное функциональное тестирование по `docs/03-test-plan.md`.
- Проверки по технике «чёрный ящик» — через UI.
- Проверки по технике «белый ящик» — просмотр логики валидации, RSVP и уведомлений на backend.
- Результаты зафиксированы в `docs/04-test-report.md`.

## 7. Внедрение и поддержка

- Локальный запуск: `docker compose up -d db`, `dotnet run --project Calendar.Api`, `npm run dev`.
- Подробная инструкция — в `docs/06-deployment.md`.
- Сопровождение: исправление критичных багов через Change Log, резервное копирование БД, обновление seed-данных — `docs/07-support.md`.

## 8. Закрытие Gate 7

- Демо-скрипт актуализирован (`docs/demo-script.md`).
- Демо продукта проведено в рамках 5 минут.
- Все acceptance criteria из SRS v1.0 продемонстрированы.
- Отчёт сдан.

## 9. Выводы

Проект показал, как команда может пройти полный цикл Waterfall: от фиксации требований и проектирования до реализации, тестирования, внедрения и демо готового MVP. Основной упор сделан на процесс, артефакты и согласование scope, а не на объём функциональности.
