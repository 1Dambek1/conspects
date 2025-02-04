
# Работа с PyJWT и RSA256 в FastAPI

Этот конспект описывает, как использовать библиотеку PyJWT для создания и проверки JWT-токенов с алгоритмом шифрования RSA256 в приложении FastAPI.

## Установка PyJWT

Установим PyJWT через pip:
```bash
pip install pyjwt
```
## Основные понятия JWT

JWT (JSON Web Token) — это стандарт для передачи данных в формате JSON между двумя сторонами, используемый для авторизации и обмена информацией.

### Структура JWT:

1. **Header** — Заголовок, содержит алгоритм шифрования и тип токена.
2. **Payload** — Полезная нагрузка, содержит данные и claims.
3. **Signature** — Подпись, формируется с использованием секретного ключа или пары ключей RSA.

## Генерация ключей RSA

Для работы с RSA256 необходимо иметь пару ключей (закрытый и открытый).

### Генерация ключей в терминале

```bash
# Генерация закрытого ключа
openssl genpkey -algorithm RSA -out private.pem -pkeyopt rsa_keygen_bits:2048

# Извлечение открытого ключа из закрытого
openssl rsa -in private.pem -pubout -out public.pem
```
## Пример создания и валидации JWT с RSA256 в FastAPI

### Конфигурация AuthData

Класс для хранения конфигурации, содержащий пути к файлам ключей и другие параметры аутентификации.

```python
from pydantic import BaseModel
from pathlib import Path

class AuthData(BaseModel):
    private_key: Path = Path("src/app_auth/tokens/private_key.pem")
    public_key: Path = Path("src/app_auth/tokens/public_key.pem")
    algorithm: str = "RS256"
    days: int = 7

config = AuthData()
```

### Создание access-токена

Функция `create_access_token` создает токен для авторизованного пользователя.

```python
import jwt
import datetime

async def create_access_token(
    user_id: int,
    algorithm: str = config.algorithm,
    private_key: str = config.private_key.read_text()
) -> str:
    # Подготовка данных payload
    payload = {
        "user_id": user_id,
        "exp": (datetime.datetime.now(datetime.timezone.utc) + datetime.timedelta(days=config.days)).timestamp()
    }
    # Создание токена
    token = jwt.encode(payload=payload, algorithm=algorithm, key=private_key)
    return token

```
### Валидация access-токена

Функция `valid_access_token` проверяет подлинность токена и возвращает `user_id`, если токен действителен.
```python
from fastapi import HTTPException

async def valid_access_token(
    token: str, 
    algorithm: str = config.algorithm,
    public_key: str = config.public_key.read_text()
) -> int:
    try:
        # Декодирование и проверка токена
        payload = jwt.decode(jwt=token, key=public_key, algorithms=[algorithm])
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token has expired.")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token.")

    # Проверка срока действия токена
    exp = payload.get("exp")
    if exp and exp > datetime.datetime.now(datetime.timezone.utc).timestamp():
        return payload.get("user_id")
    
    raise HTTPException(status_code=401, detail="Token has expired.")

```

### Использование в FastAPI

### Пример получения

```python
bearer = HTTPBearer()

  

async def get_current_id(token:HTTPAuthorizationCredentials = Depends(bearer)):

    user_id = await valid_access_token(token=token.credentials)

    if not user_id:

            raise HTTPException(status_code=426, detail={

                "token":"Your token is not valid",

                "status":426

            })

    return user_id
```
### Объяснение кода

1. **`HTTPBearer()`**:
    
    - `HTTPBearer` — это класс, предоставляемый FastAPI для работы с токенами Bearer (тип токена, используемого в заголовке `Authorization`).
    - Он позволяет нам легко извлекать Bearer-токены из заголовков запросов и передавать их в функции.
2. **Аргумент `token` в функции `get_current_id`**:
    
    - `token: HTTPAuthorizationCredentials = Depends(bearer)`:
        - Здесь `Depends(bearer)` указывает, что FastAPI автоматически вызовет `HTTPBearer`, чтобы извлечь токен из заголовка `Authorization`.
        - `token` — это объект типа `HTTPAuthorizationCredentials`, который содержит `scheme` (тип токена, в данном случае "Bearer") и `credentials` (сам токен).
3. **Проверка токена в `valid_access_token`**:
    
    - `user_id = await valid_access_token(token=token.credentials)`:
        - Здесь `token.credentials` извлекает сам JWT-токен из объекта `token`.
        - Функция `valid_access_token` проверяет токен и возвращает `user_id`, если токен действителен. Если токен недействителен, функция возвращает `None`.
4. **Проверка `user_id` и генерация исключения**:
    
    - `if not user_id`: Проверяется, был ли токен действителен. Если `user_id` отсутствует (токен недействителен), вызывается ошибка:
        - `HTTPException(status_code=426, detail={...})` — создает исключение с кодом состояния `426` (это статус "Upgrade Required", который указывает, что токен необходимо обновить или заменить).
        - Поле `detail` предоставляет информацию об ошибке в виде словаря с полями `"token"` и `"status"`.
5. **Возвращаемое значение**:
    
    - Если токен действителен, функция возвращает `user_id`. Это значение может использоваться в других маршрутах FastAPI для идентификации текущего пользователя, поскольку `get_current_id` часто будет зависимостью, предоставляющей идентификатор текущего пользователя.


### Использование в FastAPI

`get_current_id` можно использовать как зависимость, чтобы автоматически проверять токен пользователя при доступе к защищенным маршрутам.

```python
@app.get("/protected-route")
async def protected_route(user_id: int = Depends(get_current_id)):
    return {"message": "Access granted", "user_id": user_id}

```