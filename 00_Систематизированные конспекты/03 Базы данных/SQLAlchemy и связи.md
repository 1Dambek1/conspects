# SQLAlchemy и связи

Связанные темы: [[SQL и PostgreSQL]], [[Alembic, Redis и подгрузка данных]], [[FastAPI - базовый конспект|FastAPI]].

## Что такое SQLAlchemy

SQLAlchemy - библиотека для работы с базами данных из Python. Она позволяет описывать таблицы как Python-классы и выполнять запросы через ORM.

## Базовая модель

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(unique=True)
    name: Mapped[str]
```

## Engine и Session

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine("postgresql://user:password@localhost/app")
SessionLocal = sessionmaker(bind=engine)
```

Зависимость для FastAPI:

```python
def get_session():
    with SessionLocal() as session:
        yield session
```

## One-to-many

Один пользователь может иметь много заказов.

```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    orders: Mapped[list["Order"]] = relationship(back_populates="user")

class Order(Base):
    __tablename__ = "orders"

    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    user: Mapped["User"] = relationship(back_populates="orders")
```

## One-to-one

Один пользователь имеет один профиль.

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    profile: Mapped["Profile"] = relationship(back_populates="user", uselist=False)

class Profile(Base):
    __tablename__ = "profiles"

    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), unique=True)
    user: Mapped["User"] = relationship(back_populates="profile")
```

## Many-to-many

Много пользователей могут быть во многих чатах.

```python
class UserChat(Base):
    __tablename__ = "users_chats"

    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), primary_key=True)
    chat_id: Mapped[int] = mapped_column(ForeignKey("chats.id"), primary_key=True)

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    chats: Mapped[list["Chat"]] = relationship(
        secondary="users_chats",
        back_populates="users",
    )

class Chat(Base):
    __tablename__ = "chats"

    id: Mapped[int] = mapped_column(primary_key=True)
    users: Mapped[list["User"]] = relationship(
        secondary="users_chats",
        back_populates="chats",
    )
```

## CRUD-примеры

Добавление:

```python
user = User(email="anna@example.com", name="Anna")
session.add(user)
session.commit()
```

Выборка:

```python
from sqlalchemy import select

user = session.scalar(select(User).where(User.email == "anna@example.com"))
```

Обновление:

```python
user.name = "Ann"
session.commit()
```

Удаление:

```python
session.delete(user)
session.commit()
```
