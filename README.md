# Tele-Копай! v6 — Cloudflare версия для GitHub и Telegram

Эта версия сделана специально под загрузку в Cloudflare через `https://dash.cloudflare.com`.

Архитектура:

```text
Telegram Bot / Mini App
        ↓
Cloudflare Pages: frontend
        ↓
Cloudflare Pages Functions: backend API
        ↓
OpenRouter: AI-распознавание схемы
```

Telegram сам не хостит backend. Mini App — это веб-приложение, открываемое внутри Telegram по HTTPS-ссылке. Поэтому приложение размещается на Cloudflare Pages, а Telegram открывает его как Mini App.

## Что внутри

```text
public/                  # frontend Tele-Копай!
  index.html
  app.js
  styles.css
  assets/logo.svg
  assets/logo.png

functions/api/           # Cloudflare Pages Functions backend
  health.js              # проверка API
  calculate.js           # расчет объемов
  recognize.js           # OpenRouter AI / локальный fallback
  telegram-webhook.js    # ответ на /start через Telegram webhook
  set-webhook.js         # вспомогательная установка webhook

.gitignore               # чтобы ключи не ушли в GitHub
.env.example             # пример переменных, без реальных ключей
wrangler.toml            # конфиг Cloudflare
package.json             # команды для локального запуска / deploy
BOTFATHER_TEXTS.md       # готовые тексты для BotFather
```

## Важно про ключи

Реальные ключи нельзя хранить в GitHub.

Не добавляй в репозиторий:

```text
.env
.dev.vars
```

Ключи добавляются в Cloudflare Dashboard:

```text
Cloudflare Dashboard → Workers & Pages → твой Pages project → Settings → Environment variables
```

## 1. Подготовка GitHub

1. Создай репозиторий на GitHub, например:

```text
tele-kopai-cloudflare
```

2. Распакуй этот проект.

3. В терминале внутри папки проекта выполни:

```bash
git init
git add .
git commit -m "Initial Tele-Kopai Cloudflare app"
git branch -M main
git remote add origin https://github.com/ТВОЙ_ЛОГИН/tele-kopai-cloudflare.git
git push -u origin main
```

Проверь, что `.env` и `.dev.vars` не попали в GitHub.

## 2. Создание проекта в Cloudflare Pages через Dashboard

1. Открой:

```text
https://dash.cloudflare.com
```

2. Перейди:

```text
Workers & Pages → Create application → Pages → Connect to Git
```

3. Подключи GitHub.

4. Выбери репозиторий:

```text
tele-kopai-cloudflare
```

5. Настройки сборки:

```text
Framework preset: None
Build command: пусто
Build output directory: public
Root directory: / или пусто
```

6. Нажми:

```text
Save and Deploy
```

После деплоя Cloudflare даст ссылку вида:

```text
https://tele-kopai-cloudflare.pages.dev
```

Открой ее в браузере. Должно открыться приложение Tele-Копай!.

## 3. Переменные окружения в Cloudflare

Открой:

```text
Workers & Pages → tele-kopai-cloudflare → Settings → Environment variables
```

Добавь переменные для Production:

```env
MINI_APP_URL=https://ТВОЙ-PAGES-ПРОЕКТ.pages.dev
APP_BASE_URL=https://ТВОЙ-PAGES-ПРОЕКТ.pages.dev
APP_NAME=Tele-Копай!
AI_PROVIDER=openrouter
OPENROUTER_MODEL=АКТУАЛЬНАЯ_БЕСПЛАТНАЯ_VISION_МОДЕЛЬ_ИЗ_OPENROUTER
```

Добавь секреты:

```env
OPENROUTER_API_KEY=твой_новый_OpenRouter_ключ
BOT_TOKEN=твой_новый_токен_бота_из_BotFather
WEBHOOK_SETUP_SECRET=любой_длинный_пароль_для_установки_webhook
```

Если в Dashboard есть выбор между обычной переменной и Secret, ключи лучше добавлять как Secret.

## 4. Где взять бесплатную модель OpenRouter

1. Открой:

```text
https://openrouter.ai/models?max_price=0
```

2. Выбери модель, которая поддерживает изображения / vision.

3. Скопируй ее Model ID.

4. Вставь в Cloudflare:

```env
OPENROUTER_MODEL=СКОПИРОВАННЫЙ_MODEL_ID
```

Бесплатные модели могут меняться, поэтому в проекте оставлено место для правки, а не жестко зашитая модель.

## 5. Проверка API

После деплоя открой:

```text
https://ТВОЙ-PAGES-ПРОЕКТ.pages.dev/api/health
```

Должно быть:

```json
{
  "ok": true,
  "app": "Tele-Копай!",
  "runtime": "Cloudflare Pages Functions"
}
```

## 6. Настройка Telegram Mini App через BotFather

Открой Telegram → `@BotFather`.

### Название

```text
/setname
```

Вставь:

```text
Tele-Копай!
```

### Описание

```text
/setdescription
```

Вставь текст из `BOTFATHER_TEXTS.md`.

### Короткое описание

```text
/setabouttext
```

Вставь короткий текст из `BOTFATHER_TEXTS.md`.

### Аватарка

```text
/setuserpic
```

Отправь файл:

```text
public/assets/logo.png
```

### Кнопка меню

```text
/setmenubutton
```

Текст кнопки:

```text
Открыть Tele-Копай!
```

URL:

```text
https://ТВОЙ-PAGES-ПРОЕКТ.pages.dev
```

После этого кнопка будет открывать приложение внутри Telegram.

## 7. Чтобы бот отвечал на `/start`

Это необязательно, потому что Mini App уже открывается через кнопку меню. Но если хочешь, чтобы `/start` присылал кнопку:

1. Убедись, что в Cloudflare заданы:

```env
BOT_TOKEN=...
MINI_APP_URL=https://ТВОЙ-PAGES-ПРОЕКТ.pages.dev
APP_BASE_URL=https://ТВОЙ-PAGES-ПРОЕКТ.pages.dev
WEBHOOK_SETUP_SECRET=любой_длинный_пароль
```

2. Открой в браузере:

```text
https://ТВОЙ-PAGES-ПРОЕКТ.pages.dev/api/set-webhook?secret=ТВОЙ_WEBHOOK_SETUP_SECRET
```

3. Должен прийти ответ от Telegram `ok: true`.

4. После этого отправь боту:

```text
/start
```

Бот должен ответить кнопкой `Открыть Tele-Копай!`.

После установки webhook можно удалить файл:

```text
functions/api/set-webhook.js
```

или оставить, но держать `WEBHOOK_SETUP_SECRET` сложным.

## 8. Локальный запуск через Wrangler

Не обязательно, но можно проверить локально:

```bash
npm install
npm run dev
```

Для локальных секретов создай `.dev.vars`:

```env
MINI_APP_URL=http://localhost:8788
APP_BASE_URL=http://localhost:8788
APP_NAME=Tele-Копай!
AI_PROVIDER=openrouter
OPENROUTER_API_KEY=твой_ключ
OPENROUTER_MODEL=твоя_модель
BOT_TOKEN=твой_токен
WEBHOOK_SETUP_SECRET=локальный_пароль
```

`.dev.vars` уже добавлен в `.gitignore`.

## 9. Как пользоваться

1. Открой бота в Telegram.
2. Нажми `Открыть Tele-Копай!`.
3. Загрузи фото схемы.
4. Отметь пальцем ННБ, траншею, котлован или пересечение.
5. Проверь размеры в таблице.
6. Нажми `Сформировать ВОР / КС-2`.
7. Скачай CSV ВОР или КС-2.

## 10. Ограничения v6

- Excel `.xlsx` заменен на CSV, чтобы проект работал бесплатно на Cloudflare без Python backend.
- AI-распознавание зависит от доступности бесплатной vision-модели в OpenRouter.
- Если AI не настроен, приложение работает через ручную разметку и локальный текстовый разбор.
- Файлы не сохраняются на сервере, всё работает в браузере пользователя.

## 11. Места для правок

- `public/index.html` — тексты интерфейса.
- `public/app.js` — расчет, таблицы, CSV, логика разметки.
- `functions/api/recognize.js` — промпт и AI-распознавание.
- `functions/api/calculate.js` — формула объемов.
- `.env.example` — пример переменных.
- `BOTFATHER_TEXTS.md` — описание для Telegram.
