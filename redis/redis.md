[[json]]
## 🔹 Добавление данных в Redis (Python)

### Строка (`String`)

```python
import redis
r = redis.Redis()

r.set('key', 'value')  # Добавить
r.setex('temp_key', 10, 'value')  # Добавить с удалением через 10 секунд
```

### Хэш (`Hash`)

```python
r.hset('user:1', 'name', 'Denis')
r.hset('user:1', mapping={'age': 15, 'city': 'Irkutsk'})
```

### Список (`List`)

```python
r.rpush('queue', 'task1', 'task2')  # Добавление вправо
r.lpush('queue', 'task0')          # Добавление влево
```

### Множество (`Set`)

```python
r.sadd('tags', 'python', 'redis')  # Добавление
```

---

## 🔹 Удаление данных

```python
r.delete('key')  # Удалить ключ (любой тип)

r.hdel('user:1', 'age')  # Удалить поле из хэша
r.lpop('queue')          # Удалить первый элемент списка
r.srem('tags', 'redis')  # Удалить элемент из множества
```

---

## 🔹 Удаление через время (TTL)

```python
r.set('temp_key', 'value')
r.expire('temp_key', 30)  # Автоматическое удаление через 30 сек

r.ttl('temp_key')  # Узнать оставшееся время жизни

r.persist('temp_key')  # Убрать TTL и сохранить навсегда
```

---

## 🔹 Запуск Redis через Docker

### ✅ Вариант 1: через Docker CLI

```bash
docker run --name my-redis -p 6379:6379 redis
```

---

## 🔹 Запуск Redis через Docker Compose

### `docker-compose.yml`

```yaml
version: '3'
services:
  redis:
    image: redis:latest
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data


```


---

## 🔹 Использование JSON с Redis в Python

Redis **не поддерживает вложенные структуры** (например, словари внутри словарей) напрямую. Поэтому для хранения сложных объектов (например, словарей, списков) часто используется модуль `json` из Python.

---

### 🔸 Зачем использовать `json` с Redis?

Redis работает с **байтами и строками**, но мы часто хотим сохранять:

- Словари: `{'name': 'Denis', 'age': 15}`
    
- Списки: `['task1', 'task2']`
    
- Структуры из Python
    

В таких случаях мы сериализуем структуру в строку с помощью `json.dumps()` и десериализуем обратно с `json.loads()`.

---

### 🔸 Пример: сохранение словаря в Redis

```python
import redis
import json

r = redis.Redis()

# Сложный Python-объект
user_data = {'name': 'Denis', 'age': 15, 'city': 'Irkutsk'}

# Сохраняем как строку JSON
r.set('user:1', json.dumps(user_data))

# Получаем и декодируем обратно
data = json.loads(r.get('user:1'))
print(data['city'])  # Irkutsk

```

### 🔸 Отличие от `hset`

- `hset` работает только с **простыми парами** ключ-значение внутри Redis-хэша.
    
- `json.dumps()` позволяет сохранять **любые вложенные структуры** и использовать Redis как кеш JSON-объектов.