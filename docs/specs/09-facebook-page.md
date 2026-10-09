# #9 — Напрямок «сторінка Facebook»

Потік B · Блокери: #8

## Мета
Пост з бота виходить і на сторінку Facebook.

## Що зробити
- Якщо задані `FB_PAGE_ID` і `FB_PAGE_ACCESS_TOKEN`, на старті — запит `GET /{page-id}?fields=name`; успіх — upsert `Destination(kind=FACEBOOK_PAGE)` з назвою сторінки; помилка — лог і напрямок вимкнено.
- `FacebookPageAdapter` на `fetch` без SDK: текст (`/feed`), одне фото (`/photos`, multipart), кілька фото (`published=false` + `/feed` з `attached_media`), відео (`/videos`).
- Медіа з Telegram: завантажити файл через `getFile` у буфер і відправити multipart.
- `toPlainText(text, entities)`: прибрати форматування, `text_link` → «текст (адреса)».
- Коди помилок Facebook — у `classifyError` з #8.

## Поза межами
Групи Facebook — неможливо через API (див. [platform-notes.md](../platform-notes.md#facebook-graph-api)). Кілька сторінок — пізніше, якщо знадобиться.

## Критерії приймання
Текст із посиланням, одне фото і альбом із 3 фото з'являються на тестовій сторінці; недійсний токен вимикає напрямок і пише адміну.

## Тести
Юніт: `toPlainText` (емодзі, вкладене форматування, `text_link`), побудова тіла запиту для кожного типу поста; HTTP — через фейковий `fetch`.
