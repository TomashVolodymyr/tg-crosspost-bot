# Модель даних

Чернетка схеми Prisma. Остаточна схема з'являється в `prisma/schema.prisma` у тікеті #2; цей документ оновлюється разом з нею.

## Сутності

| Модель | Що зберігає |
| --- | --- |
| `Destination` | Куди публікуємо: Telegram-чат (група чи канал) або Facebook-сторінка. |
| `Source` | Звідки переносимо: канал, у якому бот адміністратор, або канал, який читає сесія MTProto. |
| `Route` | Зв'язок «джерело → напрямок» з набором правил. |
| `RewriteRule` | Одне правило переписування маршруту. |
| `Post` | Одна публікація: вміст після правил і походження. |
| `Delivery` | Доставка поста в один напрямок: статус, спроби, зовнішні id. |

## Схема

```prisma
enum DestinationKind {
  TELEGRAM_CHAT
  FACEBOOK_PAGE
}

enum SourceKind {
  BOT_CHANNEL   // бот — адміністратор каналу-джерела
  MTPROTO       // канал читає сесія користувача
}

enum PostOrigin {
  MANUAL
  RELAY
}

enum DeliveryStatus {
  PENDING
  SENT
  FAILED
  SKIPPED
}

enum RewriteRuleType {
  REPLACE_PHONE      // будь-який номер телефону -> replacement
  REPLACE_USERNAME   // @username -> replacement
  REPLACE_LINK       // посилання, що збігаються з pattern -> replacement
  REPLACE_TEXT       // довільний регулярний вираз pattern -> replacement
  APPEND_FOOTER      // дописати replacement в кінець
  SKIP_IF_CONTAINS   // не публікувати пост, якщо є pattern
}

model Destination {
  id         Int             @id @default(autoincrement())
  kind       DestinationKind
  externalId String          // chat_id Telegram або page_id Facebook
  title      String
  enabled    Boolean         @default(true)
  disabledReason String?
  createdAt  DateTime        @default(now())

  routes     Route[]
  deliveries Delivery[]

  @@unique([kind, externalId])
}

model Source {
  id         Int        @id @default(autoincrement())
  kind       SourceKind
  chatId     String     // id каналу в Telegram (-100...)
  username   String?
  title      String
  enabled    Boolean    @default(true)
  createdAt  DateTime   @default(now())

  routes     Route[]
  posts      Post[]

  @@unique([kind, chatId])
}

model Route {
  id            Int           @id @default(autoincrement())
  sourceId      Int
  destinationId Int
  enabled       Boolean       @default(true)
  createdAt     DateTime      @default(now())

  source        Source        @relation(fields: [sourceId], references: [id], onDelete: Cascade)
  destination   Destination   @relation(fields: [destinationId], references: [id], onDelete: Cascade)
  rules         RewriteRule[]
  posts         Post[]

  @@unique([sourceId, destinationId])
}

model RewriteRule {
  id          Int             @id @default(autoincrement())
  routeId     Int
  position    Int             // порядок застосування
  type        RewriteRuleType
  pattern     String?
  replacement String?

  route       Route           @relation(fields: [routeId], references: [id], onDelete: Cascade)

  @@index([routeId, position])
}

model Post {
  id               Int        @id @default(autoincrement())
  origin           PostOrigin
  sourceId         Int?
  routeId          Int?
  sourceKey        String?    // media_group_id або message_id поста в джерелі
  sourceMessageIds Int[]      // усі повідомлення джерела, з яких складено пост
  payload          Json       // PostDraft після правил, без буферів медіа
  createdById      BigInt?    // Telegram id адміна для MANUAL
  createdAt        DateTime   @default(now())

  source           Source?    @relation(fields: [sourceId], references: [id], onDelete: SetNull)
  route            Route?     @relation(fields: [routeId], references: [id], onDelete: SetNull)
  deliveries       Delivery[]

  @@unique([routeId, sourceKey])   // захист від дубля при повторному оновленні
}

model Delivery {
  id            Int            @id @default(autoincrement())
  postId        Int
  destinationId Int
  status        DeliveryStatus @default(PENDING)
  attempts      Int            @default(0)
  nextAttemptAt DateTime       @default(now())
  externalIds   String[]       // message_id у Telegram або post_id у Facebook
  lastError     String?
  sentAt        DateTime?
  createdAt     DateTime       @default(now())

  post          Post           @relation(fields: [postId], references: [id], onDelete: Cascade)
  destination   Destination    @relation(fields: [destinationId], references: [id], onDelete: Cascade)

  @@unique([postId, destinationId])
  @@index([status, nextAttemptAt])
}
```

## Рішення

- **Медіа не зберігаються в базі.** У `payload` лишаються `file_id` Telegram; буфери, завантажені через MTProto, живуть лише до відправки. Якщо доставка з таким медіа впала і повторюється після перезапуску, медіа завантажується з джерела ще раз за `sourceMessageIds`.
- **`@@unique([routeId, sourceKey])`** не дає одному посту джерела потрапити в маршрут двічі. Для ручних постів обидва поля `NULL`, і PostgreSQL не вважає такі рядки дублями.
- **`@@unique([postId, destinationId])`** — одна доставка на напрямок; повтор змінює наявний рядок.
- **`externalIds` — масив**, бо альбом у Telegram складається з кількох повідомлень, а для редагування і видалення потрібні всі.
