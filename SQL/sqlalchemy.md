
# Урок: SQLAlchemy и подключение её к Python

## Оглавление

1. [Что такое SQLAlchemy](#%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-sqlalchemy)
2. [Установка SQLAlchemy](#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-sqlalchemy)
3. [Подключение к базе данных](#%D0%BF%D0%BE%D0%B4%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BA-%D0%B1%D0%B0%D0%B7%D0%B5-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)
4. [Создание моделей](#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D0%B5%D0%B9)
5. [Создание и управление сессиями](#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B8-%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D1%81%D0%B5%D1%81%D1%81%D0%B8%D1%8F%D0%BC%D0%B8)
6. [Основные CRUD-операции](#%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%BD%D1%8B%D0%B5-crud-%D0%BE%D0%BF%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D0%B8)
7. [Асинхронный SQLAlchemy](#%D0%B0%D1%81%D0%B8%D0%BD%D1%85%D1%80%D0%BE%D0%BD%D0%BD%D1%8B%D0%B9-sqlalchemy)
8. [Заключение](#%D0%B7%D0%B0%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5)

---

### Что такое SQLAlchemy

**SQLAlchemy** — это популярная ORM (Object-Relational Mapping) библиотека для Python, которая предоставляет инструменты для работы с базами данных. SQLAlchemy позволяет разработчикам взаимодействовать с базой данных на уровне объектов, а не напрямую через SQL-запросы.

Преимущества использования SQLAlchemy:

- Абстракция SQL-запросов на уровне Python-объектов.
- Автоматизация создания и управления таблицами.
- Поддержка множества СУБД (SQLite, PostgreSQL, MySQL, Oracle и др.).
- Асинхронные возможности (через `SQLAlchemy Asyncio`).

---

### Установка SQLAlchemy

Для начала установим SQLAlchemy с помощью pip:
```bash
pip install sqlalchemy

```
Если вам нужно подключение к асинхронной базе данных (например, с использованием `asyncpg` для PostgreSQL), установите также `SQLAlchemy Asyncio` и драйвер базы данных:

```bash
pip install sqlalchemy[asyncio] asyncpg
```
### Подключение к базе данных

Подключение SQLAlchemy к базе данных осуществляется с использованием объекта `create_engine`, который управляет соединением с СУБД.

```python
from sqlalchemy import create_engine

# Пример подключения к SQLite
engine = create_engine("sqlite:///mydatabase.db")

# Пример подключения к PostgreSQL
# engine = create_engine("postgresql://username:password@localhost/mydatabase")

```
**Аргументы для подключения**:

- **sqlite:///:memory:** — подключение к временной базе данных в памяти.
- **sqlite:///mydatabase.db** — путь к файлу базы данных SQLite.
- **postgresql://username@host/database** — пример подключения к PostgreSQL.
### Создание моделей

Модели в SQLAlchemy — это классы Python, представляющие таблицы базы данных. Чтобы определить модель, нужно наследоваться от `Base`, который создается из `declarative_base`.

```python
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    
    id:Mapped[int] = mapped_column(primary_key = True)
    name:Mapped[str] 
    age:Mapped[int]

# Создаем таблицы в базе данных
Base.metadata.create_all(engine)

```
**Основные компоненты модели**:

- `__tablename__` — название таблицы в базе данных.
- `Mapped` — столбцы таблицы (поля класса), такие как `Integer`, `String` и т.д.
- mapped_column - указание доп параметров 
- `primary_key=True` — указывает на первичный ключ.
### Создание и управление сессиями

Сессии позволяют выполнять запросы и управлять транзакциями в SQLAlchemy.

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Пример использования сессии
new_user = User(name="Alice", age=30)
session.add(new_user)
session.commit()

```

- **sessionmaker** — функция, создающая экземпляры сессий.
- **session.add()** — добавляет объект в сессию.
- **session.commit()** — сохраняет изменения в базе данных.
### Основные CRUD-операции

CRUD — это операции создания (Create), чтения (Read), обновления (Update) и удаления (Delete) данных.

```python
# CREATE
new_user = User(name="Bob", age=25)
session.add(new_user)
session.commit()

# READ
user = session.scalars(select(User).where(User.name=="Alice"))
user.all()

# UPDATE
user.age = 35
session.commit()

# DELETE
session.delete(user)
session.commit()

```