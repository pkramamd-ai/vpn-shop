# VPN Shop

Telegram-бот для продажи VPN-подписок с интеграцией [Remnawave](https://remna.st/).
Поддерживает оплату Telegram Stars / YooKassa / CryptoBot, реферальную систему,
мини-апп, RU/EN/ZH локализации. Базируется на готовом образе
[Jolymmiels/remnawave-telegram-shop](https://github.com/Jolymmiels/remnawave-telegram-shop).

> _English version below._

---

## 🇷🇺 Деплой за 10 минут

### Что нужно
- VPS с публичным IP (1 CPU / 1 GB RAM достаточно).
- Установленные `docker` и `docker compose`.
- Уже работающая панель **Remnawave** + API-токен.
- Telegram Bot Token от [@BotFather](https://t.me/BotFather).
- (опционально) Доступы к YooKassa / CryptoBot.
- (опционально) Домен с HTTPS под mini-app.

### Шаги

```bash
# 1. Клонируем репо на VPS
git clone https://github.com/<your-username>/vpn-shop.git
cd vpn-shop

# 2. Готовим .env
cp .env.example .env
nano .env   # заполняем TELEGRAM_TOKEN, REMNAWAVE_URL, REMNAWAVE_TOKEN,
            # ADMIN_TELEGRAM_ID, цены и ключи платежей

# 3. Запускаем
docker compose up -d

# 4. Смотрим логи
docker compose logs -f bot
```

Бот стартует, накатывает миграции БД и регистрирует команды `/start`, `/connect`
в Telegram (на RU, EN, ZH). После этого пишешь боту `/start` — он отвечает.

### Что обязательно заполнить в `.env`
| Переменная | Что это |
|---|---|
| `TELEGRAM_TOKEN` | Токен бота из BotFather |
| `ADMIN_TELEGRAM_ID` | Твой Telegram numeric ID |
| `REMNAWAVE_URL` | URL твоей панели (с https://) |
| `REMNAWAVE_TOKEN` | API-токен Remnawave |
| `PRICE_1/3/6/12` | Цены в рублях |
| `STARS_PRICE_1/3/6/12` | Цены в Telegram Stars |

### Платежи
Включай нужные ставя `*_ENABLED=true`:
- `TELEGRAM_STARS_ENABLED` — встроенные Stars (XTR), без доп. ключей.
- `YOOKASA_ENABLED` — банковские карты, рубли. Нужны `YOOKASA_SHOP_ID` + `YOOKASA_SECRET_KEY` из [личного кабинета YooKassa](https://yookassa.ru/my).
- `CRYPTO_PAY_ENABLED` — крипта через [@CryptoBot](https://t.me/CryptoBot). В CryptoBot → Crypto Pay → Create App → копируешь токен в `CRYPTO_PAY_TOKEN`.

### Mini App
Хостишь свой мини-апп на HTTPS-домене и кладёшь его URL в `MINI_APP_URL`.
Бот покажет кнопку «Connect» как `WebApp` вместо обычной ссылки.
В этом репо мини-апп не идёт в комплекте — это отдельный фронтенд проект.
См. [официальную документацию](https://remnawave-telegram-shop-bot-doc.vercel.app/).

### Языки
В папке `translations/` лежат `ru.json`, `en.json`, `zh.json`. Бот определяет
язык юзера по `LanguageCode` из Telegram, фолбэк — `DEFAULT_LANGUAGE` из `.env`.

**Чтобы добавить новый язык:**
1. Скопируй `en.json` → `<код-языка>.json` (например `tr.json` для турецкого).
2. Переведи значения, ключи и `emoji_id` оставь как есть.
3. Перезапусти бот: `docker compose restart bot`.

> Для локализации команд меню бота (Bot Menu) на нестандартный язык нужно
> допатчить `cmd/app/main.go` в форке и собрать свой образ. ZH/RU/EN уже есть.

### Обновление
```bash
docker compose pull
docker compose up -d
```

### Бэкап БД
```bash
docker exec vpn-shop-db pg_dump -U postgres postgres | gzip > backup-$(date +%F).sql.gz
```

---

## 🇬🇧 Quick deploy (10 min)

### Requirements
- VPS with public IP (1 vCPU / 1 GB RAM is enough).
- `docker` + `docker compose`.
- Running **Remnawave** panel + API token.
- Telegram Bot Token from [@BotFather](https://t.me/BotFather).
- (optional) YooKassa / CryptoBot credentials.
- (optional) HTTPS domain for the mini app.

### Steps

```bash
git clone https://github.com/<your-username>/vpn-shop.git
cd vpn-shop
cp .env.example .env
nano .env   # fill in TELEGRAM_TOKEN, REMNAWAVE_*, ADMIN_TELEGRAM_ID, prices, payment keys

docker compose up -d
docker compose logs -f bot
```

The bot will run migrations on startup and register `/start`, `/connect`
commands localized in RU, EN, ZH. Send `/start` to your bot — it should reply.

### Adding more languages
Drop a new `<lang>.json` into `translations/` (e.g. `fa.json` for Farsi),
translate values keeping keys and `emoji_id` intact, and `docker compose
restart bot`. Bot menu commands (the `/` autocomplete) are localized only for
the languages explicitly handled in `cmd/app/main.go`; to add more, fork the
upstream repo and patch it.

### Update
```bash
docker compose pull
docker compose up -d
```

---

## Architecture

```
┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
│ Telegram Bot │────▶│ Bot (Go)        │────▶│ Remnawave    │
│ + Mini App   │     │ + Postgres      │     │ Panel API    │
└──────────────┘     │ (this compose)  │     └──────┬───────┘
                     └────────┬────────┘            │
                              │                ┌────┴────┐
                              ▼                │ Xray    │
                     ┌─────────────────┐       │ nodes   │
                     │ Stars / YooKassa│       └─────────┘
                     │ / CryptoBot     │
                     └─────────────────┘
```

## Useful links
- Upstream code: https://github.com/Jolymmiels/remnawave-telegram-shop
- Docs: https://remnawave-telegram-shop-bot-doc.vercel.app/
- Remnawave panel: https://remna.st/

## License
Upstream is AGPL-3.0. This deploy repo follows the same.
