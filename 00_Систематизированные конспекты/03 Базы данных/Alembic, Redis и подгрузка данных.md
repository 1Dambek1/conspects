# Alembic, Redis и подгрузка данных

Связанные темы: [[SQLAlchemy и связи]], [[SQL и PostgreSQL]], [[Шаблоны, файлы, WebSocket и фоновые задачи|фоновые задачи]].

## Alembic

Alembic управляет миграциями БД: изменениями структуры таблиц.

Установка:

```bash
pip install alembic
```

Инициализация:

```bash
alembic init migrations
```

Основные команды:

```bash
alembic revision --autogenerate -m "create users table"
alembic upgrade head
alembic downgrade -1
```

В `env.py` нужно подключить metadata моделей:

```python
from app.models import Base

target_metadata = Base.metadata
```

## Асинхронный SQLAlchemy

Установка драйвера:

```bash
pip install asyncpg
```

Пример engine и session:

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

engine = create_async_engine(
    "postgresql+asyncpg://user:password@localhost/app",
)

AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)
```

FastAPI-зависимость:

```python
async def get_session():
    async with AsyncSessionLocal() as session:
        yield session
```

Запрос:

```python
from sqlalchemy import select

result = await session.execute(select(User))
users = result.scalars().all()
```

## Loading strategies

Подгрузка связей важна, чтобы контролировать количество SQL-запросов.

Lazy loading - связь загружается при обращении. В async-коде может приводить к проблемам, если загрузка происходит вне ожидаемого контекста.

Joined loading - связь грузится через `JOIN`:

```python
from sqlalchemy.orm import joinedload

stmt = select(User).options(joinedload(User.orders))
```

Selectin loading - сначала грузятся основные записи, потом связи через `WHERE IN`:

```python
from sqlalchemy.orm import selectinload

stmt = select(User).options(selectinload(User.orders))
```

Для async SQLAlchemy часто удобен `selectinload`.

## Redis

Redis - быстрая in-memory база данных. Часто используется для кеша, временных токенов, rate-limit, брокера Celery.

Запуск через Docker:

```bash
docker run --name redis -p 6379:6379 -d redis
```

Python:

```bash
pip install redis
```

```python
import redis

r = redis.Redis(host="localhost", port=6379, db=0)
r.set("name", "Anna", ex=60)
value = r.get("name")
```

Типы данных:

- string - простое значение;
- hash - объект с полями;
- list - список;
- set - множество;
- sorted set - отсортированное множество.

TTL:

```python
r.set("code:123", "9999", ex=300)
```

Ключ удалится через 300 секунд.
