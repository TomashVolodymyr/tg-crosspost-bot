# #12 — Синхронізація редагувань

Потік C · Блокери: #11

## Мета
Виправлення в пості джерела з'являється і в копіях.

## Що зробити
- Обробник `edited_channel_post`: знайти `Post` за `sourceId` і `message_id` у `sourceMessageIds`, застосувати правила заново.
- `TelegramAdapter.edit`: `editMessageText` або `editMessageCaption`; заміна самого медіа — `editMessageMedia`.
- `FacebookPageAdapter.edit`: `POST /{post-id}` з новим `message`; заміну фото Facebook не підтримує — лише лог.
- Якщо після редагування спрацювало `SKIP_IF_CONTAINS` — копію видалити.
- Telegram повертає помилку «message is not modified», якщо текст не змінився, — це не помилка.

## Критерії приймання
Змінити текст і підпис до фото в джерелі — копії в Telegram і на сторінці оновились.

## Тести
Юніт: вибір методу редагування за типом поста і типом зміни.
