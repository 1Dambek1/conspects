# Pydantic, настройки и .env

Связанные темы: [[FastAPI - базовый конспект]], [[Docker, сервер и окружение|Docker и окружение]], [[Авторизация, JWT и пароли|секреты и ключи]].

## Pydantic

Pydantic описывает структуру данных и проверяет входные значения.

```python
from pydantic import BaseModel

class UserCreate(BaseModel):
    email: str
    password: str
    age: int | None = None
```

Если клиент отправит `age` строкой `"18"`, Pydantic попытается привести ее к числу. Если привести нельзя, FastAPI вернет ошибку валидации.

## Модели для API

Обычно удобно разделять схемы:

```python
class UserCreate(BaseModel):
    email: str
    password: str

class UserRead(BaseModel):
    id: int
    email: str
```

Пароль не должен попадать в схему ответа.

## `model_dump()`

В Pydantic v2 данные модели можно превратить в словарь:

```python
data = user.model_dump()
```

Это полезно перед сохранением или передачей данных дальше.

## `.env`

`.env` хранит настройки, которые не должны быть зашиты в код:

```env
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=app
SECRET_KEY=change-me
```

## pydantic-settings

Установка:

```bash
pip install pydantic-settings
```

Пример:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    db_host: str
    db_port: int = 5432
    db_user: str
    db_password: str
    db_name: str
    secret_key: str

    model_config = SettingsConfigDict(env_file=".env")

settings = Settings()
```

## DSN для базы данных

```python
database_url = (
    f"postgresql+asyncpg://{settings.db_user}:"
    f"{settings.db_password}@{settings.db_host}:"
    f"{settings.db_port}/{settings.db_name}"
)
```

## Практика

- Не коммить `.env` в Git.
- Добавляй `.env` в `.gitignore`.
- В репозитории храни `.env.example` без настоящих секретов.
- Для Docker передавай переменные через `env_file` или `environment`.
