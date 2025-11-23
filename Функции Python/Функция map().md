Функция `map()` применяет указанную функцию к каждому элементу итерируемого объекта (списка, кортежа и т.д.) и возвращает итератор с результатами.
```python
map(function, iterable, ...)
```
- **function** - функция, которую нужно применить
- **iterable** - итерируемый объект (список, кортеж, строка и т.д.)
- **...** - можно передать несколько итерируемых объектов

## Базовые примеры

### Пример 1: Преобразование чисел
```python
# Преобразуем список чисел в их квадраты
numbers = [1, 2, 3, 4, 5]

def square(x):
    return x ** 2

result = map(square, numbers)
print(list(result))  # [1, 4, 9, 16, 25]
```
### Пример 2: Использование lambda-функций
```python
# Удваиваем каждое число с помощью lambda
numbers = [1, 2, 3, 4, 5]
result = map(lambda x: x * 2, numbers)
print(list(result))  # [2, 4, 6, 8, 10]
```
### Пример 3: Преобразование строк
```python
# Преобразуем строки в верхний регистр
words = ['hello', 'world', 'python']
result = map(str.upper, words)
print(list(result))  # ['HELLO', 'WORLD', 'PYTHON']
```
## Работа с несколькими итерируемыми объектами

### Пример 4: Сложение элементов из двух списков
```python
list1 = [1, 2, 3]
list2 = [4, 5, 6]

result = map(lambda x, y: x + y, list1, list2)
print(list(result))  # [5, 7, 9]
```
### Пример 5: Объединение строк из разных списков
```python
names = ['John', 'Jane', 'Bob']
titles = ['Mr.', 'Ms.', 'Mr.']

result = map(lambda name, title: f"{title} {name}", names, titles)
print(list(result))  # ['Mr. John', 'Ms. Jane', 'Mr. Bob']
```
## Практические примеры

### Пример 6: Обработка данных
```python
# Преобразование температур из Цельсия в Фаренгейт
celsius_temps = [0, 20, 30, 40]

def to_fahrenheit(c):
    return (c * 9/5) + 32

fahrenheit_temps = list(map(to_fahrenheit, celsius_temps))
print(fahrenheit_temps)  # [32.0, 68.0, 86.0, 104.0]
```
### Пример 7: Очистка данных
```python
# Удаление пробелов и преобразование к числам
data = [' 10 ', ' 20.5 ', ' 30 ']

cleaned_data = list(map(float, map(str.strip, data)))
print(cleaned_data)  # [10.0, 20.5, 30.0]
```
### Пример 8: Работа с вложенными структурами
```python
# Извлечение конкретных полей из списка словарей
students = [
    {'name': 'Alice', 'age': 20, 'grade': 'A'},
    {'name': 'Bob', 'age': 22, 'grade': 'B'},
    {'name': 'Charlie', 'age': 21, 'grade': 'A'}
]

# Получаем только имена
names = list(map(lambda student: student['name'], students))
print(names)  # ['Alice', 'Bob', 'Charlie']

# Получаем возраст
ages = list(map(lambda student: student['age'], students))
print(ages)  # [20, 22, 21]
```
## Особенности работы с `map()`

### Ленивые вычисления
```python
numbers = [1, 2, 3, 4, 5]
result = map(lambda x: x * 2, numbers)

print(result)  # <map object at 0x...> (итератор)
print(list(result))  # [2, 4, 6, 8, 10] (преобразуем в список)

# Итератор можно использовать только один раз
print(list(result))  # [] (пустой список, так как итератор исчерпан)
```
### Сравнение с циклом for
```python
# Эквивалент с использованием цикла for
numbers = [1, 2, 3, 4, 5]
result = []

for num in numbers:
    result.append(num * 2)

print(result)  # [2, 4, 6, 8, 10]
```
### Сравнение с list comprehension
```python
numbers = [1, 2, 3, 4, 5]

# С map()
result_map = list(map(lambda x: x * 2, numbers))

# С list comprehension
result_comp = [x * 2 for x in numbers]

print(result_map)   # [2, 4, 6, 8, 10]
print(result_comp)  # [2, 4, 6, 8, 10]
```
## Продвинутые примеры

### Пример 9: Цепочка преобразований
```python
# Несколько преобразований в цепочке
numbers = [1, 2, 3, 4, 5]

# Умножаем на 2, затем возводим в квадрат
result = map(lambda x: x ** 2, map(lambda x: x * 2, numbers))
print(list(result))  # [4, 16, 36, 64, 100]
```
### Пример 10: Работа с разными типами данных
```python
# Преобразование типов данных
mixed_data = ['1', '2.5', '3', '4.7']

# Преобразуем в целые числа, где возможно
def safe_int_convert(x):
    try:
        return int(x)
    except ValueError:
        return float(x)

result = list(map(safe_int_convert, mixed_data))
print(result)  # [1, 2.5, 3, 4.7]
```
## Когда использовать `map()`

**Преимущества:**

- Более читаемый код для простых преобразований
- Ленивые вычисления (экономия памяти)
- Функциональный стиль программирования

**Когда лучше использовать list comprehension:**

- Для сложных преобразований с условиями
- Когда нужна лучшая читаемость
- Для фильтрации и преобразования одновременно

```python
# Сложное преобразование - лучше использовать list comprehension
numbers = [1, 2, 3, 4, 5, 6]
result = [x * 2 if x % 2 == 0 else x for x in numbers]
print(result)  # [1, 4, 3, 8, 5, 12]
```
Функция `map()` - это мощный инструмент для функционального программирования в Python, который особенно полезен для простых преобразований данных и работы с несколькими итерируемыми объектами одновременно.