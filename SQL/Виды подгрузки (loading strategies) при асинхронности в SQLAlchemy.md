
SQLAlchemy предоставляет несколько стратегий подгрузки данных при работе с асинхронными запросами. Это позволяет управлять тем, как загружаются связанные объекты в ORM. Разберем основные стратегии подгрузки: **lazy**, **joined**, **subquery**, **selectin**, **immediate** и **write-only**.

## Подгрузка данных через `options`

Вместо указания стратегии подгрузки в модели можно управлять этим в запросах с помощью `options`. Это дает гибкость, позволяя загружать данные разными способами в зависимости от конкретного запроса.
________________________________________

# 1. Основные
### **1. Lazy loading (ленивая подгрузка, по умолчанию)**

**Описание:**

- Данные загружаются только при первом обращении к атрибуту.
- Использует дополнительный SQL-запрос для подгрузки связанных данных.
- Может привести к проблемам N+1 запросов.

**Пример:**

```python
from sqlalchemy.future import select

with Session() as session:
    result = session.execute(select(Parent))
    parents = result.scalars().all()  # Дети будут загружаться лениво (по умолчанию)
```

# **2. Joined loading (жадная подгрузка через JOIN)**

**Описание:**

- Использует `JOIN` для загрузки связанных данных сразу в одном запросе.
- Уменьшает количество SQL-запросов.
- Может загружать больше данных, чем требуется.

**Пример:**

```python
from sqlalchemy.orm import joinedload

async with AsyncSession() as session:
    result = await session.execute(
        select(Parent).options(joinedload(Parent.children))
    )
    parents = result.scalars().all()
```

# **3.  Selectin loading (подгрузка через IN)**

**Описание:**

- Вместо `JOIN` использует подзапрос.
- Может быть эффективнее `joined`, если `JOIN` приводит к дублированию данных.

**Пример:**

```python
from sqlalchemy.orm import subqueryload

async with AsyncSession() as session:
    result = await session.execute(
        select(Parent).options(subqueryload(Parent.children))
    )
    parents = result.scalars().all()
```

__________________________________________________________________

# 1. Дополнительные

### **4. Subquery loading (жадная подгрузка через подзапрос)**

**Описание:**

- Выполняет два запроса: сначала загружает родительские объекты, затем связанные через `IN`.
- Работает эффективно при подгрузке коллекций.

**Пример:**

```python
from sqlalchemy.orm import selectinload

async with AsyncSession() as session:
    result = await session.execute(
        select(Parent).options(selectinload(Parent.children))
    )
    parents = result.scalars().all()
```

### **5. Immediate loading (немедленная подгрузка)**

**Описание:**

- Связанные данные загружаются сразу при загрузке объекта.
- Использует отдельный SQL-запрос.

**Пример:**

```python
from sqlalchemy.orm import immediateload

async with AsyncSession() as session:
    result = await session.execute(
        select(Parent).options(immediateload(Parent.children))
    )
    parents = result.scalars().all()
```

### **6. Write-only loading (только для записи)**

**Описание:**

- Атрибут доступен только для записи, но не загружается автоматически.
- Полезно для защиты от случайной загрузки данных.

❌ `Write-only` не поддерживается через `options`, так как предназначена только для управления записью данных.

## Вывод

Использование `options` позволяет гибко управлять стратегиями подгрузки на уровне запросов, оптимизируя их под конкретные сценарии. Это особенно полезно в асинхронных приложениях, где важно минимизировать количество запросов и загружать только нужные данные.