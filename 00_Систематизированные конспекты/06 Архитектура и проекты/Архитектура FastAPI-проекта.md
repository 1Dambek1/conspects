# Архитектура FastAPI-проекта

Связанные темы: [[FastAPI - базовый конспект|FastAPI]], [[SQLAlchemy и связи|SQLAlchemy]], [[Авторизация, JWT и пароли|авторизация]], [[Docker, сервер и окружение|Docker]].

## Цель архитектуры

Хорошая структура проекта помогает разделить:

- API-роуты;
- бизнес-логику;
- модели БД;
- Pydantic-схемы;
- настройки;
- зависимости;
- фоновые задачи.

## Пример структуры

```text
app/
  main.py
  config.py
  database.py
  dependencies.py
  models/
    user.py
    account.py
  schemas/
    user.py
    account.py
  routers/
    users.py
    auth.py
    accounts.py
  services/
    users.py
    auth.py
  repositories/
    users.py
  tasks/
    worker.py
migrations/
tests/
.env.example
docker-compose.yml
Dockerfile
```

## Роли модулей

- `main.py` - создание FastAPI-приложения и подключение роутеров.
- `config.py` - настройки через pydantic-settings.
- `database.py` - engine, session, зависимость БД.
- `models/` - SQLAlchemy-модели.
- `schemas/` - Pydantic-схемы входа и выхода.
- `routers/` - HTTP-endpoint.
- `services/` - бизнес-логика.
- `repositories/` - запросы к БД.
- `tasks/` - фоновые задачи Celery.

## Принцип разделения

Роутер должен быть тонким:

```python
@router.post("/users")
async def create_user(data: UserCreate, session: AsyncSession = Depends(get_session)):
    return await user_service.create_user(session, data)
```

Логика создания пользователя находится в service-слое, а SQL-запросы можно вынести в repository.

## Микросервисная структура

Микросервисы полезны, когда части системы можно развивать и масштабировать отдельно. Но для учебного проекта лучше начать с модульного монолита: один FastAPI-проект, но с аккуратным разделением по модулям.

Признаки, что микросервис может быть оправдан:

- отдельная команда;
- отдельная база или четкая зона ответственности;
- независимый деплой;
- высокая нагрузка на конкретную часть системы.

## Учебный проект: банковская система

Основные сущности:

- User - пользователь;
- Account - банковский счет;
- Transaction - транзакция;
- Currency - валюта;
- Role или Permission - права доступа.

Функциональные требования:

- регистрация пользователя;
- вход по JWT;
- просмотр и редактирование профиля;
- создание счета;
- просмотр баланса;
- закрытие счета с нулевым балансом;
- пополнение;
- перевод между счетами;
- проверка доступа к своим данным.

Минимальный порядок реализации:

1. Настроить проект, Docker, PostgreSQL.
2. Добавить User, регистрацию и вход.
3. Добавить Account.
4. Добавить Transaction.
5. Проверить права доступа.
6. Покрыть ключевые сценарии тестами.
