# Встроенные функции

В SQLite есть много встроенных функций. Они помогают обрабатывать текст, числа, даты и `NULL` прямо внутри запроса.

---

## Текстовые функции

### LENGTH

`LENGTH` возвращает длину строки.

```sql
SELECT Name, LENGTH(Name) AS "Длина имени"
FROM User;
```

Можно использовать в `WHERE`.

```sql
SELECT *
FROM User
WHERE LENGTH(Name) > 4;
```

---

### UPPER и LOWER

`UPPER` переводит строку в верхний регистр, а `LOWER` - в нижний.

```sql
SELECT 
    Name,
    UPPER(Name) AS "Большие буквы",
    LOWER(Name) AS "Маленькие буквы"
FROM User;
```

Это удобно, когда нужно сравнивать текст без учета регистра.

```sql
SELECT *
FROM User
WHERE LOWER(Name) = 'anton';
```

---

### TRIM

`TRIM` убирает пробелы в начале и конце строки.

```sql
SELECT TRIM('   Anton   ') AS name;
```

Есть ещё `LTRIM` и `RTRIM`.

```sql
SELECT 
    LTRIM('   Anton') AS left_trim,
    RTRIM('Anton   ') AS right_trim;
```

---

### SUBSTR

`SUBSTR` возвращает часть строки.

```sql
SELECT SUBSTR(Name, 1, 3) AS "Первые 3 символа"
FROM User;
```

Первый аргумент - строка, второй - позиция, с которой начинаем, третий - сколько символов взять.

```sql
SELECT SUBSTR('SQLite', 4, 3) AS result;
```

Ответ будет `ite`.

---

### REPLACE

`REPLACE` заменяет одну часть строки на другую.

```sql
SELECT REPLACE('Hello SQL', 'SQL', 'SQLite') AS result;
```

Можно использовать для данных из таблицы.

```sql
SELECT REPLACE(Name, 'Anton', 'Антон') AS "Новое имя"
FROM User;
```

---

### INSTR

`INSTR` ищет подстроку и возвращает её позицию.

```sql
SELECT INSTR('SQLite tutorial', 'tutorial') AS position;
```

Если подстрока не найдена, результат будет `0`.

```sql
SELECT *
FROM User
WHERE INSTR(Name, 'A') > 0;
```

---

## Числовые функции

### ROUND

`ROUND` округляет число.

```sql
SELECT ROUND(10.567, 2) AS result;
```

Ответ будет `10.57`.

```sql
SELECT 
    Name,
    ROUND(Price, 1) AS "Цена"
FROM Items;
```

---

### ABS

`ABS` возвращает модуль числа.

```sql
SELECT ABS(-15) AS result;
```

---

### RANDOM

`RANDOM` возвращает случайное число.

```sql
SELECT RANDOM() AS random_number;
```

Если нужно выбрать случайную запись:

```sql
SELECT *
FROM User
ORDER BY RANDOM()
LIMIT 1;
```

---

## Функции для NULL

### IFNULL

`IFNULL` заменяет `NULL` на другое значение.

```sql
SELECT 
    User.Name,
    IFNULL(SUM(Items.Price), 0) AS "Сумма покупок"
FROM User
LEFT JOIN Orders ON User.Id = Orders.UserId
LEFT JOIN Items ON Orders.ItemId = Items.Id
GROUP BY User.Id;
```

Если у пользователя нет заказов, `SUM(Items.Price)` вернёт `NULL`. Через `IFNULL` мы заменяем это на `0`.

---

### COALESCE

`COALESCE` возвращает первое значение, которое не равно `NULL`.

```sql
SELECT COALESCE(NULL, NULL, 'SQLite', 'SQL') AS result;
```

Ответ будет `SQLite`.

Пример с таблицей:

```sql
SELECT 
    Name,
    COALESCE(Price, 0) AS price
FROM Items;
```

---

### NULLIF

`NULLIF` возвращает `NULL`, если два значения равны.

```sql
SELECT NULLIF(10, 10) AS result;
```

Ответ будет `NULL`.

Если значения разные, вернётся первое значение.

```sql
SELECT NULLIF(10, 5) AS result;
```

Ответ будет `10`.

---

## Функции даты и времени

В SQLite нет отдельного типа `DATE`, поэтому даты часто хранятся как текст в формате `YYYY-MM-DD`.

### DATE

```sql
SELECT DATE('now') AS today;
```

Можно прибавлять или вычитать дни.

```sql
SELECT DATE('now', '+7 days') AS next_week;
```

```sql
SELECT DATE('now', '-1 month') AS previous_month;
```

---

### TIME

```sql
SELECT TIME('now') AS current_time;
```

---

### DATETIME

```sql
SELECT DATETIME('now') AS current_datetime;
```

---

### STRFTIME

`STRFTIME` позволяет получить часть даты.

```sql
SELECT STRFTIME('%Y', '2024-05-10') AS year;
```

Частые варианты:

- `%Y` - год
- `%m` - месяц
- `%d` - день
- `%H` - часы
- `%M` - минуты
- `%S` - секунды

Пример: получить год из даты.

```sql
SELECT STRFTIME('%Y', '2023-05-10') AS year;
```

---

## Условная логика - CASE

`CASE` позволяет делать условия прямо в `SELECT`.

```sql
SELECT 
    Name,
    Age,
    CASE
        WHEN Age < 18 THEN 'Ребенок'
        WHEN Age < 30 THEN 'Молодой'
        ELSE 'Взрослый'
    END AS "Группа"
FROM User;
```

Можно использовать `CASE` вместе с агрегацией.

```sql
SELECT 
    COUNT(CASE WHEN Price >= 800 THEN 1 END) AS expensive_count,
    COUNT(CASE WHEN Price < 800 THEN 1 END) AS cheap_count
FROM Items;
```

---

## Функции вместе

Функции можно комбинировать.

```sql
SELECT 
    UPPER(TRIM(Name)) AS clean_name,
    LENGTH(TRIM(Name)) AS name_length
FROM User;
```

Пример с заказами:

```sql
SELECT 
    User.Name AS "Пользователь",
    IFNULL(ROUND(SUM(Items.Price), 2), 0) AS "Сумма покупок"
FROM User
LEFT JOIN Orders ON User.Id = Orders.UserId
LEFT JOIN Items ON Orders.ItemId = Items.Id
GROUP BY User.Id
ORDER BY "Сумма покупок" DESC;
```
