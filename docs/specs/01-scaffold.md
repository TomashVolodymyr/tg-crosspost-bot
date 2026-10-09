# #1 — Каркас NestJS, Docker Compose, lint, тести, CI

Потік A · Блокери: —

## Мета
Порожній, але робочий застосунок: запускається локально, перевіряє конфіг, підключається до PostgreSQL, проходить lint, тести і збірку в CI.

## Що зробити
- `nest new` у корінь репозиторію, TypeScript у strict-режимі, Node.js 22 (`.nvmrc`, `engines` у `package.json`).
- `ConfigModule` з валідацією змінних з [configuration.md](../configuration.md): обов'язкові `DATABASE_URL`, `BOT_TOKEN`, `ADMIN_IDS`; `ADMIN_IDS` розбирається в масив чисел. Помилка валідації — зрозуміле повідомлення і вихід з кодом 1.
- Prisma: `prisma init`, `PrismaService` з `onModuleInit` / `enableShutdownHooks`.
- `docker-compose.yml` з PostgreSQL 16 для розробки (том для даних, порт 5432).
- Lint (oxlint або ESLint), Prettier, Vitest; скрипти `lint`, `test`, `build`, `start:dev`.
- `.github/workflows/ci.yml`: на push і pull request — `npm ci`, `lint`, `test`, `build`.
- Оновити розділ «Local setup» у README.

## Поза межами
Схема бази (#2), будь-яка логіка бота (#3).

## Критерії приймання
- `docker compose up -d` і `npm run start:dev` запускають застосунок без помилок.
- Без `BOT_TOKEN` застосунок не стартує і пише, якої змінної бракує.
- CI зелений на першому PR.

## Тести
Юніт-тест валідації конфігу: валідний набір проходить, без `ADMIN_IDS` і з `ADMIN_IDS=abc` — помилка.
