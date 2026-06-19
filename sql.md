# PostgreSQL с нуля до уверенного Junior+

Ниже — подробный учебник по PostgreSQL: от самых базовых понятий до уровня, на котором ты сможешь уверенно работать с базой данных на junior-позиции.

---

# 0. Что такое PostgreSQL

## PostgreSQL — это СУБД

**СУБД** — система управления базами данных.

PostgreSQL позволяет:

- хранить данные;
- искать данные;
- изменять данные;
- удалять данные;
- связывать данные между собой;
- защищать данные;
- делать резервные копии;
- обрабатывать большие объёмы информации.

Примеры данных:

- пользователи сайта;
- товары интернет-магазина;
- заказы;
- комментарии;
- платежи;
- логи;
- настройки.

---

# 1. Что такое база данных

## Простое объяснение

База данных — это организованное хранилище информации.

Например, у интернет-магазина есть:

- пользователи;
- товары;
- заказы;
- оплаты;
- доставки.

В PostgreSQL это обычно хранится в виде **таблиц**.

---

# 2. Таблицы

## Таблица похожа на Excel

Например, таблица `users`:

| id | name | email | age |
|---|---|---|---|
| 1 | Ivan | ivan@mail.com | 25 |
| 2 | Anna | anna@mail.com | 30 |

У таблицы есть:

- **столбцы** — `id`, `name`, `email`, `age`;
- **строки** — конкретные записи;
- **типы данных** — число, текст, дата и т.д.

---

# 3. Что такое SQL

**SQL** — язык, с помощью которого мы общаемся с базой данных.

Примеры:

```sql
SELECT * FROM users;
```

Получить всех пользователей.

```sql
INSERT INTO users (name, email)
VALUES ('Ivan', 'ivan@mail.com');
```

Добавить пользователя.

```sql
UPDATE users
SET age = 26
WHERE id = 1;
```

Изменить пользователя.

```sql
DELETE FROM users
WHERE id = 1;
```

Удалить пользователя.

---

# 4. Установка PostgreSQL

## Вариант 1: Через официальный сайт

Скачать PostgreSQL можно здесь:

```text
https://www.postgresql.org/download/
```

После установки обычно появятся:

- PostgreSQL Server;
- pgAdmin — графический интерфейс;
- psql — консольный клиент.

---

## Вариант 2: Через Docker

Если знаешь Docker, можно запустить так:

```bash
docker run --name postgres-db \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=testdb \
  -p 5432:5432 \
  -d postgres:16
```

Подключиться:

```bash
docker exec -it postgres-db psql -U postgres -d testdb
```

---

# 5. Основные термины PostgreSQL

## Database

**Database** — база данных.

Например:

```sql
CREATE DATABASE shop;
```

---

## Schema

**Schema** — логическое пространство внутри базы данных.

По умолчанию используется схема:

```text
public
```

Пример полного имени таблицы:

```sql
public.users
```

---

## Table

Таблица:

```sql
CREATE TABLE users (
    id integer,
    name text
);
```

---

## Row

Строка таблицы.

```text
1 | Ivan
2 | Anna
```

---

## Column

Столбец таблицы.

```text
id, name, email
```

---

## Primary Key

**Primary key** — главный уникальный идентификатор строки.

Например:

```sql
id SERIAL PRIMARY KEY
```

Это значит:

- `id` обязателен;
- `id` уникален;
- по нему удобно находить строку.

---

## Foreign Key

**Foreign key** — внешний ключ, связь между таблицами.

Например:

```sql
user_id integer REFERENCES users(id)
```

Это значит:

- в таблице заказов есть `user_id`;
- он ссылается на пользователя из таблицы `users`.

---

# 6. Подключение к PostgreSQL через psql

Команда:

```bash
psql -U postgres -d testdb
```

Где:

- `-U postgres` — пользователь;
- `-d testdb` — база данных.

---

## Полезные команды psql

Посмотреть базы данных:

```sql
\l
```

Посмотреть таблицы:

```sql
\dt
```

Посмотреть структуру таблицы:

```sql
\d users
```

Подключиться к базе:

```sql
\c shop
```

Выйти:

```sql
\q
```

Очистить экран:

```sql
\! clear
```

---

# 7. Создание базы данных

```sql
CREATE DATABASE shop;
```

Подключиться к ней:

```sql
\c shop
```

Удалить базу:

```sql
DROP DATABASE shop;
```

Важно: нельзя удалить базу, к которой ты сейчас подключён.

---

# 8. Типы данных PostgreSQL

Тип данных говорит базе, какие значения можно хранить в столбце.

---

## Числовые типы

### integer

Целое число.

```sql
age integer
```

Примеры:

```text
10
25
100
```

---

### bigint

Большое целое число.

```sql
views_count bigint
```

Используется, когда значений может быть очень много.

---

### numeric

Точное число, часто для денег.

```sql
price numeric(10, 2)
```

Это значит:

- всего до 10 цифр;
- 2 цифры после запятой.

Пример:

```text
99999999.99
```

---

### real / double precision

Числа с плавающей точкой.

Для денег лучше **не использовать**, потому что возможны погрешности.

---

## Текстовые типы

### text

Текст любой длины.

```sql
description text
```

---

### varchar(n)

Строка длиной максимум `n`.

```sql
email varchar(255)
```

---

### char(n)

Строка фиксированной длины.

Используется редко.

---

## Логический тип

```sql
is_active boolean
```

Значения:

```text
true
false
```

---

## Дата и время

### date

Только дата.

```sql
birthday date
```

Пример:

```text
2000-05-15
```

---

### time

Только время.

```sql
start_time time
```

---

### timestamp

Дата и время без часового пояса.

```sql
created_at timestamp
```

---

### timestamptz

Дата и время с часовым поясом.

```sql
created_at timestamptz
```

В реальных проектах часто лучше использовать именно `timestamptz`.

---

## UUID

Уникальный идентификатор.

```sql
id uuid
```

Пример:

```text
550e8400-e29b-41d4-a716-446655440000
```

Для генерации UUID часто используют расширение:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

И потом:

```sql
id uuid PRIMARY KEY DEFAULT uuid_generate_v4()
```

---

## JSON / JSONB

PostgreSQL умеет хранить JSON.

Лучше чаще использовать `jsonb`, потому что он оптимизирован для поиска.

```sql
data jsonb
```

Пример:

```json
{
  "color": "red",
  "size": "XL"
}
```

---

# 9. Создание таблицы

Создадим таблицу пользователей:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name text NOT NULL,
    email text NOT NULL UNIQUE,
    age integer,
    is_active boolean DEFAULT true,
    created_at timestamptz DEFAULT now()
);
```

Разберём подробно.

---

## id SERIAL PRIMARY KEY

```sql
id SERIAL PRIMARY KEY
```

`SERIAL` — автоматическое увеличение числа.

Если добавить пользователя без `id`, PostgreSQL сам поставит:

```text
1, потом 2, потом 3...
```

`PRIMARY KEY` — главный идентификатор.

---

## NOT NULL

```sql
name text NOT NULL
```

Это значит: поле обязательно.

Нельзя добавить пользователя без имени.

---

## UNIQUE

```sql
email text NOT NULL UNIQUE
```

Это значит: email должен быть уникальным.

Нельзя создать двух пользователей с одинаковым email.

---

## DEFAULT

```sql
is_active boolean DEFAULT true
```

Если при вставке не указать значение, будет `true`.

---

## now()

```sql
created_at timestamptz DEFAULT now()
```

При создании записи автоматически сохранится текущее время.

---

# 10. Удаление таблицы

```sql
DROP TABLE users;
```

Если таблица может не существовать:

```sql
DROP TABLE IF EXISTS users;
```

---

# 11. Изменение таблицы

Добавить колонку:

```sql
ALTER TABLE users
ADD COLUMN phone text;
```

Удалить колонку:

```sql
ALTER TABLE users
DROP COLUMN phone;
```

Переименовать колонку:

```sql
ALTER TABLE users
RENAME COLUMN name TO full_name;
```

Переименовать таблицу:

```sql
ALTER TABLE users
RENAME TO app_users;
```

Добавить ограничение:

```sql
ALTER TABLE users
ADD CONSTRAINT users_age_check CHECK (age >= 0);
```

---

# 12. INSERT — добавление данных

```sql
INSERT INTO users (name, email, age)
VALUES ('Ivan', 'ivan@example.com', 25);
```

Можно добавить несколько строк:

```sql
INSERT INTO users (name, email, age)
VALUES
    ('Anna', 'anna@example.com', 30),
    ('Petr', 'petr@example.com', 22),
    ('Maria', 'maria@example.com', 27);
```

---

## INSERT RETURNING

PostgreSQL умеет сразу возвращать добавленную строку:

```sql
INSERT INTO users (name, email, age)
VALUES ('Oleg', 'oleg@example.com', 35)
RETURNING *;
```

Очень полезно в backend-разработке.

---

# 13. SELECT — получение данных

Получить все строки:

```sql
SELECT * FROM users;
```

Получить конкретные колонки:

```sql
SELECT id, name, email FROM users;
```

---

## WHERE — фильтрация

```sql
SELECT *
FROM users
WHERE age > 25;
```

---

## Операторы сравнения

```sql
=
!=
<>
>
<
>=
<=
```

Примеры:

```sql
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age >= 18;
SELECT * FROM users WHERE age <> 30;
```

`<>` и `!=` означают "не равно".

---

## AND / OR

```sql
SELECT *
FROM users
WHERE age > 18 AND is_active = true;
```

```sql
SELECT *
FROM users
WHERE age < 18 OR age > 65;
```

---

## BETWEEN

```sql
SELECT *
FROM users
WHERE age BETWEEN 18 AND 30;
```

Это значит:

```text
age >= 18 AND age <= 30
```

---

## IN

```sql
SELECT *
FROM users
WHERE age IN (20, 25, 30);
```

---

## LIKE

Поиск по шаблону.

```sql
SELECT *
FROM users
WHERE email LIKE '%@gmail.com';
```

`%` означает любое количество символов.

Примеры:

```sql
WHERE name LIKE 'A%'
```

Имя начинается на A.

```sql
WHERE name LIKE '%a'
```

Имя заканчивается на a.

```sql
WHERE name LIKE '%nn%'
```

В имени есть `nn`.

---

## ILIKE

То же самое, но без учёта регистра.

```sql
SELECT *
FROM users
WHERE name ILIKE 'anna';
```

Найдёт:

```text
Anna
ANNA
anna
```

---

## IS NULL

В SQL `NULL` — это отсутствие значения.

Проверять так:

```sql
SELECT *
FROM users
WHERE age IS NULL;
```

Не так:

```sql
WHERE age = NULL
```

Это неправильно.

---

## IS NOT NULL

```sql
SELECT *
FROM users
WHERE age IS NOT NULL;
```

---

# 14. ORDER BY — сортировка

```sql
SELECT *
FROM users
ORDER BY age;
```

По возрастанию:

```sql
ORDER BY age ASC
```

По убыванию:

```sql
ORDER BY age DESC
```

Сортировка по нескольким полям:

```sql
SELECT *
FROM users
ORDER BY is_active DESC, age ASC;
```

---

# 15. LIMIT и OFFSET

Ограничить количество строк:

```sql
SELECT *
FROM users
LIMIT 10;
```

Пропустить первые 10:

```sql
SELECT *
FROM users
LIMIT 10 OFFSET 10;
```

Часто используется для пагинации.

Пример:

```sql
-- страница 1
LIMIT 10 OFFSET 0;

-- страница 2
LIMIT 10 OFFSET 10;

-- страница 3
LIMIT 10 OFFSET 20;
```

---

# 16. UPDATE — изменение данных

```sql
UPDATE users
SET age = 26
WHERE id = 1;
```

Очень важно использовать `WHERE`.

Если написать:

```sql
UPDATE users
SET age = 26;
```

То возраст изменится у всех пользователей.

---

## UPDATE нескольких полей

```sql
UPDATE users
SET 
    name = 'Ivan Petrov',
    age = 27
WHERE id = 1;
```

---

## UPDATE RETURNING

```sql
UPDATE users
SET age = age + 1
WHERE id = 1
RETURNING *;
```

---

# 17. DELETE — удаление данных

```sql
DELETE FROM users
WHERE id = 1;
```

Без `WHERE` удалятся все строки:

```sql
DELETE FROM users;
```

---

## DELETE RETURNING

```sql
DELETE FROM users
WHERE id = 2
RETURNING *;
```

---

# 18. TRUNCATE

```sql
TRUNCATE TABLE users;
```

Удаляет все строки быстрее, чем `DELETE`.

Отличия:

- `DELETE` удаляет строки по одной;
- `TRUNCATE` очищает таблицу целиком;
- `TRUNCATE` обычно быстрее;
- `TRUNCATE` может сбросить счётчик `SERIAL`.

```sql
TRUNCATE TABLE users RESTART IDENTITY;
```

---

# 19. Практическая база: интернет-магазин

Создадим несколько таблиц.

---

## Таблица пользователей

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name text NOT NULL,
    email text NOT NULL UNIQUE,
    created_at timestamptz DEFAULT now()
);
```

---

## Таблица товаров

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    title text NOT NULL,
    price numeric(10, 2) NOT NULL CHECK (price >= 0),
    stock integer NOT NULL DEFAULT 0 CHECK (stock >= 0),
    created_at timestamptz DEFAULT now()
);
```

---

## Таблица заказов

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id integer NOT NULL REFERENCES users(id),
    status text NOT NULL DEFAULT 'new',
    created_at timestamptz DEFAULT now()
);
```

---

## Таблица позиций заказа

```sql
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id integer NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id integer NOT NULL REFERENCES products(id),
    quantity integer NOT NULL CHECK (quantity > 0),
    price numeric(10, 2) NOT NULL CHECK (price >= 0)
);
```

---

## Что здесь происходит

### users

Хранит пользователей.

### products

Хранит товары.

### orders

Хранит заказы.

`user_id` показывает, кто сделал заказ.

### order_items

Хранит товары внутри заказа.

Один заказ может содержать несколько товаров.

---

# 20. Связи между таблицами

## Один ко многим

Один пользователь может иметь много заказов.

```text
users 1 ---- N orders
```

В таблице `orders` есть поле:

```sql
user_id integer REFERENCES users(id)
```

---

## Многие ко многим

Например:

- один заказ содержит много товаров;
- один товар может быть во многих заказах.

Такую связь делают через промежуточную таблицу:

```text
orders ---- order_items ---- products
```

---

# 21. JOIN — объединение таблиц

JOIN нужен, чтобы получать связанные данные из нескольких таблиц.

---

## INNER JOIN

Получить заказы вместе с пользователями:

```sql
SELECT
    orders.id AS order_id,
    users.name AS user_name,
    orders.status,
    orders.created_at
FROM orders
INNER JOIN users ON users.id = orders.user_id;
```

`INNER JOIN` возвращает только те строки, где есть совпадение в обеих таблицах.

---

## LEFT JOIN

```sql
SELECT
    users.id,
    users.name,
    orders.id AS order_id
FROM users
LEFT JOIN orders ON orders.user_id = users.id;
```

`LEFT JOIN` вернёт всех пользователей, даже если у них нет заказов.

Если заказа нет, поля заказа будут `NULL`.

---

## RIGHT JOIN

Используется редко.

```sql
SELECT *
FROM orders
RIGHT JOIN users ON users.id = orders.user_id;
```

Обычно можно заменить на `LEFT JOIN`, поменяв таблицы местами.

---

## FULL JOIN

Возвращает все строки из обеих таблиц:

```sql
SELECT *
FROM users
FULL JOIN orders ON orders.user_id = users.id;
```

Используется нечасто.

---

# 22. Агрегатные функции

Агрегатные функции считают данные по группе строк.

---

## COUNT

Количество строк:

```sql
SELECT COUNT(*)
FROM users;
```

Количество заказов пользователя:

```sql
SELECT COUNT(*)
FROM orders
WHERE user_id = 1;
```

---

## SUM

Сумма:

```sql
SELECT SUM(quantity)
FROM order_items;
```

---

## AVG

Среднее значение:

```sql
SELECT AVG(price)
FROM products;
```

---

## MIN / MAX

```sql
SELECT MIN(price), MAX(price)
FROM products;
```

---

# 23. GROUP BY

`GROUP BY` группирует строки.

Например, посчитать количество заказов каждого пользователя:

```sql
SELECT
    users.id,
    users.name,
    COUNT(orders.id) AS orders_count
FROM users
LEFT JOIN orders ON orders.user_id = users.id
GROUP BY users.id, users.name;
```

---

## Важно

Если ты используешь агрегатные функции и обычные колонки, обычные колонки должны быть в `GROUP BY`.

Неправильно:

```sql
SELECT name, COUNT(*)
FROM users;
```

Правильно:

```sql
SELECT name, COUNT(*)
FROM users
GROUP BY name;
```

---

# 24. HAVING

`WHERE` фильтрует строки до группировки.

`HAVING` фильтрует группы после группировки.

Найти пользователей, у которых больше 2 заказов:

```sql
SELECT
    users.id,
    users.name,
    COUNT(orders.id) AS orders_count
FROM users
JOIN orders ON orders.user_id = users.id
GROUP BY users.id, users.name
HAVING COUNT(orders.id) > 2;
```

---

# 25. Подзапросы

Подзапрос — запрос внутри запроса.

---

## Пример

Найти товары дороже средней цены:

```sql
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

---

## Подзапрос в FROM

```sql
SELECT *
FROM (
    SELECT user_id, COUNT(*) AS orders_count
    FROM orders
    GROUP BY user_id
) AS user_orders
WHERE orders_count > 2;
```

---

## EXISTS

Проверить, есть ли связанные строки:

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Это значит: найти пользователей, у которых есть хотя бы один заказ.

---

# 26. CTE — WITH-запросы

CTE помогает писать сложные запросы понятнее.

```sql
WITH user_orders AS (
    SELECT user_id, COUNT(*) AS orders_count
    FROM orders
    GROUP BY user_id
)
SELECT
    users.name,
    user_orders.orders_count
FROM users
JOIN user_orders ON user_orders.user_id = users.id;
```

CTE — это временная именованная таблица внутри одного запроса.

---

# 27. UNION

`UNION` объединяет результаты нескольких запросов.

```sql
SELECT email FROM users
UNION
SELECT email FROM admins;
```

`UNION` убирает дубликаты.

Если нужны дубликаты:

```sql
SELECT email FROM users
UNION ALL
SELECT email FROM admins;
```

---

# 28. DISTINCT

Убрать дубликаты:

```sql
SELECT DISTINCT status
FROM orders;
```

---

## DISTINCT ON

Особенность PostgreSQL.

Например, получить последний заказ каждого пользователя:

```sql
SELECT DISTINCT ON (user_id)
    id,
    user_id,
    status,
    created_at
FROM orders
ORDER BY user_id, created_at DESC;
```

Важно: `ORDER BY` должен начинаться с колонок из `DISTINCT ON`.

---

# 29. CASE

`CASE` — условная логика в SQL.

```sql
SELECT
    title,
    price,
    CASE
        WHEN price < 100 THEN 'cheap'
        WHEN price < 1000 THEN 'normal'
        ELSE 'expensive'
    END AS price_category
FROM products;
```

---

# 30. COALESCE

`COALESCE` возвращает первое не-NULL значение.

```sql
SELECT COALESCE(age, 0)
FROM users;
```

Если `age = NULL`, вернётся `0`.

---

# 31. NULL — очень важная тема

`NULL` — это не ноль и не пустая строка.

`NULL` значит: значение неизвестно или отсутствует.

---

## Примеры

```sql
SELECT NULL = NULL;
```

Результат не `true`, а `NULL`.

Правильно проверять:

```sql
IS NULL
IS NOT NULL
```

---

## NULL в выражениях

```sql
SELECT 10 + NULL;
```

Результат:

```text
NULL
```

Потому что если часть значения неизвестна, итог тоже неизвестен.

---

# 32. Ограничения Constraints

Ограничения защищают данные от ошибок.

---

## PRIMARY KEY

```sql
id SERIAL PRIMARY KEY
```

Гарантирует уникальность строки.

---

## UNIQUE

```sql
email text UNIQUE
```

Не даёт двум строкам иметь одинаковый email.

---

## NOT NULL

```sql
name text NOT NULL
```

Не даёт сохранить пустое значение.

---

## CHECK

```sql
age integer CHECK (age >= 0)
```

Не даст вставить отрицательный возраст.

---

## FOREIGN KEY

```sql
user_id integer REFERENCES users(id)
```

Не даст создать заказ для несуществующего пользователя.

---

# 33. ON DELETE

Когда удаляем строку, на которую ссылаются другие строки, PostgreSQL должен понять, что делать.

---

## ON DELETE CASCADE

```sql
order_id integer REFERENCES orders(id) ON DELETE CASCADE
```

Если удалить заказ, удалятся и его позиции.

---

## ON DELETE SET NULL

```sql
user_id integer REFERENCES users(id) ON DELETE SET NULL
```

Если удалить пользователя, поле `user_id` станет `NULL`.

---

## ON DELETE RESTRICT / NO ACTION

Запрещает удаление, если есть связанные записи.

---

# 34. Индексы

Индекс — структура данных, которая ускоряет поиск.

Простая аналогия: оглавление в книге.

Без оглавления нужно листать всю книгу.

С оглавлением можно быстро найти нужную страницу.

---

## Создание индекса

```sql
CREATE INDEX idx_users_email
ON users(email);
```

---

## Когда индекс полезен

Если часто ищешь по колонке:

```sql
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

Индекс на `email` поможет.

---

## Индекс для сортировки

```sql
CREATE INDEX idx_orders_created_at
ON orders(created_at);
```

Может помочь запросам:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 10;
```

---

## Составной индекс

```sql
CREATE INDEX idx_orders_user_id_created_at
ON orders(user_id, created_at DESC);
```

Полезен для запроса:

```sql
SELECT *
FROM orders
WHERE user_id = 1
ORDER BY created_at DESC;
```

---

## Важный порядок колонок

Индекс:

```sql
ON orders(user_id, created_at)
```

хорош для:

```sql
WHERE user_id = 1
```

и для:

```sql
WHERE user_id = 1 AND created_at > now() - interval '1 month'
```

Но не всегда хорош для:

```sql
WHERE created_at > now() - interval '1 month'
```

Потому что первая колонка индекса — `user_id`.

---

## Уникальный индекс

```sql
CREATE UNIQUE INDEX idx_users_email_unique
ON users(email);
```

Но обычно проще:

```sql
email text UNIQUE
```

---

## Частичный индекс

Индекс только по части строк.

```sql
CREATE INDEX idx_active_users_email
ON users(email)
WHERE is_active = true;
```

Полезно, если часто ищешь только активных пользователей.

---

## Индекс по выражению

```sql
CREATE INDEX idx_users_lower_email
ON users(lower(email));
```

Запрос:

```sql
SELECT *
FROM users
WHERE lower(email) = lower('IVAN@example.com');
```

---

# 35. EXPLAIN

`EXPLAIN` показывает план выполнения запроса.

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

Лучше использовать:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

`EXPLAIN ANALYZE` реально выполняет запрос и показывает время.

---

## Основные термины

### Seq Scan

Последовательное сканирование таблицы.

PostgreSQL читает всю таблицу.

На маленьких таблицах это нормально.

---

### Index Scan

Используется индекс.

Обычно хорошо, если нужно найти небольшую часть строк.

---

### Bitmap Index Scan

Комбинированный способ через индекс.

Часто встречается при выборке большого количества строк.

---

## Пример

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

Если индекса нет, может быть:

```text
Seq Scan on users
```

Создаём индекс:

```sql
CREATE INDEX idx_users_email ON users(email);
```

Повторяем:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

Теперь может быть:

```text
Index Scan using idx_users_email
```

---

# 36. Транзакции

Транзакция — набор операций, которые выполняются как единое целое.

Главная идея:

> либо выполняется всё, либо не выполняется ничего.

---

## Пример

Покупатель оформляет заказ:

1. создать заказ;
2. добавить товары в заказ;
3. уменьшить остатки на складе;
4. списать оплату.

Если что-то сломалось на шаге 3, нельзя оставить заказ наполовину созданным.

---

## BEGIN, COMMIT, ROLLBACK

```sql
BEGIN;

UPDATE products
SET stock = stock - 1
WHERE id = 1;

INSERT INTO orders (user_id)
VALUES (1);

COMMIT;
```

Если всё хорошо — `COMMIT`.

Если ошибка:

```sql
ROLLBACK;
```

---

## Пример с откатом

```sql
BEGIN;

UPDATE products
SET stock = stock - 10
WHERE id = 1;

-- передумали
ROLLBACK;
```

Изменения не сохранятся.

---

# 37. ACID

Транзакции в PostgreSQL следуют принципам ACID.

---

## Atomicity — атомарность

Либо всё, либо ничего.

---

## Consistency — согласованность

Данные остаются корректными.

Например, нельзя создать заказ с несуществующим пользователем, если есть foreign key.

---

## Isolation — изолированность

Параллельные транзакции не должны ломать друг друга.

---

## Durability — долговечность

После `COMMIT` данные не должны исчезнуть даже при сбое.

---

# 38. Уровни изоляции транзакций

PostgreSQL поддерживает разные уровни изоляции.

---

## READ COMMITTED

Уровень по умолчанию.

Транзакция видит только зафиксированные данные.

```sql
BEGIN ISOLATION LEVEL READ COMMITTED;
```

---

## REPEATABLE READ

Внутри транзакции данные выглядят одинаково на протяжении всей транзакции.

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
```

---

## SERIALIZABLE

Самый строгий уровень.

Транзакции выполняются так, будто они идут по очереди.

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
```

---

# 39. Блокировки

Блокировка нужна, чтобы две транзакции не изменили одни и те же данные неправильно.

---

## SELECT FOR UPDATE

```sql
BEGIN;

SELECT *
FROM products
WHERE id = 1
FOR UPDATE;

UPDATE products
SET stock = stock - 1
WHERE id = 1;

COMMIT;
```

`FOR UPDATE` блокирует выбранную строку до конца транзакции.

---

## Пример проблемы

На складе остался 1 товар.

Два пользователя одновременно покупают товар.

Без правильной транзакции оба могут купить последний товар.

Правильнее:

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 1
FOR UPDATE;

UPDATE products
SET stock = stock - 1
WHERE id = 1 AND stock > 0;

COMMIT;
```

---

# 40. UPSERT — INSERT ON CONFLICT

Иногда нужно:

- если строки нет — добавить;
- если строка есть — обновить.

---

## Пример

```sql
INSERT INTO users (email, name)
VALUES ('ivan@example.com', 'Ivan')
ON CONFLICT (email)
DO UPDATE SET name = EXCLUDED.name;
```

`EXCLUDED` — это новая строка, которую пытались вставить.

---

## Ничего не делать при конфликте

```sql
INSERT INTO users (email, name)
VALUES ('ivan@example.com', 'Ivan')
ON CONFLICT (email)
DO NOTHING;
```

---

# 41. Представления Views

View — сохранённый SQL-запрос, который выглядит как таблица.

---

## Создание view

```sql
CREATE VIEW user_orders_view AS
SELECT
    users.id AS user_id,
    users.name,
    COUNT(orders.id) AS orders_count
FROM users
LEFT JOIN orders ON orders.user_id = users.id
GROUP BY users.id, users.name;
```

Использование:

```sql
SELECT *
FROM user_orders_view;
```

---

## Зачем нужны views

- упростить сложные запросы;
- скрыть часть данных;
- дать удобный интерфейс для отчётов.

---

# 42. Materialized View

Обычный `VIEW` каждый раз выполняет запрос заново.

`MATERIALIZED VIEW` хранит результат физически.

```sql
CREATE MATERIALIZED VIEW product_stats AS
SELECT
    product_id,
    SUM(quantity) AS total_sold
FROM order_items
GROUP BY product_id;
```

Обновить:

```sql
REFRESH MATERIALIZED VIEW product_stats;
```

---

# 43. Оконные функции

Оконные функции позволяют считать значения по строкам, не объединяя их в одну строку как `GROUP BY`.

---

## ROW_NUMBER

```sql
SELECT
    id,
    user_id,
    created_at,
    ROW_NUMBER() OVER (
        PARTITION BY user_id
        ORDER BY created_at DESC
    ) AS order_number
FROM orders;
```

Это нумерует заказы каждого пользователя отдельно.

---

## RANK

```sql
SELECT
    title,
    price,
    RANK() OVER (ORDER BY price DESC) AS price_rank
FROM products;
```

Если цены одинаковые, будет одинаковый ранг.

---

## SUM OVER

```sql
SELECT
    id,
    created_at,
    SUM(price) OVER (ORDER BY created_at) AS running_total
FROM payments;
```

Это накопительная сумма.

---

# 44. Работа с датами

Текущая дата:

```sql
SELECT current_date;
```

Текущее время:

```sql
SELECT now();
```

Добавить интервал:

```sql
SELECT now() + interval '1 day';
```

Вычесть:

```sql
SELECT now() - interval '7 days';
```

Заказы за последние 7 дней:

```sql
SELECT *
FROM orders
WHERE created_at >= now() - interval '7 days';
```

---

## date_trunc

Округление даты.

```sql
SELECT date_trunc('day', created_at)
FROM orders;
```

Продажи по дням:

```sql
SELECT
    date_trunc('day', orders.created_at) AS day,
    SUM(order_items.quantity * order_items.price) AS revenue
FROM orders
JOIN order_items ON order_items.order_id = orders.id
GROUP BY day
ORDER BY day;
```

---

# 45. JSONB

PostgreSQL хорошо работает с JSON.

Создадим таблицу:

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data jsonb NOT NULL,
    created_at timestamptz DEFAULT now()
);
```

Добавим данные:

```sql
INSERT INTO events (data)
VALUES (
    '{"type": "click", "user_id": 1, "page": "/home"}'
);
```

---

## Получить поле из JSON

```sql
SELECT data->'type'
FROM events;
```

Вернёт JSON-значение.

```sql
SELECT data->>'type'
FROM events;
```

Вернёт текст.

---

## Фильтрация JSONB

```sql
SELECT *
FROM events
WHERE data->>'type' = 'click';
```

---

## Проверка наличия ключа

```sql
SELECT *
FROM events
WHERE data ? 'page';
```

---

## JSONB индекс

```sql
CREATE INDEX idx_events_data
ON events USING GIN (data);
```

GIN-индекс полезен для JSONB-поиска.

---

# 46. Массивы

PostgreSQL умеет хранить массивы.

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title text,
    tags text[]
);
```

Добавить:

```sql
INSERT INTO posts (title, tags)
VALUES ('PostgreSQL intro', ARRAY['postgresql', 'sql', 'database']);
```

Найти посты с тегом:

```sql
SELECT *
FROM posts
WHERE 'sql' = ANY(tags);
```

Но в реальных проектах часто лучше делать отдельную таблицу тегов, а не массив.

---

# 47. Enum

Enum — ограниченный список значений.

```sql
CREATE TYPE order_status AS ENUM ('new', 'paid', 'shipped', 'cancelled');
```

Использование:

```sql
CREATE TABLE orders2 (
    id SERIAL PRIMARY KEY,
    status order_status NOT NULL DEFAULT 'new'
);
```

Плюсы:

- нельзя записать неправильный статус.

Минусы:

- сложнее менять список значений.

В реальных проектах часто вместо enum используют `text + CHECK`.

```sql
status text CHECK (status IN ('new', 'paid', 'shipped', 'cancelled'))
```

---

# 48. Нормализация базы данных

Нормализация — это способ проектировать таблицы так, чтобы:

- не было лишнего дублирования;
- данные были логичными;
- было меньше ошибок.

---

## Плохой пример

```text
orders
id | user_name | user_email | product_title | product_price
```

Проблемы:

- имя пользователя повторяется в каждом заказе;
- если email изменился, нужно менять много строк;
- товар повторяется много раз.

---

## Хороший пример

```text
users
id | name | email

products
id | title | price

orders
id | user_id

order_items
id | order_id | product_id | quantity | price
```

Так данные разделены по смыслу.

---

# 49. Денормализация

Иногда ради скорости данные специально дублируют.

Например, в `order_items` хранят `price`.

Хотя цена есть в `products`.

Почему?

Потому что цена товара может измениться, а в старом заказе должна остаться старая цена.

---

# 50. Миграции

Миграции — это изменения структуры базы данных, сохранённые в файлах.

Например:

```sql
001_create_users.sql
002_create_products.sql
003_add_phone_to_users.sql
```

Миграции нужны, чтобы:

- команда работала с одинаковой структурой БД;
- можно было воспроизвести базу;
- изменения были под контролем Git.

---

## Пример миграции вверх

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email text NOT NULL UNIQUE
);
```

## Пример отката

```sql
DROP TABLE users;
```

---

# 51. Роли и права

PostgreSQL использует роли.

Роль может быть:

- пользователем;
- группой;
- администратором.

---

## Создать пользователя

```sql
CREATE USER app_user WITH PASSWORD 'strong_password';
```

---

## Дать права на базу

```sql
GRANT CONNECT ON DATABASE shop TO app_user;
```

---

## Дать права на схему

```sql
GRANT USAGE ON SCHEMA public TO app_user;
```

---

## Дать права на таблицы

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO app_user;
```

---

## Дать права на будущие таблицы

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
```

---

## Отнять права

```sql
REVOKE DELETE ON users FROM app_user;
```

---

# 52. Backup и Restore

## pg_dump

Сделать бэкап базы:

```bash
pg_dump -U postgres -d shop -f shop_backup.sql
```

---

## Восстановить

```bash
psql -U postgres -d shop -f shop_backup.sql
```

---

## Custom формат

```bash
pg_dump -U postgres -d shop -Fc -f shop_backup.dump
```

Восстановление:

```bash
pg_restore -U postgres -d shop shop_backup.dump
```

---

# 53. VACUUM и ANALYZE

PostgreSQL не всегда физически удаляет старые версии строк сразу.

Из-за MVCC после UPDATE/DELETE могут оставаться "мёртвые" строки.

---

## VACUUM

Очищает мёртвые строки:

```sql
VACUUM;
```

---

## ANALYZE

Собирает статистику для планировщика запросов:

```sql
ANALYZE;
```

---

## VACUUM ANALYZE

```sql
VACUUM ANALYZE;
```

Обычно в PostgreSQL работает autovacuum, но понимать это важно.

---

# 54. MVCC

MVCC — механизм конкурентного доступа в PostgreSQL.

Расшифровка:

```text
Multi-Version Concurrency Control
```

Идея:

- когда строку обновляют, PostgreSQL создаёт новую версию строки;
- старые транзакции могут видеть старую версию;
- новые транзакции видят новую версию.

Это позволяет читать данные без постоянных блокировок.

---

# 55. Последовательности Sequence

`SERIAL` внутри использует sequence.

Пример:

```sql
CREATE SEQUENCE users_id_seq;
```

Получить следующее значение:

```sql
SELECT nextval('users_id_seq');
```

Посмотреть текущее:

```sql
SELECT currval('users_id_seq');
```

В новых проектах часто используют:

```sql
id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

Вместо старого:

```sql
id SERIAL PRIMARY KEY
```

Современный вариант:

```sql
CREATE TABLE users (
    id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL UNIQUE
);
```

---

# 56. Функции

PostgreSQL позволяет создавать свои функции.

---

## Простая SQL-функция

```sql
CREATE FUNCTION add_numbers(a integer, b integer)
RETURNS integer
LANGUAGE SQL
AS $$
    SELECT a + b;
$$;
```

Использование:

```sql
SELECT add_numbers(2, 3);
```

---

## PL/pgSQL функция

```sql
CREATE FUNCTION get_discount(price numeric)
RETURNS numeric
LANGUAGE plpgsql
AS $$
BEGIN
    IF price > 1000 THEN
        RETURN price * 0.10;
    ELSE
        RETURN 0;
    END IF;
END;
$$;
```

Использование:

```sql
SELECT get_discount(1500);
```

---

# 57. Триггеры

Триггер — действие, которое автоматически выполняется при INSERT, UPDATE или DELETE.

---

## Пример: обновлять updated_at

Создадим таблицу:

```sql
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title text NOT NULL,
    content text,
    created_at timestamptz DEFAULT now(),
    updated_at timestamptz DEFAULT now()
);
```

Функция:

```sql
CREATE FUNCTION set_updated_at()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$;
```

Триггер:

```sql
CREATE TRIGGER articles_set_updated_at
BEFORE UPDATE ON articles
FOR EACH ROW
EXECUTE FUNCTION set_updated_at();
```

Теперь при обновлении статьи `updated_at` будет меняться автоматически.

---

# 58. Full-text search

PostgreSQL умеет полнотекстовый поиск.

```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title text,
    body text
);
```

Поиск:

```sql
SELECT *
FROM documents
WHERE to_tsvector('russian', title || ' ' || body)
      @@ plainto_tsquery('russian', 'поиск текста');
```

Индекс:

```sql
CREATE INDEX idx_documents_fts
ON documents
USING GIN (to_tsvector('russian', title || ' ' || body));
```

---

# 59. Расширения Extensions

Расширения добавляют возможности.

Посмотреть доступные:

```sql
SELECT *
FROM pg_available_extensions;
```

Установить:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

Популярные:

```text
uuid-ossp
pgcrypto
postgis
pg_trgm
citext
```

---

## pgcrypto для UUID

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

```sql
CREATE TABLE users_uuid (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    email text NOT NULL UNIQUE
);
```

---

## citext

`citext` — текст без учёта регистра.

```sql
CREATE EXTENSION IF NOT EXISTS citext;
```

```sql
CREATE TABLE users_ci (
    email citext UNIQUE
);
```

Теперь:

```text
Test@mail.com
test@mail.com
```

считаются одинаковыми.

---

# 60. Практические запросы уровня Junior+

## 1. Найти пользователя по email

```sql
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

---

## 2. Получить последние 10 заказов пользователя

```sql
SELECT *
FROM orders
WHERE user_id = 1
ORDER BY created_at DESC
LIMIT 10;
```

Индекс:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);
```

---

## 3. Посчитать сумму заказа

```sql
SELECT
    order_id,
    SUM(quantity * price) AS total
FROM order_items
WHERE order_id = 1
GROUP BY order_id;
```

---

## 4. Получить заказы с суммой

```sql
SELECT
    orders.id,
    users.name,
    orders.created_at,
    SUM(order_items.quantity * order_items.price) AS total
FROM orders
JOIN users ON users.id = orders.user_id
JOIN order_items ON order_items.order_id = orders.id
GROUP BY orders.id, users.name, orders.created_at
ORDER BY orders.created_at DESC;
```

---

## 5. Найти товары, которые ни разу не покупали

```sql
SELECT p.*
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.id
WHERE oi.id IS NULL;
```

---

## 6. Топ-5 товаров по продажам

```sql
SELECT
    p.id,
    p.title,
    SUM(oi.quantity) AS total_sold
FROM products p
JOIN order_items oi ON oi.product_id = p.id
GROUP BY p.id, p.title
ORDER BY total_sold DESC
LIMIT 5;
```

---

## 7. Выручка по дням

```sql
SELECT
    date_trunc('day', o.created_at) AS day,
    SUM(oi.quantity * oi.price) AS revenue
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
GROUP BY day
ORDER BY day;
```

---

## 8. Пользователи без заказов

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL;
```

---

## 9. Последний заказ каждого пользователя

```sql
SELECT DISTINCT ON (user_id)
    id,
    user_id,
    status,
    created_at
FROM orders
ORDER BY user_id, created_at DESC;
```

---

## 10. Количество заказов по статусам

```sql
SELECT
    status,
    COUNT(*) AS count
FROM orders
GROUP BY status
ORDER BY count DESC;
```

---

# 61. Типичные ошибки новичков

## Ошибка 1: UPDATE без WHERE

Опасно:

```sql
UPDATE users
SET is_active = false;
```

Так ты изменишь всех пользователей.

Правильно:

```sql
UPDATE users
SET is_active = false
WHERE id = 1;
```

---

## Ошибка 2: DELETE без WHERE

Опасно:

```sql
DELETE FROM users;
```

Правильно:

```sql
DELETE FROM users
WHERE id = 1;
```

---

## Ошибка 3: Проверка NULL через =

Неправильно:

```sql
WHERE deleted_at = NULL
```

Правильно:

```sql
WHERE deleted_at IS NULL
```

---

## Ошибка 4: Использовать float для денег

Плохо:

```sql
price double precision
```

Лучше:

```sql
price numeric(10, 2)
```

---

## Ошибка 5: Не создавать индексы для foreign key

Часто полезно индексировать внешние ключи:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

---

## Ошибка 6: Хранить всё в одной таблице

Плохо:

```text
orders_with_all_data
```

Лучше разделять:

```text
users
orders
products
order_items
```

---

# 62. Что должен знать уверенный Junior+ по PostgreSQL

Ты должен уметь:

1. Создавать базы данных и таблицы.
2. Понимать типы данных.
3. Использовать `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
4. Писать `JOIN`.
5. Использовать `GROUP BY`, `HAVING`, агрегатные функции.
6. Понимать `NULL`.
7. Создавать `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`.
8. Понимать связи один-ко-многим и многие-ко-многим.
9. Читать простые планы `EXPLAIN`.
10. Создавать индексы.
11. Понимать транзакции.
12. Использовать `BEGIN`, `COMMIT`, `ROLLBACK`.
13. Понимать базовую конкуренцию и `FOR UPDATE`.
14. Делать простые backup/restore.
15. Работать с `jsonb`.
16. Писать CTE.
17. Использовать оконные функции.
18. Понимать миграции.
19. Работать с ролями и правами.
20. Проектировать нормальную схему БД.

---

# 63. Мини-проект для практики

Создай базу данных для интернет-магазина.

## Нужно реализовать таблицы

- `users`
- `products`
- `orders`
- `order_items`
- `payments`
- `categories`
- `product_categories`

---

## Пример категорий

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    title text NOT NULL UNIQUE
);
```

---

## Связь товаров и категорий

```sql
CREATE TABLE product_categories (
    product_id integer NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    category_id integer NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    PRIMARY KEY (product_id, category_id)
);
```

Это связь многие-ко-многим:

- товар может быть в нескольких категориях;
- категория может содержать много товаров.

---

## Платежи

```sql
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    order_id integer NOT NULL REFERENCES orders(id),
    amount numeric(10, 2) NOT NULL CHECK (amount >= 0),
    status text NOT NULL CHECK (status IN ('pending', 'paid', 'failed', 'refunded')),
    created_at timestamptz DEFAULT now()
);
```

---

# 64. Практические задания

## Простые

1. Добавь 5 пользователей.
2. Добавь 10 товаров.
3. Создай 3 заказа.
4. Добавь товары в заказы.
5. Получи всех пользователей.
6. Получи товары дороже 1000.
7. Найди пользователя по email.

---

## Средние

1. Получи все заказы пользователя.
2. Посчитай сумму каждого заказа.
3. Получи топ-3 самых дорогих товара.
4. Найди пользователей без заказов.
5. Найди товары, которых нет на складе.

---

## Junior+

1. Получи выручку по дням.
2. Получи топ-5 товаров по продажам.
3. Получи последний заказ каждого пользователя.
4. Реализуй транзакцию оформления заказа.
5. Добавь индексы и проверь через `EXPLAIN ANALYZE`.
6. Сделай backup и restore.
7. Добавь JSONB-таблицу событий.
8. Сделай view для статистики пользователей.

---

# 65. Пример транзакции оформления заказа

Допустим, пользователь покупает товар.

```sql
BEGIN;

-- 1. Проверяем и блокируем товар
SELECT *
FROM products
WHERE id = 1
FOR UPDATE;

-- 2. Уменьшаем остаток, если товара хватает
UPDATE products
SET stock = stock - 2
WHERE id = 1 AND stock >= 2;

-- 3. Создаём заказ
INSERT INTO orders (user_id, status)
VALUES (1, 'new')
RETURNING id;

-- допустим вернулся order_id = 10

-- 4. Добавляем товар в заказ
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (10, 1, 2, 999.99);

COMMIT;
```

В реальном приложении нужно проверить, что `UPDATE products` реально обновил строку.

---

# 66. Хорошие привычки

## Всегда используй WHERE в UPDATE/DELETE

Перед `UPDATE` можно сначала сделать `SELECT`:

```sql
SELECT *
FROM users
WHERE id = 1;
```

Потом:

```sql
UPDATE users
SET name = 'New Name'
WHERE id = 1;
```

---

## Используй транзакции для связанных операций

Если операция состоит из нескольких шагов — используй транзакцию.

---

## Не храни деньги во float

Используй:

```sql
numeric(10, 2)
```

---

## Индексируй частые фильтры

Если часто используешь:

```sql
WHERE user_id = ?
```

создай индекс:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

---

## Не создавай индексы на всё подряд

Индексы ускоряют чтение, но замедляют вставку и обновление.

Каждый индекс нужно поддерживать.

---

## Используй понятные имена

Хорошо:

```text
users
orders
order_items
created_at
updated_at
```

Плохо:

```text
tbl1
data2
usr_nm
```

---

# 67. Краткая шпаргалка SQL

```sql
-- создать таблицу
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email text NOT NULL UNIQUE
);

-- добавить строку
INSERT INTO users (email)
VALUES ('test@example.com');

-- получить данные
SELECT *
FROM users;

-- фильтрация
SELECT *
FROM users
WHERE email = 'test@example.com';

-- обновить
UPDATE users
SET email = 'new@example.com'
WHERE id = 1;

-- удалить
DELETE FROM users
WHERE id = 1;

-- сортировка
SELECT *
FROM users
ORDER BY id DESC;

-- лимит
SELECT *
FROM users
LIMIT 10;

-- join
SELECT *
FROM orders
JOIN users ON users.id = orders.user_id;

-- группировка
SELECT user_id, COUNT(*)
FROM orders
GROUP BY user_id;

-- транзакция
BEGIN;
UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT;

-- индекс
CREATE INDEX idx_users_email ON users(email);
```

---

# 68. Как учиться дальше

Лучший путь:

1. Выучи базовые SQL-команды.
2. Много практикуй `SELECT`.
3. Разбери `JOIN`.
4. Разбери `GROUP BY`.
5. Научись проектировать таблицы.
6. Изучи индексы.
7. Изучи транзакции.
8. Начни читать `EXPLAIN`.
9. Сделай мини-проект.
10. Подключи PostgreSQL к backend-приложению.

---

# 69. Итог

PostgreSQL — мощная реляционная база данных.

Для уровня Junior+ тебе особенно важно уверенно знать:

- таблицы;
- типы данных;
- ограничения;
- связи;
- CRUD;
- JOIN;
- GROUP BY;
- индексы;
- транзакции;
- миграции;
- backup/restore;
- основы оптимизации;
- нормализацию.

Если ты хорошо отработаешь примеры из этой документации и сделаешь мини-проект интернет-магазина, у тебя будет хорошая база для Junior/Juinor+ уровня.
