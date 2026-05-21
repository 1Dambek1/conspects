[[Виды связей в SQL]]
Связь one to many в SQL используется для моделирования отношений между двумя таблицами, где одна запись в первой таблице может быть связана с несколькими записями во второй таблице. Рассмотрим пример, когда у нас есть таблица пользователей и таблица заказов. Один пользователь может сделать несколько заказов, но каждый заказ относится к одному конкретному пользователю.

Создадим схему базы данных:

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

В таблице `users` мы сохраняем информацию о пользователях, включая их имя пользователя и электронную почту. В таблице `orders` мы сохраняем информацию о заказах, включая идентификатор пользователя, который сделал этот заказ, идентификатор продукта и количество товара.

### Пример добавления данных

Для начала добавим несколько пользователей и заказов:

```sql
INSERT INTO users (username, email) VALUES ('john', '<EMAIL>'), ('mary', '<EMAIL>');

INSERT INTO orders (user_id, product_id, quantity) VALUES (1, 100, 2), (1, 101, 1), (2, 100, 1);
```

### Пример обновления данных

Допустим, пользователь с ID 1 изменил свой e-mail:

```sql
UPDATE users SET email = '<EMAIL>' WHERE id = 1;
```

### Пример удаления данных

Удалим все заказы пользователя с ID 1:

```sql
DELETE FROM orders WHERE user_id = 1;
```

### Пример выборки данных

Получим список всех пользователей и количество сделанных ими заказов:

```sql
SELECT u.id, u.username, u.email, COUNT(*) as order_count
FROM users AS u
LEFT JOIN orders AS o ON u.id = o.user_id
GROUP BY u.id;
```

Этот запрос выбирает всех пользователей, подсчитывает количество заказов, связанных с каждым из них, и группирует результаты по идентификатору пользователя.