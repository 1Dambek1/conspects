# FastAPI - базовый конспект

Связанные темы: [[Pydantic, настройки и .env]], [[Шаблоны, файлы, WebSocket и фоновые задачи]], [[SQLAlchemy и связи|SQLAlchemy]], [[Авторизация, JWT и пароли|авторизация]].

## Что такое FastAPI

FastAPI - Python-фреймворк для создания backend-приложений и API. Он использует:

- Starlette - работа с HTTP, роутингом и ASGI;
- Pydantic - валидация и сериализация данных;
- Swagger UI и ReDoc - автоматическая документация API.

## Минимальное приложение

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    return {"Hello": "World"}
```

Запуск:

```bash
uvicorn main:app --reload --port 8000
```

После запуска:

- `http://127.0.0.1:8000` - приложение;
- `http://127.0.0.1:8000/docs` - Swagger UI.

## Endpoint

Endpoint - конкретная точка API, к которой обращается клиент.

```python
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

`item_id` берется из пути и автоматически приводится к `int`.

## HTTP-методы

- `GET` - получить данные.
- `POST` - создать данные.
- `PUT` - обновить данные.
- `DELETE` - удалить данные.

```python
@app.post("/items/")
async def create_item(item: Item):
    return item

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"id": item_id, **item.model_dump()}

@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    return {"message": f"Item {item_id} deleted"}
```

## Передача данных

Через путь:

```python
@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"user_id": user_id}
```

Через query-параметры:

```python
@app.get("/users")
async def search_users(name: str | None = None):
    return {"name": name}
```

Через тело запроса:

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

@app.post("/items")
async def create_item(item: Item):
    return item
```

## Routers

Роутеры разбивают приложение на модули.

```python
from fastapi import APIRouter

router = APIRouter(prefix="/items", tags=["items"])

@router.get("/")
async def read_items():
    return []
```

Подключение:

```python
from fastapi import FastAPI
from app.routers.items import router as items_router

app = FastAPI()
app.include_router(items_router)
```

## Request и Response

Request - входящий HTTP-запрос: метод, URL, заголовки, параметры, тело.

Response - ответ сервера: статус-код, заголовки, тело.

```python
from fastapi import Request

@app.get("/info")
async def info(request: Request):
    return {"client": request.client.host}
```

## Depends

`Depends` подключает зависимости: сессию БД, текущего пользователя, настройки, проверки доступа.

```python
from fastapi import Depends

def get_token():
    return "token"

@app.get("/profile")
async def profile(token: str = Depends(get_token)):
    return {"token": token}
```

## Ошибки

```python
from fastapi import HTTPException

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    if user_id <= 0:
        raise HTTPException(status_code=400, detail="Invalid user id")
    return {"id": user_id}
```

Стандартные коды:

- `400` - некорректный запрос;
- `401` - не авторизован;
- `403` - нет прав;
- `404` - не найдено;
- `500` - ошибка сервера.
