# Обработка ошибок в FastAPI

В FastAPI можно обрабатывать ошибки и создавать собственные исключения для возврата определённых статусов HTTP. Это позволяет гибко реагировать на различные ситуации, возникающие в процессе обработки запросов.

## Стандартные ошибки HTTP

FastAPI автоматически поднимает стандартные HTTP-исключения, используя класс `HTTPException`:

### Пример:

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id < 1:
        raise HTTPException(status_code=400, detail="Invalid item ID")
    return {"item_id": item_id}
```

### Параметры `HTTPException`:

- `status_code`: Код статуса HTTP, который будет возвращён (например, 400, 404, 500).
- [`detail`](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%BA%D0%BE%D0%B4%D0%BE%D0%B2_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F_HTTP): Сообщение, которое будет отправлено в ответе.
- `headers`: Дополнительные заголовки HTTP.

## Кастомные исключения

Можно определить свои классы исключений и настроить их обработку. Это удобно для создания более специфичных ошибок.

### Пример:

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse

class CustomException(Exception):
    def __init__(self, name: str):
        self.name = name

app = FastAPI()

@app.exception_handler(CustomException)
async def custom_exception_handler(request: Request, exc: CustomException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something wrong."},
    )

@app.get("/custom-error/")
async def custom_error():
    raise CustomException(name="TeaPotError")

```


### Пояснение:

- Класс `CustomException` — пользовательское исключение.
- Декоратор `@app.exception_handler` — обработчик для данного типа исключений.