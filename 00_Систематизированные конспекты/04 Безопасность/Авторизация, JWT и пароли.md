# Авторизация, JWT и пароли

Связанные темы: [[FastAPI - базовый конспект|FastAPI]], [[Pydantic, настройки и .env|настройки и секреты]], [[SQLAlchemy и связи|пользователи в БД]].

## Аутентификация и авторизация

Аутентификация отвечает на вопрос: кто пользователь?

Авторизация отвечает на вопрос: что ему разрешено?

Пример:

- пользователь вошел по email и паролю - это аутентификация;
- пользователь может открыть админку только с ролью `admin` - это авторизация.

## JWT

JWT - токен, который клиент хранит и отправляет серверу. Обычно используется схема:

1. Пользователь отправляет email и пароль.
2. Сервер проверяет пароль.
3. Сервер выдает access token.
4. Клиент отправляет токен в заголовке `Authorization: Bearer <token>`.
5. Сервер проверяет подпись и срок действия токена.

## Структура JWT

JWT состоит из трех частей:

```text
header.payload.signature
```

В payload обычно кладут:

- `sub` - id пользователя;
- `exp` - срок действия;
- `role` или permissions - если нужны права.

## PyJWT

Установка:

```bash
pip install pyjwt
```

Создание токена:

```python
import jwt
from datetime import datetime, timedelta, timezone

SECRET_KEY = "secret"
ALGORITHM = "HS256"

payload = {
    "sub": "1",
    "exp": datetime.now(timezone.utc) + timedelta(minutes=30),
}

token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
```

Проверка:

```python
try:
    data = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
except jwt.ExpiredSignatureError:
    raise HTTPException(status_code=401, detail="Token expired")
except jwt.InvalidTokenError:
    raise HTTPException(status_code=401, detail="Invalid token")
```

## RSA256

RS256 использует пару ключей:

- private key - подписывает токен;
- public key - проверяет токен.

Генерация:

```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

## Хеширование паролей

Пароли нельзя хранить в открытом виде. Их хранят как хеш.

Хорошие варианты:

- Argon2id;
- bcrypt.

Не используй MD5, SHA1, SHA256 или SHA512 для паролей как обычный хеш. Они слишком быстрые для перебора.

## Argon2

```bash
pip install argon2-cffi
```

```python
from argon2 import PasswordHasher

ph = PasswordHasher()
hashed_password = ph.hash("password")

ph.verify(hashed_password, "password")
```

Рекомендации из исходных заметок:

- память: 64-256 МБ;
- параллелизм: 1-4;
- итерации: 2-4;
- соль хранится вместе с хешем;
- минимум 16 байт соли;
- rate-limit на вход обязателен.

## bcrypt

```bash
pip install bcrypt
```

```python
import bcrypt

password = b"secret"
hashed = bcrypt.hashpw(password, bcrypt.gensalt())

is_valid = bcrypt.checkpw(password, hashed)
```

## Практика для FastAPI

- При регистрации: валидировать данные, хешировать пароль, сохранить пользователя.
- При входе: найти пользователя, проверить пароль, выдать JWT.
- На защищенных endpoint: достать токен, проверить, получить текущего пользователя.
- Секреты хранить в `.env`, а не в коде.
