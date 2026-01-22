# Сложные запросы

В этойглаве мы будем рассматривать как запросом выводить данные сразу из нескольких таблиц и не только.

Для начала, создадим три таблицы

```sql
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS user;
DROP TABLE IF EXISTS Items;

CREATE TABLE User (
    Id INTEGER PRIMARY KEY AUTOINCREMENT,
    Name TEXT NOT NULL
);

CREATE TABLE Items (
    Id INTEGER PRIMARY KEY AUTOINCREMENT,
    Name TEXT NOT NULL,
    Price DECIMAL(10, 2)
);

CREATE TABLE Orders (
    Id INTEGER PRIMARY KEY AUTOINCREMENT,
    UserId INTEGER,
    ItemId INTEGER,
    FOREIGN KEY (UserId) REFERENCES User(Id) ON DELETE CASCADE,
    FOREIGN KEY (ItemId) REFERENCES Items(Id) ON DELETE CASCADE
);
```

Теперь, заполним их данными

```sql  
INSERT INTO User (Name) VALUES
    ('Anton'),
    ('Jane'),
    ('Bob'),
    ('Tom');

INSERT INTO Items (Name, Price) VALUES
    ('Laptop', 1000.00),
    ('Phone', 500.00),
    ('Tablet', 800.00);

INSERT INTO Orders (UserId, ItemId) VALUES
    (1, 1),
    (2, 2),
    (3, 3),
    (1, 2),
    (2, 1),
    (3, 3);
``` 

---


## Неявное соединение таблиц

```sql
SELECT * FROM orders, user;
```

Если вывести так, то в результате этого нам выведется кажа из записей - не понятно и не удобно, поэтому можно использовать `WHERE`

```sql
SELECT * FROM orders, user
WHERE orders.UserId = user.Id
ORDER BY user.Name;
```

Так понятно, но можно вывести ещё удобнее

```sql
	
SELECT orders.Id as "Номер заказа", items.Name as "Товар", items.Price as "Цена", User.Name as "Имя пользователя" 
FROM orders, user, items
WHERE orders.UserId = user.Id AND orders.ItemId = items.Id
ORDER BY user.Name;
```

Это всё хорошо, но давай усовершенствуем, ведь нам нужны структурированные данные.


---

## Соенинение при помощи `JOIN`

`JOIN` служит для установления связи между таблицами, чтобы вывести данные из нескольких таблиц вместе.

Вот его структура 

```sql
SELECT столбцы
FROM таблица1
    [INNER] JOIN таблица2
    ON условие1
    [[INNER] JOIN таблица3
    ON условие2]
```

Рассмотрим на реальном примере 
  a                                         
```sql
SELECT orders.Id as "Номер заказа", items.Name as "Товар", items.Price as "Цена", User.Name as "Имя пользователя" 
FROM orders
    JOIN user
    ON orders.UserId = user.Id
    JOIN items
    ON orders.ItemId = items.Id
ORDER BY user.Name;
```

Этот запрос выведет то же самое, что и предыдущий.

Так же можно можно добавлять условие, например вывести товары, стоимость которых больше 600

```sql
SELECT orders.Id as "Номер заказа", items.Name as "Товар", items.Price as "Цена", User.Name as "Имя пользователя" 
FROM orders
    JOIN user
    ON orders.UserId = user.Id
    JOIN items
    ON orders.ItemId = items.Id AND items.Price > 600
ORDER BY user.Name;
```

`JOIN` позволяет делать сложные запросы, например 
```sql
SELECT 
    user.Name as "Имя пользователя",
    GROUP_CONCAT(items.Name, ', ') as "Товар",
    SUM(items.Price) as "Цена"
FROM 
    orders
JOIN 
    user ON orders.UserId = user.Id
JOIN 
    items ON orders.ItemId = items.Id
GROUP BY 
    user.Id
ORDER BY 
    user.Name;
```

Этот запрос выведет сумму все покупок у пользователей. 

Так же можно использовать `LEFT JOIN`, который позволяет выбрать NULL из присоединяемой таблицы

Добавим пользователя без заказов
```sql
INSERT INTO User (Name) VALUES ('Anna');
```

выполним запрос
```sql
SELECT 
    User.Id,
    User.Name as "Имя пользователя",
    COUNT(orders.Id) as "Количество заказов"
FROM User
LEFT JOIN orders ON User.Id = orders.UserId
GROUP BY User.Id
```
тут нам выведет Anna, где будет 0 заказов


```sql
SELECT 
    User.Name as "Имя пользователя",
    IFNULL(SUM(items.Price), 0) as "Общая сумма покупок",
    COUNT(DISTINCT orders.Id) as "Количество заказов",
    GROUP_CONCAT(DISTINCT items.Name) as "Купленные товары"
FROM User
LEFT JOIN orders ON User.Id = orders.UserId
LEFT JOIN items ON orders.ItemId = items.Id
GROUP BY User.Id
ORDER BY "Общая сумма покупок" DESC;
```

тут нам выведет строчку с пользователем Anna, несмотря на то, что у неё нет купленных товаров