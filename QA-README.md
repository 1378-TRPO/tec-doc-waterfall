# QA README — «Календарик»

Практическая инструкция для QA на Gate 3–4. Требования проекта — в `docs/01-srs.md`, сценарии и ожидаемые результаты — в `docs/03-test-plan.md`.

## Роль QA

- подготовить и поддерживать ручной Test Plan;
- подготовить тестовые данные и окружение;
- выполнить приёмочный прогон на Gate 4;
- зафиксировать результаты и дефекты в `docs/04-test-report.md`;
- не вносить изменения в scope MVP без Change Request.

## Тестовые пользователи

| Роль | Email | Пароль |
|---|---|---|
| Организатор | `alice@local` | `alice123` |
| Участник | `bob@local` | `bob123` |
| Второй участник | `carol@local` | `carol123` |

Alice создаёт встречу; Bob принимает приглашение; Carol остаётся `pending` или выбирает `declined`.

## Подготовка среды

Backend находится в соседнем репозитории `waterfall-calendar-backend`.

```powershell
cd ..\waterfall-calendar-backend
docker compose up -d db
Copy-Item Calendar\Calendar.Api\appsettings.Development.example.json Calendar\Calendar.Api\appsettings.Development.json
cd Calendar
dotnet tool restore
dotnet ef database update --project Calendar.Api
dotnet build
dotnet run --project Calendar.Api
```

В `appsettings.Development.json` нужно задать собственный JWT-ключ длиной не менее 32 символов. Файл не коммитится.

После появления frontend в репозитории `waterfall-calendar-frontend` выполнить его инструкции установки и проверить сборку:

```powershell
npm install
npm run build
npm run dev
```

## Как выполнять тесты

1. Открыть `docs/03-test-plan.md` и выбрать тест-кейс.
2. Убедиться, что БД содержит seed-пользователей и есть доступ к JWT Alice, Bob и Carol.
3. API проверять через Swagger или Postman с заголовком `Authorization: Bearer <JWT>`; UI — вручную в браузере.
4. Время передавать в ISO 8601 UTC. Для базового сценария использовать встречу на завтра с 10:00 до 11:00 и ссылкой `https://meet.local/qa-demo`.
5. Сразу фиксировать результат в `docs/04-test-report.md`: `Passed`, `Failed`, `Blocked` или `Not run`.

## Текущие зависимости

| Область | Состояние | Действие QA |
|---|---|---|
| Аутентификация, события, встречи, RSVP | API реализован | Можно начинать API-проверки. |
| Уведомления и пинг | API ещё не реализован | TC-09 и TC-10 отмечать `Blocked`. |
| Календарь, колокольчик, demo-ссылка | Frontend ещё не реализован | UI-части TC-03, TC-06, TC-11, TC-12, TC-13 отмечать `Blocked`. |

`Blocked` означает внешнюю зависимость и не равен дефекту. `Failed` ставится только тогда, когда реализованная функция не соответствует ожидаемому результату.

## Шаблон дефекта

| Поле | Что указать |
|---|---|
| ID | Например, `BUG-01` |
| Версия сборки | Проверяемый commit или дата сборки |
| Тест-кейс | Например, `TC-07` |
| Предусловия | Учётная запись, данные и состояние БД |
| Шаги | Минимальные шаги для воспроизведения |
| Факт / ожидание | Что произошло и что требует SRS |
| Серьёзность | `Critical`, `Major` или `Minor` |
| Статус | `Open`, `Fixed`, `Retest`, `Closed` |

## Условие завершения Gate 4

Все acceptance criteria из SRS имеют результат, Test Report заполнен и готов к передаче PM.
