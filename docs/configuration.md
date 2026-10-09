# Налаштування

Усі параметри — змінні середовища. Шаблон — [`.env.example`](../.env.example); застосунок перевіряє їх на старті і не запускається, якщо обов'язкової змінної немає або вона має неправильний формат.

## Змінні

| Змінна | Обов'язкова | Опис |
| --- | --- | --- |
| `DATABASE_URL` | так | Рядок підключення PostgreSQL. |
| `BOT_TOKEN` | так | Токен бота від @BotFather. |
| `ADMIN_IDS` | так | Telegram id адмінів через кому. Свій id можна дізнатись у @userinfobot. |
| `LOG_LEVEL` | ні | `debug`, `info`, `warn`, `error`; за замовчуванням `info`. |
| `DELIVERY_POLL_INTERVAL_MS` | ні | Як часто воркер шукає доставки; 3 000. |
| `DELIVERY_BATCH_SIZE` | ні | Скільки доставок воркер бере за раз; 20. |
| `MAX_ATTEMPTS` | ні | Спроб до статусу `FAILED`; 5. |
| `ALBUM_WAIT_MS` | ні | Скільки чекати наступну частину альбому; 1 500. |
| `TG_API_ID`, `TG_API_HASH`, `TG_SESSION` | для `MtprotoSource` | Доступ до Telegram від імені користувача. Без них MTProto-джерела вимкнені. |
| `MTPROTO_POLL_INTERVAL_S` | ні | Інтервал опитування каналів, у які акаунт не вступив; 60. |
| `FB_GRAPH_VERSION` | для Facebook | Версія Graph API, наприклад `v23.0`; актуальну дивитись у документації Meta. |
| `FB_PAGE_ID`, `FB_PAGE_ACCESS_TOKEN` | для Facebook | Сторінка і її токен. Без них напрямок Facebook не реєструється. |

## Бот у Telegram

1. У @BotFather: `/newbot`, зберегти токен у `BOT_TOKEN`.
2. Там само `/setjoingroups` — Enable, щоб бота можна було додавати в групи.
3. Додати бота адміністратором у кожну групу чи канал, куди публікувати; для каналу — з правом «Публікація повідомлень». Бот напише адміну і запропонує зареєструвати напрямок.
4. Для каналу-джерела, яким ви керуєте, — так само додати бота адміністратором; права публікації там не потрібні.

## Сесія MTProto

Потрібна лише для джерел, де бот не може бути адміністратором.

1. Увійти на my.telegram.org акаунтом, від імені якого бот читатиме канали (краще окремим), відкрити API development tools і створити застосунок. `api_id` і `api_hash` — у `TG_API_ID` і `TG_API_HASH`.
2. Виконати `npm run session:login`, ввести номер, код з Telegram і пароль 2FA, якщо він є.
3. Скопіювати надрукований рядок у `TG_SESSION`. Рядок дає повний доступ до акаунта — зберігати як пароль.

## Сторінка Facebook

1. На developers.facebook.com створити застосунок типу Business і додати продукт Facebook Login for Business.
2. У Graph API Explorer обрати застосунок, запросити токен користувача з дозволами `pages_show_list`, `pages_manage_posts`, `pages_read_engagement`.
3. Обміняти його на довгостроковий: `GET /oauth/access_token?grant_type=fb_exchange_token&client_id={app-id}&client_secret={app-secret}&fb_exchange_token={short-token}`.
4. З довгостроковим токеном викликати `GET /me/accounts`: у відповіді для кожної сторінки є `id` і `access_token`. Це `FB_PAGE_ID` і `FB_PAGE_ACCESS_TOKEN`; такий токен сторінки не має строку дії.
5. Перевірити токен у Access Token Debugger: `Expires` — Never, серед дозволів є `pages_manage_posts`.

Якщо змінити пароль Facebook або забрати роль на сторінці, токен стане недійсним; бот вимкне напрямок і напише адміну.
