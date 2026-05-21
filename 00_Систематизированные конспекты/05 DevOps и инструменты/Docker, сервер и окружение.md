# Docker, сервер и окружение

Связанные темы: [[Pydantic, настройки и .env|.env]], [[Шаблоны, файлы, WebSocket и фоновые задачи|Celery и Redis]], [[Git, GitHub и Pull Request]].

## Docker

Docker упаковывает приложение и зависимости в контейнер. Это помогает запускать проект одинаково на разных машинах.

Основные понятия:

- image - образ приложения;
- container - запущенный экземпляр образа;
- Dockerfile - инструкция сборки образа;
- volume - постоянное хранилище данных;
- network - сеть между контейнерами.

## Основные команды

```bash
docker --version
docker ps
docker ps -a
docker images
docker build -t app .
docker run -p 8000:8000 app
docker stop <container_id>
docker logs <container_id>
```

## Dockerfile для FastAPI

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Docker Compose

`docker-compose.yml` описывает несколько сервисов: API, PostgreSQL, Redis, Celery.

```yaml
services:
  api:
    build: .
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"

  redis:
    image: redis:7
    ports:
      - "6379:6379"
```

Запуск:

```bash
docker compose up --build
docker compose down
```

## Ubuntu-сервер

Базовая подготовка:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git curl ca-certificates
```

Установка Docker обычно включает:

1. добавить GPG-ключ Docker;
2. подключить официальный репозиторий;
3. установить Docker Engine и Docker Compose plugin;
4. проверить `docker --version` и `docker compose version`.

## uv

`uv` - быстрый инструмент для управления Python-проектом и окружением.

Установка Windows:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Создание окружения:

```bash
uv venv
```

Активация Windows:

```powershell
.venv\Scripts\activate
```

Установка пакетов:

```bash
uv pip install fastapi uvicorn
```

## Dropbox API

Для загрузки видео в Dropbox обычно нужны:

1. включить offline access;
2. получить App key и App secret;
3. получить authorization code;
4. обменять code на refresh token;
5. использовать refresh token для получения access token.

Секреты Dropbox тоже хранятся в `.env`.
