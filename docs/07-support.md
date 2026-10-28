# Support Doc — «Календарик»

## Сопровождение

- Исправления после релиза оформляются только через Change Request в `docs/05-change-log.md`.
- Критичные баги (невозможность войти, падение API, потеря данных) правит backend-разработчик; мелкие UI-проблемы — frontend-разработчик.
- Логи backend смотрятся в консоли процесса `dotnet run`; ошибки фронтенда — в DevTools браузера.

## Резервное копирование

- Данные хранятся в Docker-volume `calendar-db-data`. Для бэкапа: `docker exec calendar-db pg_dump -U calendar calendar > backup.sql`.
- Восстановление: `docker exec -i calendar-db psql -U calendar calendar < backup.sql`.

## Обновления

- Seed-пользователей и первичные данные можно менять через миграции EF Core или отдельный seed-скрипт.
- Перед обновлением backend/frontend выполнять `dotnet build` и `npm run build`; после — ручной smoke-test по `docs/03-test-plan.md`.

## Ограничения и известные проблемы

- Уведомления работают через опрос раз в 30 секунд; мгновенная доставка не поддерживается.
- Вне scope MVP: email/push-уведомления, интеграции с Google/Yandex Calendar, повторяющиеся события, часовые пояса, мобильная вёрстка.

## Контакты

- PM / тимлид: Виноградов Андрей
