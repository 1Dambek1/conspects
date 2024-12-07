# Урок по Pydantic: от Python до FastAPI

## Введение в Pydantic

[Pydantic](https://pydantic-docs.helpmanual.io/)  — это библиотека для проверки, валидации и [[типизацией]] данных в Python, основанная на [[аннотациях]] типов. Она позволяет описывать модели данных и автоматически валидировать входящие значения, обеспечивая безопасность и удобство работы с данными. В FastAPI Pydantic играет ключевую роль, облегчая проверку входных данных и автоматическое создание документации.


## Установка Pydantic

Установите Pydantic через pip:

```bash
pip install pydantic
```

## 1. Основы использования Pydantic в Python

Основной компонент Pydantic — это классы, унаследованные от `BaseModel`. Pydantic использует аннотации типов для проверки входных данных.

### Простой пример
```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    age: int

# Создание экземпляра модели
user = User(id=1, name="John Doe", age=30)

# Доступ к атрибутам модели
print(user.name)  # John Doe
```

### Валидация и преобразование данных

Pydantic автоматически преобразует типы данных и выполняет валидацию:

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    age: int

user = User(id='1', name='Jane Doe', age='25')
print(user)  # id=1 name='Jane Doe' age=25
```

Pydantic автоматически преобразовал строковые значения `'1'` и `'25'` в целочисленные типы.

### Работа с дополнительными методами

- **`.model_dump()`**: Преобразует экземпляр модели в словарь.
- **`.model_dump_json()`**: Преобразует экземпляр модели в строку JSON.
- **`.copy()`**: Создает копию объекта модели.

```python
user_dict = user.model_dump()
print(user_dict)  # {'id': 1, 'name': 'Jane Doe', 'age': 25}

user_json = user.model_dump_json()
print(user_json)  # {"id": 1, "name": "Jane Doe", "age": 25}

```


## 2. Расширенные возможности Pydantic

### Валидация с помощью встроенных валидаторов

Для расширения логики валидации можно использовать декоратор `@validator`.

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    id: int
    name: str
    age: int

    @field_validator('age') # Название поля
    def age_must_be_positive(cls, v): # v - это наше значание. И Далее мы его проверяем 
        if v <= 0:
            raise ValueError('Age must be a positive number')
        return v

```
### Указание значений по умолчанию и опциональных полей

Для опциональных полей используйте модуль `typing`:

```python
from pydantic import BaseModel
from typing import Optional

class User(BaseModel):
    id: int
    name: str
    age: Optional[int] = None  # Поле может быть `None`

```

```python
class User(BaseModel):
    id: int
    name: str
    is_active: bool = True  # Значение по умолчанию

user = User(id=1, name="John Doe")
print(user.is_active)  # True

```
### Вложенные модели

Pydantic поддерживает вложенные модели.

```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    zip_code: str

class User(BaseModel):
    id: int
    name: str
    address: Address

user = User(id=1, name='John Doe', address={'city': 'New York', 'zip_code': '10001'})
print(user.address.city)  # New York
```


### Обработка и конвертация данных

Pydantic может автоматически конвертировать данные к нужным типам.

```python
from datetime import datetime

class Event(BaseModel):
    start_time: datetime

event = Event(start_time="2024-09-18 12:00")
print(event.start_time)  # 2024-09-18 12:00:00

```

# Использование Pydantic с FastAPI

Pydantic идеально подходит для работы с FastAPI, поскольку обеспечивает валидацию входных данных для маршрутов.

### Пример использования моделей Pydantic в FastAPI

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.post("/items/")
async def create_item(item: Item):
    return item

```

 - Здесь модель `Item` определяет ожидаемые данные для запроса. FastAPI автоматически проверяет входящие данные и возвращает ошибку, если они не соответствуют модели.

# 2. Валидация вложенных объектов

Pydantic позволяет обрабатывать вложенные модели:

```python
from typing import List

class SubItem(BaseModel):
    name: str

class Item(BaseModel):
    name: str
    sub_items: List[SubItem]

@app.post("/items/")
async def create_item(item: Item):
    return item

```

# 3. Передача списков значений

Для этого нам нужно обернуть класс pydantic в list[name]

```python
from typing import List

class SubItem(BaseModel):
    name: str

class Item(BaseModel):
    name: str
    sub_items: List[SubItem]

@app.post("/items/")
async def create_item(item: list[Item]):
    return item

```

```python
from typing import List

class SubItem(BaseModel):
    name: str

class Item(BaseModel):
    name: str
    sub_items: List[SubItem]

@app.post("/items/", response_model = list[item])
async def create_item(item: list[Item]):
    return item

```

Тоже самое работает и при обычных значениях 

```python
@app.post("/items/")
async def create_item(item_ids: list[int]):
    return item_ids
```


# 4. Валидация [[response]]

Если мы можем проверять полученные данные, то можем проверять, то что мы отправляем. Для этого после передачи url, через запятую передаем параметр response_model
```python
from typing import List

class SubItem(BaseModel):
    name: str

class Item(BaseModel):
    name: str
    sub_items: List[SubItem]

@app.post("/items/", response_model = list[item])
async def create_item(item: list[Item]):
    return item

```



## Заключение

Pydantic — мощный инструмент для работы с данными в Python, особенно в сочетании с FastAPI. Он обеспечивает строгую типизацию, автоматическую конвертацию типов, валидацию и обработку ошибок. Используя Pydantic, можно улучшить надежность и читаемость кода, а также упростить разработку API на FastAPI.

### Дополнительные ресурсы

- Документация Pydantic
- Документация FastAPI