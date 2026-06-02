# Дополнительно

В этой главе собраны темы, которые часто встречаются после базовых запросов: индексы, представления, ограничения, каскадное удаление и полезные команды SQLite.

---

## Индексы

Индекс помогает базе быстрее находить строки. Его часто создают для столбцов, по которым часто ищут, сортируют или соединяют таблицы.

```sql
CREATE INDEX idx_user_name
ON User(Name);
```

Теперь запросы по `Name` могут выполняться быстрее.

```sql
SELECT *
FROM User
WHERE Name = 'Anton';
```

---

### Уникальный индекс

Уникальный индекс не позволяет хранить одинаковые значения.

```sql
CREATE UNIQUE INDEX idx_items_name
ON Items(Name);
```

После этого нельзя добавить два товара с одинаковым названием.

```sql
INSERT INTO Items (Name, Price)
VALUES ('Laptop', 1000);
```

Если `Laptop` уже есть, будет ошибка.

---

### Удаление индекса

```sql
DROP INDEX idx_user_name;
```

Индексы ускоряют чтение, но могут замедлять вставку и обновление данных, потому что базе нужно обновлять ещё и индекс.

---

## Представления - VIEW

`VIEW` - это сохраненный запрос. Он выглядит как таблица, но данные берутся из других таблиц.

Создадим представление с заказами:

```sql
CREATE VIEW user_orders AS
SELECT 
    Orders.Id AS OrderId,
    User.Name AS UserName,
    Items.Name AS ItemName,
    Items.Price AS Price
FROM Orders
JOIN User ON Orders.UserId = User.Id
JOIN Items ON Orders.ItemId = Items.Id;
```

Теперь можно обращаться к нему как к таблице.

```sql
SELECT *
FROM user_orders;
```

Можно добавлять фильтрацию.

```sql
SELECT *
FROM user_orders
WHERE Price > 600;
```

Удаление представления:

```sql
DROP VIEW user_orders;
```

---

## Ограничения внешних ключей

В SQLite внешние ключи нужно включать отдельно.

```sql
PRAGMA foreign_keys = ON;
```

Если этого не сделать, `FOREIGN KEY` может не проверяться.

Проверить состояние можно так:

```sql
PRAGMA foreign_keys;
```

Если ответ `1` - внешние ключи включены. Если `0` - выключены.

---

## Каскадное удаление

В таблице `Orders` можно настроить удаление заказов вместе с пользователем.

```sql
CREATE TABLE Orders (
    Id INTEGER PRIMARY KEY AUTOINCREMENT,
    UserId INTEGER,
    ItemId INTEGER,
    FOREIGN KEY (UserId) REFERENCES User(Id) ON DELETE CASCADE,
    FOREIGN KEY (ItemId) REFERENCES Items(Id) ON DELETE CASCADE
);
```

Теперь, если удалить пользователя, его заказы тоже удалятся.

```sql
DELETE FROM User
WHERE Id = 1;
```

Без `ON DELETE CASCADE` база либо запретила бы удаление, либо оставила бы заказы со ссылкой на несуществующего пользователя.

---

## CHECK

`CHECK` проверяет значение перед вставкой или обновлением.

```sql
CREATE TABLE Products (
    Id INTEGER PRIMARY KEY AUTOINCREMENT,
    Name TEXT NOT NULL,
    Price REAL CHECK (Price > 0)
);
```

Такой запрос выполнится:

```sql
INSERT INTO Products (Name, Price)
VALUES ('Phone', 500);
```

А такой даст ошибку:

```sql
INSERT INTO Products (Name, Price)
VALUES ('Broken item', -10);
```

---

## UNIQUE

`UNIQUE` запрещает одинаковые значения.

```sql
CREATE TABLE Accounts (
    Id INTEGER PRIMARY KEY AUTOINCREMENT,
    Email TEXT NOT NULL UNIQUE,
    Name TEXT NOT NULL
);
```

Теперь нельзя добавить два аккаунта с одним email.

```sql
INSERT INTO Accounts (Email, Name)
VALUES ('test@example.com', 'Anton');
```

---

## DEFAULT

`DEFAULT` задаёт значение по умолчанию.

```sql
CREATE TABLE Tasks (
    Id INTEGER PRIMARY KEY AUTOINCREMENT,
    Title TEXT NOT NULL,
    Status TEXT NOT NULL DEFAULT 'new'
);
```

Можно не указывать `Status`.

```sql
INSERT INTO Tasks (Title)
VALUES ('Изучить SQLite');
```

В столбце `Status` будет `new`.

---

## EXPLAIN QUERY PLAN

Команда показывает, как SQLite собирается выполнять запрос.

```sql
EXPLAIN QUERY PLAN
SELECT *
FROM User
WHERE Name = 'Anton';
```

Это полезно, когда нужно понять, используется ли индекс.

Если индекс есть, в результате можно увидеть что-то вроде `USING INDEX`.

---

## Полезные команды sqlite3

Эти команды выполняются в консольной программе `sqlite3`.

Открыть базу:

```bash
sqlite3 example.db
```

Показать таблицы:

```sql
.tables
```

Показать структуру таблицы:

```sql
.schema User
```

Красивый вывод:

```sql
.mode column
.headers on
```

Выполнить SQL-файл:

```sql
.read file.sql
```

Выйти:

```sql
.exit
```

---

## Импорт CSV

Можно импортировать данные из CSV-файла.

```sql
.mode csv
.import users.csv User
```

Важно, чтобы столбцы в CSV совпадали со столбцами таблицы.

---

## Экспорт результата

Можно сохранить результат запроса в файл.

```sql
.headers on
.mode csv
.output result.csv
SELECT *
FROM User;
.output stdout
```

После этого результат запроса будет записан в `result.csv`.

