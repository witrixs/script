# Этап 1: сборка фронтенда
FROM node:20-alpine AS frontend-build

WORKDIR /frontend

COPY frontend/package*.json ./
RUN npm install --legacy-peer-deps

COPY frontend/ ./
# Относительные URL к API (панель и API на одном хосте)
ENV VITE_API_BASE_URL=
RUN npm run build

# Этап 2: бот + API + статика
FROM python:3.12-slim

WORKDIR /app

# Зависимости системы (для psycopg/PostgreSQL при необходимости)
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Python-зависимости
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Код приложения
COPY app/ ./app/
COPY main.py .

# Собранный фронтенд из первого этапа
COPY --from=frontend-build /frontend/dist ./frontend/dist

# Каталог для SQLite (монтируется volume в docker-compose)
RUN mkdir -p /app/data

# Порт API (переменные API_HOST/API_PORT задаются через .env)
EXPOSE 4000

# Запуск бота и API (env подключается через docker-compose env_file или --env-file)
CMD ["python", "-u", "main.py"]
