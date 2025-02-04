# PyJWT в Python

## Установка

```sh
pip install pyjwt
```

## Основные функции

### Создание JWT-токена

```python
import jwt
import datetime

SECRET_KEY = "my_secret_key"

def create_token(data, exp_minutes=30):
    payload = {
        "exp": datetime.datetime.utcnow() + datetime.timedelta(minutes=exp_minutes),
        "iat": datetime.datetime.utcnow(),
        "data": data
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")
    return token

data = {"user_id": 123}
token = create_token(data)
print(token)
```

### Декодирование и проверка JWT-токена

```python
def decode_token(token):
    try:
        decoded_data = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return decoded_data
    except jwt.ExpiredSignatureError:
        return "Token expired"
    except jwt.InvalidTokenError:
        return "Invalid token"

decoded = decode_token(token)
print(decoded)
```

## Принципы работы

- Основан на стандарте JSON Web Token (JWT).
- Использует подпись для проверки целостности токена.
- Может содержать метаданные (например, `exp` для времени истечения).

## Лучшие практики

- Использовать безопасный секретный ключ (`SECRET_KEY`).
- Ограничивать время жизни токена (`exp`).
- Использовать `RS256` (асимметричное шифрование) для более высокой безопасности.

## Альтернативы

- `authlib`
- `djangorestframework-simplejwt`
- `pyjwt[crypto]` для поддержки асимметричных алгоритмов

## Полезные ссылки

- [Документация PyJWT](https://pyjwt.readthedocs.io/)