# Python - основы для backend

Связанные темы: [[FastAPI - базовый конспект|FastAPI]], [[Асинхронность в Python]], [[Pydantic, настройки и .env|Pydantic]].

## Типизация

Типизация помогает явно описывать, какие данные ожидает функция или модель. В backend это важно, потому что данные приходят из запросов, JSON, базы данных и внешних сервисов.

```python
def get_user(user_id: int) -> dict:
    return {"id": user_id}
```

В FastAPI аннотации типов используются не только для читаемости, но и для валидации входных данных:

```python
@app.get("/users/{user_id}")
async def read_user(user_id: int):
    return {"user_id": user_id}
```

Если передать не число, FastAPI вернет ошибку валидации.

## Частые типы

- `int`, `float`, `str`, `bool` - базовые типы.
- `list[str]` - список строк.
- `dict[str, int]` - словарь, где ключи строки, значения числа.
- `str | None` - строка или `None`.
- `Optional[str]` - старый вариант записи `str | None`.

## Аннотации сложных структур

```python
def calculate_total(prices: list[float]) -> float:
    return sum(prices)

def get_users() -> list[dict[str, str]]:
    return [{"name": "Anna"}]
```

## `map()`

`map()` применяет функцию к каждому элементу итерируемого объекта.

```python
numbers = [1, 2, 3]
squares = list(map(lambda x: x * x, numbers))
```

Используй `map`, когда преобразование короткое и очевидное. Если логика сложная, обычный цикл или list comprehension читается лучше.

```python
squares = [x * x for x in numbers]
```

## `zip()`

`zip()` объединяет несколько последовательностей по элементам.

```python
names = ["apple", "banana"]
prices = [1.2, 0.8]

items = list(zip(names, prices))
# [("apple", 1.2), ("banana", 0.8)]
```

Полезно для параллельной обработки связанных списков:

```python
for name, price in zip(names, prices):
    print(name, price)
```

Если списки разной длины, `zip()` остановится на самом коротком.

## JSON

JSON - текстовый формат обмена данными. В API почти все запросы и ответы передаются как JSON.

```json
{
  "id": 1,
  "name": "Product",
  "price": 120.5,
  "is_active": true
}
```

Правила:

- ключи пишутся в двойных кавычках;
- строки пишутся в двойных кавычках;
- доступны типы: object, array, string, number, boolean, null;
- комментарии в JSON запрещены.

В Python JSON обычно превращается в `dict` или `list`, а FastAPI делает это автоматически.
