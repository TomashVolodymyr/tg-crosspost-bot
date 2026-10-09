# #2 — Схема бази і перша міграція

Потік A · Блокери: #1

## Мета
Таблиці для напрямків, джерел, маршрутів, правил, постів і доставок.

## Що зробити
- Перенести схему з [data-model.md](../data-model.md) у `prisma/schema.prisma`; розбіжності, що виникнуть, відобразити в data-model.md у тому самому PR.
- `prisma migrate dev --name init`.
- Seed (`prisma/seed.ts`) не потрібен: дані з'являються з бота.

## Критерії приймання
- `npx prisma migrate dev` на чистій базі створює всі таблиці та індекси.
- Унікальні обмеження з data-model.md є в міграції (`Destination(kind, externalId)`, `Post(routeId, sourceKey)`, `Delivery(postId, destinationId)`).

## Тести
Інтеграційний тест на тестовій базі: дві `Delivery` з однаковими `postId` і `destinationId` — помилка унікальності; два ручні пости з `routeId = NULL` — дозволено.
