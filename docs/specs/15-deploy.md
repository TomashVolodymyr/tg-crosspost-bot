# #15 — Docker-образ, деплой і health-check

Потік D · Блокери: #8

## Мета
Бот запускається на сервері однією командою і сам перезапускається після збою.

## Що зробити
- Багатоетапний `Dockerfile` (збірка → runtime на `node:22-slim`, користувач не root), `prisma migrate deploy` перед стартом.
- `docker-compose.prod.yml`: застосунок і PostgreSQL з томом, `restart: unless-stopped`, змінні з `.env`.
- `GET /health`: база доступна, бот отримав `getMe`, воркер працював за останню хвилину.
- `docs/deploy.md`: сервер, змінні, оновлення, резервна копія бази (`pg_dump` за cron).
- CI: збірка образу на теги `v*`, публікація в GitHub Container Registry.

## Критерії приймання
На чистому сервері з Docker за інструкцією з `docs/deploy.md` бот запускається і публікує пост.
