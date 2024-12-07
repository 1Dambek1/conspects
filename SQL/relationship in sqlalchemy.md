# Relationship в SQLAlchemy с использованием Mapped

## Введение
В SQLAlchemy 2.0 для определения отношений между таблицами используются `Mapped` и `mapped_column`. Эти аннотации обеспечивают строгую типизацию и упрощают чтение и поддержку кода. 

## Основные Типы Отношений

1. **Один ко многим (One-to-Many)**:
   - Одна запись связана с несколькими записями в другой таблице.
   - Пример: один пользователь может иметь несколько статей.

2. **Многие ко многим (Many-to-Many)**:
   - Каждая запись может быть связана с несколькими записями в другой таблице.
   - Требуется промежуточная таблица для связи.
   - Пример: студент может записаться на несколько курсов, а курс может включать нескольких студентов.

3. **Один к одному (One-to-One)**:
   - Один элемент одной таблицы связан с единственным элементом другой таблицы.
   - Пример: пользователь имеет один профиль.

## Использование Relationship

### Один ко многим (One-to-Many)
Для связи "один ко многим" используйте `Mapped` и `relationship` с `mapped_column` для создания внешнего ключа.

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column()
    articles: Mapped[list["Article"]] = relationship(back_populates="author")

class Article(Base):
    __tablename__ = 'articles'
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column()
    user_id: Mapped[int] = mapped_column(ForeignKey('users.id'))
    author: Mapped["User"] = relationship(back_populates="articles")
```

### Многие ко многим (Many-to-Many)

Для связи "многие ко многим" потребуется ассоциативная таблица.
```python
from sqlalchemy import Table, ForeignKey



class StudentCoursre(Base):
	__tablename__ = "studentcourse"
	student_id:Mapped[int] = mapped_column(ForeignKey('students.id')primary_key=True)
	course_id:Mapped[int] = mapped_column(ForeignKey('courses.id')primary_key=True)
	

class Student(Base):
    __tablename__ = 'students'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column()
    courses: Mapped[list["Course"]] = relationship(secondary="studentcourse", back_populates="students",uselist=True)

class Course(Base):
    __tablename__ = 'courses'
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column()
    students: Mapped[list["Student"]] = relationship(secondary="studentcourse", back_populates="courses",uselist=True)

```

### Один к одному (One-to-One)

Для связи "один к одному" используйте `uselist=False` в `relationship`.

```python
class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] 
    profile: Mapped["Profile"] = relationship(back_populates="user", uselist=False)

class Profile(Base):
    __tablename__ = 'profiles'
    id: Mapped[int] = mapped_column(primary_key=True)
    bio: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey('users.id'))
    user: Mapped["User"] = relationship(back_populates="profile")

```

## Дополнительные Параметры Relationship

- **back_populates** – устанавливает двустороннюю связь.
- **secondary** – используется для таблицы связывания в связи "многие ко многим".
- **uselist** – при `False` создает связь "один к одному".