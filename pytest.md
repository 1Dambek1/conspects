# Pytest: конспект по написанию тестов

## 1. Зачем нужны тесты

Тесты помогают проверить, что код работает правильно сейчас и не ломается после изменений.

Обычно тест проверяет три вещи:

1. Подготовка данных.
2. Вызов функции или метода.
3. Проверка результата.

Такую структуру часто называют Arrange, Act, Assert.

```python
def test_sum_numbers():
    numbers = [1, 2, 3]

    result = sum(numbers)

    assert result == 6
```

## 2. Установка pytest

```bash
pip install pytest
```

Запуск тестов:

```bash
pytest
```

Запуск с подробным выводом:

```bash
pytest -v
```

Запуск конкретного файла:

```bash
pytest tests/test_math.py
```

Запуск конкретного теста:

```bash
pytest tests/test_math.py::test_sum_numbers
```

## 3. Как pytest находит тесты

Pytest автоматически ищет:

- файлы с именами `test_*.py` или `*_test.py`;
- функции с именами `test_*`;
- классы с именами `Test*`, если внутри есть методы `test_*`.

Пример структуры проекта:

```text
project/
    app/
        calculator.py
    tests/
        test_calculator.py
```

## 4. Простейший тест функции

Код приложения:

```python
# app/calculator.py

def add(a, b):
    return a + b
```

Тест:

```python
# tests/test_calculator.py

from app.calculator import add


def test_add_positive_numbers():
    result = add(2, 3)

    assert result == 5
```

## 5. assert в pytest

В pytest обычно используют обычный Python-оператор `assert`.

```python
assert user.name == "Ivan"
assert len(items) == 3
assert "admin" in user.roles
assert response.status_code == 200
assert result is True
assert value is None
```

Если проверка не пройдет, pytest покажет, какие значения сравнивались.

## 6. Тестирование ошибок

Если функция должна выбрасывать исключение, используют `pytest.raises`.

```python
import pytest


def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b


def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)
```

Можно проверить текст ошибки:

```python
def test_divide_by_zero_message():
    with pytest.raises(ValueError, match="Cannot divide"):
        divide(10, 0)
```

## 7. Несколько похожих проверок: parametrize

Если нужно проверить одну функцию на разных данных, используют `@pytest.mark.parametrize`.

```python
import pytest


def is_even(number):
    return number % 2 == 0


@pytest.mark.parametrize(
    "number, expected",
    [
        (2, True),
        (3, False),
        (0, True),
        (-4, True),
    ],
)
def test_is_even(number, expected):
    assert is_even(number) is expected
```

Это лучше, чем писать много почти одинаковых тестов.

## 8. Фикстуры

Фикстура - это функция, которая готовит данные для теста.

```python
import pytest


@pytest.fixture
def user():
    return {
        "id": 1,
        "name": "Anna",
        "is_active": True,
    }


def test_user_is_active(user):
    assert user["is_active"] is True
```

Pytest сам передаст фикстуру в тест, если имя аргумента совпадает с именем фикстуры.

## 9. Фикстуры для подготовки и очистки

Фикстура может выполнить код до теста и после теста.

```python
import pytest


@pytest.fixture
def file_with_text(tmp_path):
    file_path = tmp_path / "example.txt"
    file_path.write_text("hello", encoding="utf-8")

    yield file_path

    # Код после yield выполнится после теста.
```

```python
def test_read_file(file_with_text):
    assert file_with_text.read_text(encoding="utf-8") == "hello"
```

## 10. Встроенная фикстура tmp_path

`tmp_path` создает временную папку для теста.

```python
def test_write_file(tmp_path):
    file_path = tmp_path / "result.txt"

    file_path.write_text("ok", encoding="utf-8")

    assert file_path.exists()
    assert file_path.read_text(encoding="utf-8") == "ok"
```

Это удобно, потому что тест не мусорит в проекте.

## 11. Проверка вывода в консоль: capsys

`capsys` позволяет проверить, что функция напечатала в консоль.

```python
def greet(name):
    print(f"Hello, {name}!")


def test_greet_output(capsys):
    greet("Max")

    captured = capsys.readouterr()

    assert captured.out == "Hello, Max!\n"
```

## 12. Monkeypatch

`monkeypatch` позволяет временно заменить переменную, функцию или переменную окружения.

```python
import os


def get_mode():
    return os.getenv("APP_MODE", "dev")


def test_get_mode_from_env(monkeypatch):
    monkeypatch.setenv("APP_MODE", "prod")

    assert get_mode() == "prod"
```

Пример замены функции:

```python
def get_current_user_id():
    return 42


def build_profile_url():
    user_id = get_current_user_id()
    return f"/users/{user_id}"


def test_build_profile_url(monkeypatch):
    monkeypatch.setattr(
        "app.profile.get_current_user_id",
        lambda: 100,
    )

    assert build_profile_url() == "/users/100"
```

## 13. Тестирование классов

```python
class Cart:
    def __init__(self):
        self.items = []

    def add(self, name, price):
        self.items.append({"name": name, "price": price})

    def total(self):
        return sum(item["price"] for item in self.items)
```

```python
def test_cart_total():
    cart = Cart()
    cart.add("Book", 500)
    cart.add("Pen", 50)

    assert cart.total() == 550
```

## 14. Организация тестов

Хорошие правила:

- один тест проверяет одну идею;
- имя теста объясняет ожидаемое поведение;
- тест не зависит от порядка запуска других тестов;
- тест не использует реальные внешние сервисы без необходимости;
- тесты должны быть быстрыми;
- фикстуры не стоит делать слишком магическими.

Плохое имя:

```python
def test_1():
    ...
```

Хорошее имя:

```python
def test_add_product_increases_cart_total():
    ...
```

## 15. Частые виды тестов

### Unit-тесты

Проверяют маленький кусок кода: функцию, метод, класс.

Пример: функция `calculate_discount`.

### Integration-тесты

Проверяют, как несколько частей работают вместе.

Пример: API-эндпоинт создает пользователя в тестовой базе данных.

### End-to-end-тесты

Проверяют сценарий целиком, почти как реальный пользователь.

Пример: пользователь открывает сайт, регистрируется и попадает в личный кабинет.

## 16. Что именно тестировать

В первую очередь стоит тестировать:

- бизнес-логику;
- расчеты;
- обработку ошибок;
- граничные случаи;
- права доступа;
- работу с разными типами входных данных;
- код, который часто ломается или часто меняется.

Примеры граничных случаев:

- пустой список;
- ноль;
- отрицательное число;
- очень большое число;
- `None`;
- пустая строка;
- дубликаты;
- неверный формат данных.

## 17. Пример хорошего набора тестов

Функция:

```python
def apply_discount(price, percent):
    if price < 0:
        raise ValueError("Price cannot be negative")
    if not 0 <= percent <= 100:
        raise ValueError("Percent must be between 0 and 100")

    return price - price * percent / 100
```

Тесты:

```python
import pytest


@pytest.mark.parametrize(
    "price, percent, expected",
    [
        (1000, 10, 900),
        (1000, 0, 1000),
        (1000, 100, 0),
    ],
)
def test_apply_discount(price, percent, expected):
    assert apply_discount(price, percent) == expected


def test_apply_discount_rejects_negative_price():
    with pytest.raises(ValueError, match="Price"):
        apply_discount(-100, 10)


@pytest.mark.parametrize("percent", [-1, 101])
def test_apply_discount_rejects_invalid_percent(percent):
    with pytest.raises(ValueError, match="Percent"):
        apply_discount(1000, percent)
```

## 18. Полезные команды

```bash
pytest
pytest -v
pytest -q
pytest tests/test_users.py
pytest tests/test_users.py::test_create_user
pytest -k "user"
pytest -x
pytest --maxfail=3
```

Что означают команды:

- `-v` - подробный вывод;
- `-q` - короткий вывод;
- `-k "user"` - запустить только тесты, в названии которых есть `user`;
- `-x` - остановиться после первой ошибки;
- `--maxfail=3` - остановиться после трех ошибок.

## 19. Мини-шпаргалка

```python
def test_something():
    assert result == expected
```

```python
with pytest.raises(ValueError):
    function()
```

```python
@pytest.mark.parametrize("value, expected", [(1, True), (2, False)])
def test_example(value, expected):
    assert function(value) == expected
```

```python
@pytest.fixture
def data():
    return {"name": "Test"}
```

## Задания для практики

### Задание 1. Калькулятор

Создай файл `calculator.py` с функциями:

```python
def add(a, b): ...
def subtract(a, b): ...
def multiply(a, b): ...
def divide(a, b): ...
```

Напиши тесты:

- сложение двух положительных чисел;
- сложение отрицательных чисел;
- вычитание;
- умножение на ноль;
- деление;
- деление на ноль должно выбрасывать `ValueError`.

### Задание 2. Проверка пароля

Создай функцию:

```python
def is_strong_password(password):
    ...
```

Пароль считается надежным, если:

- длина не меньше 8 символов;
- есть хотя бы одна цифра;
- есть хотя бы одна заглавная буква;
- есть хотя бы одна строчная буква.

Напиши тесты на:

- надежный пароль;
- слишком короткий пароль;
- пароль без цифр;
- пароль без заглавных букв;
- пароль без строчных букв;
- пустую строку.

Используй `parametrize`.

### Задание 3. Скидки

Создай функцию:

```python
def apply_discount(price, percent):
    ...
```

Требования:

- скидка 10% от 1000 возвращает 900;
- скидка 0% возвращает исходную цену;
- скидка 100% возвращает 0;
- отрицательная цена вызывает `ValueError`;
- процент меньше 0 или больше 100 вызывает `ValueError`.

Напиши тесты обычные и с `parametrize`.

### Задание 4. Корзина

Создай класс:

```python
class Cart:
    def add_item(self, name, price): ...
    def remove_item(self, name): ...
    def total(self): ...
    def count(self): ...
```

Напиши тесты:

- новая корзина пустая;
- после добавления товара `count()` увеличивается;
- `total()` считает сумму;
- удаление товара уменьшает количество;
- удаление несуществующего товара не ломает программу;
- товар с отрицательной ценой нельзя добавить.

Используй фикстуру `cart`.

### Задание 5. Работа с файлами

Создай функцию:

```python
def count_lines(file_path):
    ...
```

Она должна возвращать количество строк в файле.

Напиши тесты:

- файл с тремя строками;
- пустой файл;
- файл с одной строкой;
- несуществующий файл вызывает `FileNotFoundError`.

Используй встроенную фикстуру `tmp_path`.

### Задание 6. Консольный вывод

Создай функцию:

```python
def print_user_info(name, age):
    print(f"{name}: {age}")
```

Напиши тест, который проверяет вывод через `capsys`.

### Задание 7. Переменные окружения

Создай функцию:

```python
def get_database_url():
    ...
```

Она должна читать переменную окружения `DATABASE_URL`.
Если переменной нет, функция должна возвращать `"sqlite:///local.db"`.

Напиши тесты:

- когда `DATABASE_URL` задана;
- когда `DATABASE_URL` не задана.

Используй `monkeypatch`.

### Задание 8. Мини-проект: пользователи

Создай файл `users.py`.

Функции:

```python
def normalize_email(email):
    ...

def create_user(name, email):
    ...
```

Требования:

- `normalize_email` убирает пробелы по краям и приводит email к нижнему регистру;
- `create_user` возвращает словарь с полями `name`, `email`, `is_active`;
- если имя пустое, выбрасывается `ValueError`;
- если в email нет `@`, выбрасывается `ValueError`.

Напиши тесты:

- нормализация email;
- создание корректного пользователя;
- пустое имя;
- неверный email;
- проверка, что `is_active` равен `True`.

### Задание 9. Мини-проект: заметки

Создай класс:

```python
class Notes:
    def add(self, title, text): ...
    def get_all(self): ...
    def search(self, query): ...
    def delete(self, title): ...
```

Напиши тесты:

- добавление заметки;
- получение всех заметок;
- поиск по заголовку;
- поиск по тексту;
- удаление заметки;
- удаление отсутствующей заметки;
- пустой заголовок вызывает `ValueError`.

### Задание 10. Проверка чужого кода

Возьми любой небольшой модуль из своего проекта и напиши для него тесты.

План:

1. Найди функцию с понятным входом и выходом.
2. Напиши тест для обычного случая.
3. Напиши тест для граничного случая.
4. Напиши тест для ошибочного случая.
5. Запусти `pytest -v`.
6. Исправь код или тесты, если что-то упало.

## Как тренироваться

Хорошая последовательность:

1. Сначала напиши функцию.
2. Потом напиши 2-3 простых теста.
3. Запусти `pytest`.
4. Добавь тест на ошибку.
5. Добавь `parametrize`, если тесты похожи.
6. Добавь фикстуру, если в тестах повторяется подготовка данных.

Когда станет комфортно, попробуй писать наоборот: сначала тест, потом код. Это называется TDD.
