
## Установка

```sh
pip install bcrypt
```

## Основные функции

### Хеширование пароля

```python
import bcrypt

password = b"my_secure_password"
salt = bcrypt.gensalt()
hashed_password = bcrypt.hashpw(password, salt)
print(hashed_password)
```

### Проверка пароля

```python
entered_password = b"my_secure_password"
if bcrypt.checkpw(entered_password, hashed_password):
    print("Пароль верный")
else:
    print("Неверный пароль")
```

### Генерация соли с разной сложностью

```python
salt = bcrypt.gensalt(rounds=12)
print(salt)
```

## Принципы работы

- Использует алгоритм Blowfish для хеширования паролей.
- Применяет технику "salting" для защиты от атак по радужным таблицам.
- Регулируемая сложность вычислений (число раундов).

## Лучшие практики

- Использовать минимум `12` раундов (`rounds=12`) для повышения стойкости к атакам.
- Не хранить исходные пароли, только хеши.
- Использовать `bcrypt` для хранения паролей, но не для шифрования данных.

## Альтернативы

- `argon2-cffi`
- `pbkdf2` (медленнее, но встроен в hashlib)
- `scrypt` (альтернативный медленный хеш)

## Полезные ссылки

- [Документация bcrypt](https://pypi.org/project/bcrypt/)
- [Официальный репозиторий](https://github.com/pyca/bcrypt)