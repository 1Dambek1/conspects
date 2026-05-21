[[Виды связей в SQL]]
Связь one to one в SQL используется для моделирования отношений между двумя таблицами, где каждая запись в одной таблице соответствует только одной записи в другой таблице и наоборот. Например, представим, что у нас есть две таблицы: `users` и `user_address`. Каждый пользователь может иметь только одну адресную запись, и каждая адресная запись принадлежит только одному пользователю.

Пример создания схемы базы данных:

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL
);

CREATE TABLE user_address (
    id INT PRIMARY KEY AUTO_INCREMENT,
    address VARCHAR(255) NOT NULL,
    city VARCHAR(50),
    state CHAR(2),
    zipcode INT,
    FOREIGN KEY (id) REFERENCES users(id) ON DELETE CASCADE
);
```

В данном примере мы создаем две таблицы: `users` и `user_address`. В таблице `users` хранятся данные о пользователях, включая их имена и идентификатор (`id`). Таблица `user_address` содержит информацию об адресе каждого пользователя, включая город, штат и почтовый индекс. Мы также добавляем внешний ключ (`FOREIGN KEY`) для связи этих двух таблиц по полю `id`, которое является первичным ключом в обеих таблицах.

Теперь рассмотрим примеры использования этой связи:

### Добавление записей

```sql
INSERT INTO users (first_name, last_name) VALUES ('John', 'Doe');
INSERT INTO user_address (id, address, city, state, zipcode) VALUES (1, '123 Main St', 'New York', 'NY', 10001);
```

### Обновление записей

```sql
UPDATE users SET first_name = 'Jane' WHERE id = 1;
UPDATE user_address SET address = '123 New St' WHERE id = 1;
```

### Удаление записей

```sql
DELETE FROM users WHERE id = 1;
-- Так как связь установлена с ON DELETE CASCADE, то будет удалена соответствующая запись в таблице user_address
```

### Получение данных

```sql
SELECT u.id, u.first_name, u.last_name, a.address, a.city, a.state, a.zipcode
FROM users AS u
LEFT JOIN user_address AS a ON u.id = a.id;
```

Этот запрос объединяет данные из обеих таблиц и позволяет получить полную информацию о каждом пользователе вместе с его адресом.