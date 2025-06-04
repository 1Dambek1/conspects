Вот дополненный конспект с **интеграцией Flower** — веб-интерфейса мониторинга для Celery.

---

## 🔧 Добавляем **Flower** в стек `Celery + Docker`

### 🔹 Что такое Flower?

**[Flower](https://flower.readthedocs.io/)** — это **реалтайм мониторинг и управление Celery задачами** через веб-интерфейс.

Позволяет:

- отслеживать статус задач (`PENDING`, `STARTED`, `SUCCESS`, `FAILURE`)
    
- перезапускать задачи
    
- смотреть логи
    
- контролировать очередь задач
    

---

### 🔸 1. Добавим Flower в `docker-compose.yml`

```yaml
version: '3.9'
services:
  api:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - redis
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  worker:
    build: .
    depends_on:
      - redis
    command: celery -A app.worker worker --loglevel=info

  redis:

    image: redis:alpine

    container_name: redis

    ports:

      - 6379:6379

  flower:
    image: mher/flower:0.9.7
    command: flower --broker=redis://redis:6379/0 --port=5555
    ports:
      - "5555:5555"
    depends_on:
      - redis
```

---

### 🔸 2. Убедись, что Celery и Redis используют одинаковый брокер:

```python
# app/worker.py
celery_app = Celery(
    "worker",
    broker="redis://redis:6379/0",
    backend="redis://redis:6379/0"
)
```

---

### 🔸 3. Обнови `requirements.txt` (если нужно использовать Flower локально):

```txt
fastapi
uvicorn
celery
redis
flower
```

---

### 🔸 4. Запуск

```bash
docker compose up --build
```

---

### 🔸 5. Интерфейс Flower

После запуска открой в браузере:

```
http://localhost:5555
```

Ты увидишь:

- Список задач
    
- Очереди
    
- Статус воркеров
    
- Историю задач
    



