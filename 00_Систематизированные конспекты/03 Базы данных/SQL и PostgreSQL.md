# SQL и PostgreSQL

Связанные темы: [[SQLAlchemy и связи]], [[Alembic, Redis и подгрузка данных]], [[FastAPI - базовый конспект|FastAPI]].

## База данных

База данных хранит структурированные данные. В backend чаще всего используются реляционные БД: PostgreSQL, MySQL, SQLite.

SQL - язык запросов к реляционным базам данных.

## Основные операции

Создание таблицы:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

Добавление:

```sql
INSERT INTO users (name, email)
VALUES ('Anna', 'anna@example.com');
```

Выборка:

```sql
SELECT id, name, email
FROM users
WHERE name ILIKE '%ан%';
```

Обновление:

```sql
UPDATE users
SET name = 'Ivan'
WHERE id = 1;
```

Удаление:

```sql
DELETE FROM users
WHERE id = 1;
```

## Сортировка и ограничение

```sql
SELECT *
FROM products
ORDER BY price DESC
LIMIT 5;
```

## JOIN

JOIN связывает данные из нескольких таблиц.

```sql
SELECT orders.id, users.name, orders.total
FROM orders
JOIN users ON users.id = orders.user_id
WHERE users.id = 1;
```

## Агрегатные функции

- `COUNT()` - количество строк.
- `SUM()` - сумма.
- `AVG()` - среднее.
- `MIN()` - минимум.
- `MAX()` - максимум.

```sql
SELECT user_id, COUNT(*) AS orders_count, SUM(total) AS total_sum
FROM orders
GROUP BY user_id;
```

## Связи в SQL

One-to-one:

```text
users 1 -> 1 profiles
```

One-to-many:

```text
users 1 -> many orders
```

Many-to-many:

```text
students many -> many courses
```

Many-to-many обычно реализуется через промежуточную таблицу.

```sql
CREATE TABLE users_chats (
    user_id INT REFERENCES users(id),
    chat_id INT REFERENCES chats(id),
    PRIMARY KEY (user_id, chat_id)
);
```

## PostgreSQL из Python

Через `psycopg2`:

```bash
pip install psycopg2-binary
```

```python
import psycopg2

conn = psycopg2.connect(
    dbname="app",
    user="postgres",
    password="postgres",
    host="localhost",
    port=5432,
)

cursor = conn.cursor()
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
conn.close()
```

В реальных FastAPI-проектах чаще используют SQLAlchemy, чтобы не писать все SQL-запросы вручную.
