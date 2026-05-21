# Шаблоны, файлы, WebSocket и фоновые задачи

Связанные темы: [[FastAPI - базовый конспект]], [[Alembic, Redis и подгрузка данных|Redis]], [[Docker, сервер и окружение|Docker]].

## Jinja + FastAPI

Jinja нужен, если FastAPI должен отдавать HTML-страницы, а не только JSON.

Установка:

```bash
pip install jinja2
```

Структура:

```text
app/
  main.py
  templates/
    index.html
```

Пример:

```python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates

app = FastAPI()
templates = Jinja2Templates(directory="templates")

@app.get("/")
async def index(request: Request):
    return templates.TemplateResponse(
        "index.html",
        {"request": request, "title": "Главная"}
    )
```

## Базовый HTML

```html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <title>{{ title }}</title>
</head>
<body>
  <h1>{{ title }}</h1>
</body>
</html>
```

## Загрузка файлов

FastAPI принимает файлы через `UploadFile` и `File`.

```python
import shutil
from fastapi import File, UploadFile

@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    file_path = f"uploads/{file.filename}"
    with open(file_path, "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
    return {"path": file_path}
```

Для продакшена лучше генерировать безопасное имя файла через `uuid`, проверять расширение и размер.

## WebSocket

WebSocket нужен для постоянного соединения: чат, уведомления, онлайн-статусы.

```python
from fastapi import WebSocket, WebSocketDisconnect

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            message = await websocket.receive_text()
            await websocket.send_text(f"Echo: {message}")
    except WebSocketDisconnect:
        pass
```

В исходных заметках WebSocket связан с чатами, пользователями и сообщениями через SQLAlchemy-связи many-to-many.

## BackgroundTasks

`BackgroundTasks` подходит для простых задач после ответа клиенту: отправить письмо, записать лог, удалить временный файл.

```python
from fastapi import BackgroundTasks

def send_email(email: str):
    print(f"Send email to {email}")

@app.post("/register")
async def register(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(send_email, email)
    return {"status": "created"}
```

## Celery + Redis

Celery подходит для тяжелых и надежных фоновых задач. Redis часто используется как broker.

```python
from celery import Celery

celery_app = Celery(
    "worker",
    broker="redis://redis:6379/0",
    backend="redis://redis:6379/0",
)

@celery_app.task
def process_video(video_id: int):
    return video_id
```

## Flower

Flower - веб-интерфейс для мониторинга Celery-задач.

Типичный запуск в Docker Compose:

```yaml
flower:
  image: mher/flower
  command: celery --broker=redis://redis:6379/0 flower
  ports:
    - "5555:5555"
```

После запуска интерфейс доступен на `http://localhost:5555`.
