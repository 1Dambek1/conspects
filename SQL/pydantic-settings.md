# Работа с `.env` и Pydantic Settings в Python

## Введение
Использование `.env` файлов – это удобный способ хранения конфиденциальной информации, такой как пароли, API-ключи и параметры соединения. Библиотека **Pydantic** с модулем `pydantic-settings` позволяет удобно загружать и управлять настройками конфигурации приложения.

## Установка pydantic-settings

Для работы с pydantic-settings необходимо установить саму библиотеку Pydantic:

```bash
pip install pydantic pydantic_settings
```
## Создание файла `.env`

Создайте файл `.env` в корне вашего проекта для хранения переменных окружения. Каждая переменная записывается как `ключ=значение`.

Пример `.env` файла:
```.env
#.env
DB_NAME=my_database
DB_USER=my_user
DB_PASSWORD=my_password
DB_HOST=localhost
DB_PORT=5432

```
## Настройка pydantic-settings для использования `.env`

### Шаг 1: Создание модели настроек с использованием Pydantic

Создайте Python-файл (например, `config.py`) и импортируйте необходимые модули из Pydantic. Используйте класс `BaseSettings`, чтобы создать модель для ваших настроек.
```python
# config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    db_name: str
    db_user: str
    db_password: str
    db_host: str = 'localhost'
    db_port: int = 5432

    class Config:
        env_file = ".env"
config = Settings()

```

### Шаг 2: Инициализация настроек

В основном файле вашего проекта создайте объект настроек, который автоматически загрузит значения из `.env`.

```python
# main.py
from config import config


# Используем переменные
print(f"Подключаемся к базе данных {config.db_name} на хосте {config.db_host}")

```

### Шаг 3: Использование настроек в проекте

Теперь переменные из `.env` файла доступны через объект `settings`, и вы можете обращаться к ним по именам полей.
```python
# Например, для подключения к базе данных
import psycopg2

def connect_to_db():
    conn = psycopg2.connect(
        dbname=settings.db_name,
        user=settings.db_user,
        password=settings.db_password,
        host=settings.db_host,
        port=settings.db_port
    )
    return conn

```

## Параметры конфигурации и валидация

Pydantic позволяет добавлять валидацию для параметров:
```python
from pydantic import BaseSettings, validator

class Settings(BaseSettings):
    db_name: str
    db_user: str
    db_password: str
    db_host: str
    db_port: int

    @validator("db_port")
    def port_must_be_valid(cls, v):
        if v < 1 or v > 65535:
            raise ValueError("db_port должен быть между 1 и 65535")
        return v

    class Config:
        env_file = ".env"

```

## Преимущества использования Pydantic Settings

1. **Чтение из `.env` файла** – автоматически загружает переменные окружения.
2. **Поддержка валидации** – проверка и фильтрация значений.
3. **Удобный доступ к конфигурации** – позволяет сосредоточить все настройки в одном месте.

## Заключение

Использование `.env` и Pydantic Settings упрощает управление конфиденциальными данными и конфигурацией в проекте. Эта комбинация позволяет улучшить структуру кода, облегчить поддержку и повысить безопасность приложения.