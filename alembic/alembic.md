 Alembic для FastAPI + SQLAlchemy

## 1. Введение

Alembic — это инструмент миграции базы данных для SQLAlchemy, который позволяет управлять изменениями схемы базы данных.

FastAPI часто используется с SQLAlchemy для работы с базами данных, и Alembic помогает вносить изменения в структуру таблиц без потери данных.

## 2. Установка Alembic

```bash
pip install alembic
```

## 3. Инициализация Alembic

В корневой директории проекта выполните команду:

```bash
alembic init alembic
```

Это создаст папку `alembic` с файлами конфигурации.

## 4. Настройка Alembic

### 4.1. Конфигурация подключения к базе данных

Откройте файл `alembic.ini` и найдите строку:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Замените её на вашу строку подключения к базе данных или используйте переменные окружения.

### 4.2. Настройка модуля `env.py`

В файле `alembic/env.py` замените:

```python
from my_project.database import Base

target_metadata = Base.metadata
```

где `Base` — это базовый класс моделей SQLAlchemy.

## 5. Создание миграций

### 5.1. Генерация миграции

Если у вас уже есть модели SQLAlchemy, можно автоматически создать миграцию:

```bash
alembic revision --autogenerate -m "initial migration"
```

Если требуется создать пустую миграцию:

```bash
alembic revision -m "new migration"
```

### 5.2. Применение миграций

```bash
alembic upgrade head
```

### 5.3. Откат миграции

```bash
alembic downgrade -1
```

### 5.4. Полный откат базы данных

```bash
alembic downgrade base
```

## 6. Пример использования с FastAPI

### 6.1. Определение модели

```python
from sqlalchemy import Column, Integer, String
from my_project.database import Base

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
```

### 6.2. Автоматическая генерация миграции

```bash
alembic revision --autogenerate -m "add users table"
alembic upgrade head
```

