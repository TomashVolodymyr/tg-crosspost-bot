# #13 — MTProto-джерело (GramJS)

Потік C · Блокери: #11

## Мета
Переносити пости з публічного каналу, де бот не може бути адміністратором.

## Що зробити
- Скрипт `npm run session:login` (`scripts/session-login.ts`): інтерактивний вхід GramJS, друк `StringSession`.
- `MtprotoSource`: якщо задані `TG_API_ID`, `TG_API_HASH`, `TG_SESSION`, — підключення на старті; інакше модуль вимкнений.
- Додавання джерела з адмін-бота за `@username` або посиланням: `contacts.ResolveUsername`, вступ у канал (`channels.JoinChannel`) з підтвердженням адміна; якщо вступити не можна — режим опитування історії.
- Події `NewMessage`, `EditedMessage`, `UpdateDeleteChannelMessages` для зареєстрованих каналів.
- `fromMtprotoMessage(message)` → `PostDraft`: текст і сутності GramJS переводяться у формат Bot API; медіа — `client.downloadMedia` у буфер, файли більші за 50 МБ — `SKIPPED`.
- Видалення в джерелі → `adapter.remove` для всіх доставок поста.
- `FLOOD_WAIT` довше за `floodSleepThreshold` — пауза джерела і повідомлення адміну.

## Критерії приймання
Пост у чужому публічному каналі з'являється в цільовому каналі з правилами; видалення в джерелі видаляє копію.

## Нюанси
Див. [platform-notes.md](../platform-notes.md#telegram-mtproto-gramjs): окремий акаунт, сесія як пароль, відповідальне використання.

## Тести
Юніт: перетворення сутностей GramJS у сутності Bot API на фікстурах.
