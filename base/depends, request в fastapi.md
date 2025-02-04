# Depends и Request в FastAPI

## Введение
FastAPI — это веб-фреймворк для создания современных веб-приложений и API на Python. Одними из важных компонентов FastAPI являются `Depends` и `Request`. Эти элементы помогают организовать логику приложения, управлять зависимостями и обрабатывать входящие запросы.

## 1. Depends

### Что такое `Depends`
`Depends` — это инструмент, позволяющий определять и внедрять зависимости в функции маршрутов. Зависимость может быть любой функцией, которая возвращает данные или выполняет проверку. FastAPI автоматически вызывает зависимости, выполняет их и передает результат в основную функцию маршрута.

### Пример использования
```python
from fastapi import Depends, FastAPI

app = FastAPI()

def common_parameters(q: str = None, skip: int = 0, limit: int = 10):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons
```

В этом примере функция `common_parameters` внедряется в эндпоинт через `Depends`. Теперь любой запрос к маршруту `/items/` может автоматически получать общий параметр `q`.

### Зависимости для безопасности

`Depends` также широко используется для обработки авторизации и аутентификации.

```python
from fastapi import Depends, FastAPI, HTTPException, status

app = FastAPI()

def verify_token(token: str):
    if token != "mysecrettoken":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN, detail="Invalid token"
        )

@app.get("/secure-data/")
async def read_secure_data(token: str = Depends(verify_token)):
    return {"message": "Secure data"}

```
## 2. Request

### Что такое Request?

`Request` — это объект, который предоставляет доступ к информации о текущем HTTP-запросе, такому как заголовки, параметры, тело запроса и т.д.

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.post("/log/")
async def log_request(request: Request):
    json_body = await request.json()
    return {"method": request.method, "json_body": json_body}

```

В этом примере объект `Request` используется для получения метода HTTP-запроса и его JSON тела.

```python
@app.get("/client-info/")
async def get_client_info(request: Request):
    client_host = request.client.host
    user_agent = request.headers.get('user-agent')
    return {"client_host": client_host, "user_agent": user_agent}

```

