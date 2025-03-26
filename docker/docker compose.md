

## 1. Установка Docker и Docker Compose

Перед началом работы убедитесь, что у вас установлены **Docker** и **Docker Compose**.

### Проверка установленных версий:

```sh
docker --version
docker-compose --version
```



## 2. Создание `Dockerfile` для FastAPI

В папке `backend/` создаём `Dockerfile`:

```dockerfile
FROM python

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD gunicorn  src.main:app --workers 1 --worker-class uvicorn.workers.UvicornWorker --bind=0.0.0.0:8000
```

## 3. Создание `docker-compose.yml`

```yaml
version: "3.8"

services:
  db:
    image: postgres
    container_name: db
    restart: always
	env_file:
		- .env
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    ports:
      - 5432:5432

  backend:
    build: ./backend
    container_name: app
    restart: always
    depends_on:
      - db
	env_file:
		- .env
    ports:
      - 8000:8000
    volumes:
      - ./backend:/app


```

## 4. Файл `.env`

Создаём `.env` в корневой папке:

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=adminpassword
POSTGRES_DB=fastapi_db
```


## 5. Запуск проекта

### 1. Запускаем сборку контейнеров:

```sh
docker-compose up --build
```

### 2. Проверяем контейнеры:

```sh
docker ps
```

### 3. Останавливаем контейнеры:

```sh
docker-compose down
```

Теперь FastAPI работает на `http://localhost:8000`, а PostgreSQL – на порту `5432`. 