## 🚢 Развертывание

### Docker

Переменные окружения хранятся в отдельном файле `.env` (не попадает в образ).

1. Скопируйте пример и заполните секреты: `copy .env.example .env` (Windows) или `cp .env.example .env` (Linux/macOS).
2. Для SQLite в контейнере в `.env` укажите: `DB_URL=sqlite:////app/data/bot.db` (данные сохраняются в volume).
3. Соберите и запустите:

```bash
docker compose up -d --build
```

API, бот и веб-панель будут доступны на порту **4000** (фронтенд собирается в образ при `docker compose build`). Остановка: `docker compose down`.

Запуск без docker-compose (только образ, env из файла):

```bash
docker build -t witrix-discordbot .
docker run --env-file .env -e API_HOST=0.0.0.0 -p 4000:4000 -v bot_data:/app/data witrix-discordbot
```

#### Развёртывание без кода (только `docker-compose` + `.env` на любой VM)

На виртуальной машине нужны только два файла: **docker-compose** и **.env**. Исходный код не нужен — подтягивается образ из registry.

**Шаг 1 — один раз с машины, где есть репозиторий:** собрать образ и запушить в Docker Hub (или другой registry):

```bash
docker compose build
docker tag witrix-discordbot:latest YOUR_DOCKERHUB_USER/witrix-discordbot:latest
docker push YOUR_DOCKERHUB_USER/witrix-discordbot:latest
```

**Шаг 2 — на любой VM:** положить в папку два файла.

- **docker-compose.deploy.yml** — скопировать из репозитория и в секции `image:` подставить свой образ, например:
  `image: YOUR_DOCKERHUB_USER/witrix-discordbot:latest`
- **.env** — скопировать из `.env.example`, заполнить `DISCORD_TOKEN`, `SECRET_KEY`, Discord OAuth и при необходимости `DB_URL=sqlite:////app/data/bot.db`.

Затем на VM:

```bash
docker compose -f docker-compose.deploy.yml pull
docker compose -f docker-compose.deploy.yml up -d
```

После этого бот и панель доступны на порту **4000**. Для обновления: снова собрать и запушить образ, на VM выполнить `docker compose -f docker-compose.deploy.yml pull && docker compose -f docker-compose.deploy.yml up -d`.

#### Установка одним скриптом (как PasarGuard)

Чтобы скачать и установить проект на сервере с Linux, выполните:

```bash
curl -fsSL https://raw.githubusercontent.com/witrixs/script/main/witrixdriscordbot/scripts/witrix.sh -o /tmp/witrixdiscordbot.sh \
  && sed -i 's/\r$//' /tmp/witrixdiscordbot.sh \
  && sudo bash /tmp/witrixdiscordbot.sh install \
  && sudo bash /tmp/witrixdiscordbot.sh install-script
```

Скрипт установит Docker (если нужно), скачает `docker-compose` и создаст `.env` в `/opt/witrixdiscordbot`, затем установит команду `witrixdiscordbot` в систему. Дальше: отредактируйте `.env` и запустите бота:

```bash
witrixdiscordbot edit-env   # заполнить DISCORD_TOKEN, DB_URL, SECRET_KEY и т.д.
witrixdiscordbot up
```

**Команды:** `install`, `install-script`, `up`, `down`, `restart`, `status`, `logs`, `update`, `uninstall`, `edit`, `edit-env`, `help`. Каталог установки: `/opt/witrixdiscordbot`.
