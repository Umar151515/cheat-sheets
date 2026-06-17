# PostgreSQL с нуля до уверенного Junior+

Это учебная документация-конспект по PostgreSQL: от базовых понятий до уровня, когда ты можешь уверенно работать с БД в реальном проекте: проектировать таблицы, писать SQL-запросы, понимать индексы, транзакции, связи, ограничения, базовую оптимизацию и администрирование.

---

# 1. Что такое PostgreSQL

**PostgreSQL** — это реляционная система управления базами данных, или СУБД.

Простыми словами:

- данные хранятся в таблицах;
- таблицы связаны между собой;
- для работы используется язык SQL;
- PostgreSQL умеет обеспечивать целостность данных, транзакции, индексы, права доступа, резервное копирование и многое другое.

PostgreSQL часто используют в backend-разработке, аналитике, финтехе, CRM, ERP, интернет-магазинах, SaaS-сервисах.

---

# 2. Базовые термины

## База данных

**База данных** — логическое хранилище данных.

Пример:

```text
online_shop
```

Внутри базы данных могут быть таблицы:

```text
users
orders
products
payments
```

---

## Таблица

**Таблица** — структура, в которой данные хранятся строками и столбцами.

Пример таблицы `users`:

| id | name | email |
|---:|------|-------|
| 1 | Ivan | ivan@mail.com |
| 2 | Anna | anna@mail.com |

---

## Строка

**Строка** — одна запись в таблице.

Например:

```text
1, Ivan, ivan@mail.com
```

---

## Столбец

**Столбец** — поле записи.

Например:

```text
id
name
email
```

---

## SQL

**SQL** — язык для работы с реляционными базами данных.

С его помощью можно:

- создавать таблицы;
- добавлять данные;
- читать данные;
- изменять данные;
- удалять данные;
- создавать индексы;
- управлять правами;
- работать с транзакциями.

---

# 3. Установка PostgreSQL

## Вариант 1. Установка через Docker

Это самый удобный способ для обучения.

Создай файл `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: postgres_db
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: shop
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Запуск:

```bash
docker compose up -d
```

Подключение:

```bash
docker exec -it postgres_db psql -U admin -d shop
```

Остановка:

```bash
docker compose down
```

---

## Вариант 2. Установка на Windows/macOS/Linux

Скачать можно с официального сайта:

```text
https://www.postgresql.org/download/
```

После установки обычно появляются:

- PostgreSQL Server;
- pgAdmin;
- psql.

---

# 4. Подключение к PostgreSQL

## Через psql

```bash
psql -U admin -d shop -h localhost -p 5432
```

Где:

```text
-U admin       пользователь
-d shop        база данных
-h localhost   хост
-p 5432        порт
```

---

## Полезные команды psql

Показать список баз данных:

```sql
\l
```

Подключиться к базе:

```sql
\c shop
```

Показать таблицы:

```sql
\dt
```

Показать структуру таблицы:

```sql
\d users
```

Выйти:

```sql
\q
```

Очистить экран:

```sql
\! clear
```

или на Windows:

```sql
\! cls
```

---

# 5. Создание базы данных

```sql
CREATE DATABASE shop;
```

Удаление базы:

```sql
DROP DATABASE shop;
```

Создание пользователя:

```sql
CREATE USER app_user WITH PASSWORD '123456';
```

Выдать права:

```sql
GRANT ALL PRIVILEGES ON DATABASE shop TO app_user;
```

---

# 6. Типы данных PostgreSQL

## Числовые типы

| Тип | Описание |
|---|---|
| `smallint` | маленькое целое число |
| `integer` / `int` | обычное целое число |
| `bigint` | большое целое число |
| `numeric` | точное число, например для денег |
| `real` | число с плавающей точкой |
| `double precision` | более точное число с плавающей точкой |

Пример:

```sql
price numeric(10, 2)
```

Это значит:

- всего 10 цифр;
- 2 цифры после запятой.

Например:

```text
99999999.99
```

---

## Строковые типы

| Тип | Описание |
|---|---|
| `varchar(n)` | строка максимум n символов |
| `text` | строка произвольной длины |
| `char(n)` | строка фиксированной длины |

На практике чаще всего используют:

```sql
text
varchar(255)
```

---

## Даты и время

| Тип | Описание |
|---|---|
| `date` | дата |
| `time` | время |
| `timestamp` | дата и время |
| `timestamptz` | дата и время с часовым поясом |

Для реальных приложений часто лучше использовать:

```sql
timestamptz
```

Пример:

```sql
created_at timestamptz DEFAULT now()
```

---

## Логический тип

```sql
boolean
```

Значения:

```text
true
false
```

Пример:

```sql
is_active boolean DEFAULT true
```

---

## UUID

UUID — уникальный идентификатор.

Пример:

```text
550e8400-e29b-41d4-a716-446655440000
```

Чтобы использовать генерацию UUID:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

Пример поля:

```sql
id uuid PRIMARY KEY DEFAULT uuid_generate_v4()
```

В новых версиях PostgreSQL также можно использовать:

```sql
gen_random_uuid()
```

Для этого:

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

---

## JSON и JSONB

PostgreSQL умеет хранить JSON.

```sql
data jsonb
```

Лучше чаще использовать `jsonb`, потому что он эффективнее для поиска и индексации.

Пример:

```sql
CREATE TABLE events (
    id serial PRIMARY KEY,
    payload jsonb
);
```

---

# 7. Создание таблиц

## Простейшая таблица

```sql
CREATE TABLE users (
    id serial PRIMARY KEY,
    name text NOT NULL,
    email text NOT NULL,
    age int
);
```

Что здесь происходит:

```sql
id serial PRIMARY KEY
```

- `id` — поле;
- `serial` — автоинкремент;
- `PRIMARY KEY` — первичный ключ.

```sql
name text NOT NULL
```

- `name` — имя пользователя;
- `text` — строка;
- `NOT NULL` — значение обязательно.

---

## Современный вариант вместо serial

Сейчас часто используют:

```sql
GENERATED ALWAYS AS IDENTITY
```

Пример:

```sql
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL,
    email text NOT NULL
);
```

Это более современный стандарт SQL.

---

# 8. Ограничения таблиц

Ограничения нужны, чтобы база данных сама защищала данные от ошибок.

## PRIMARY KEY

Первичный ключ уникально идентифицирует строку.

```sql
id bigint PRIMARY KEY
```

Обычно:

- не может быть `NULL`;
- должен быть уникальным.

---

## NOT NULL

Запрещает пустое значение.

```sql
email text NOT NULL
```

---

## UNIQUE

Гарантирует уникальность.

```sql
email text UNIQUE
```

Теперь нельзя добавить двух пользователей с одинаковым email.

---

## CHECK

Проверяет условие.

```sql
age int CHECK (age >= 0)
```

Пример:

```sql
CREATE TABLE products (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL,
    price numeric(10, 2) NOT NULL CHECK (price >= 0)
);
```

---

## DEFAULT

Значение по умолчанию.

```sql
created_at timestamptz DEFAULT now()
```

Пример:

```sql
CREATE TABLE posts (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title text NOT NULL,
    is_published boolean DEFAULT false,
    created_at timestamptz DEFAULT now()
);
```

---

## FOREIGN KEY

Внешний ключ связывает одну таблицу с другой.

Пример:

```sql
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL
);

CREATE TABLE orders (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),
    total numeric(10, 2) NOT NULL
);
```

Здесь:

```sql
user_id bigint NOT NULL REFERENCES users(id)
```

означает:

- заказ принадлежит пользователю;
- нельзя создать заказ для несуществующего пользователя.

---

# 9. Удаление и изменение таблиц

## Удалить таблицу

```sql
DROP TABLE users;
```

Если таблица может быть связана с другими:

```sql
DROP TABLE users CASCADE;
```

`CASCADE` удалит зависимые объекты. Используй осторожно.

---

## Добавить колонку

```sql
ALTER TABLE users ADD COLUMN phone text;
```

---

## Удалить колонку

```sql
ALTER TABLE users DROP COLUMN phone;
```

---

## Изменить тип колонки

```sql
ALTER TABLE users ALTER COLUMN age TYPE bigint;
```

---

## Добавить ограничение

```sql
ALTER TABLE users ADD CONSTRAINT users_email_unique UNIQUE (email);
```

---

## Удалить ограничение

```sql
ALTER TABLE users DROP CONSTRAINT users_email_unique;
```

---

# 10. CRUD-операции

CRUD:

- `CREATE` — создать данные;
- `READ` — прочитать данные;
- `UPDATE` — обновить данные;
- `DELETE` — удалить данные.

---

## INSERT

Добавить пользователя:

```sql
INSERT INTO users (name, email, age)
VALUES ('Ivan', 'ivan@example.com', 25);
```

Добавить несколько пользователей:

```sql
INSERT INTO users (name, email, age)
VALUES 
    ('Anna', 'anna@example.com', 30),
    ('Petr', 'petr@example.com', 22),
    ('Maria', 'maria@example.com', 27);
```

Вернуть добавленную строку:

```sql
INSERT INTO users (name, email, age)
VALUES ('Alex', 'alex@example.com', 20)
RETURNING *;
```

---

## SELECT

Получить все данные:

```sql
SELECT * FROM users;
```

Получить конкретные колонки:

```sql
SELECT id, name, email FROM users;
```

---

## WHERE

Фильтрация:

```sql
SELECT * FROM users
WHERE age > 25;
```

Несколько условий:

```sql
SELECT * FROM users
WHERE age > 20 AND age < 30;
```

Или:

```sql
SELECT * FROM users
WHERE age < 20 OR age > 60;
```

---

## UPDATE

Обновить данные:

```sql
UPDATE users
SET age = 26
WHERE id = 1;
```

Важно: почти всегда нужен `WHERE`.

Опасный запрос:

```sql
UPDATE users
SET age = 0;
```

Он обновит все строки.

---

## DELETE

Удалить строку:

```sql
DELETE FROM users
WHERE id = 1;
```

Опасный запрос:

```sql
DELETE FROM users;
```

Он удалит все строки.

---

# 11. Сортировка, лимиты, смещения

## ORDER BY

```sql
SELECT * FROM users
ORDER BY age ASC;
```

`ASC` — по возрастанию.

```sql
SELECT * FROM users
ORDER BY age DESC;
```

`DESC` — по убыванию.

---

## LIMIT

```sql
SELECT * FROM users
LIMIT 10;
```

Вернёт максимум 10 строк.

---

## OFFSET

```sql
SELECT * FROM users
ORDER BY id
LIMIT 10 OFFSET 20;
```

Пропустит 20 строк и вернёт следующие 10.

Используется для пагинации, но на больших таблицах может быть медленным.

---

# 12. Операторы сравнения

```sql
=       равно
<>      не равно
!=      не равно
>       больше
<       меньше
>=      больше или равно
<=      меньше или равно
```

Пример:

```sql
SELECT * FROM products
WHERE price >= 1000;
```

---

# 13. BETWEEN, IN, LIKE, ILIKE, IS NULL

## BETWEEN

```sql
SELECT * FROM products
WHERE price BETWEEN 100 AND 500;
```

То же самое:

```sql
WHERE price >= 100 AND price <= 500
```

---

## IN

```sql
SELECT * FROM users
WHERE id IN (1, 2, 3);
```

---

## LIKE

Поиск по шаблону, чувствительный к регистру:

```sql
SELECT * FROM users
WHERE name LIKE 'A%';
```

`%` означает любое количество символов.

---

## ILIKE

Поиск без учёта регистра:

```sql
SELECT * FROM users
WHERE name ILIKE 'a%';
```

---

## IS NULL

Проверка на `NULL`:

```sql
SELECT * FROM users
WHERE phone IS NULL;
```

Проверка, что значение не `NULL`:

```sql
SELECT * FROM users
WHERE phone IS NOT NULL;
```

Важно:

```sql
WHERE phone = NULL
```

так писать нельзя. Нужно:

```sql
WHERE phone IS NULL
```

---

# 14. NULL

`NULL` — отсутствие значения.

Это не ноль, не пустая строка, не `false`.

Пример:

```sql
SELECT NULL = NULL;
```

Результат не `true`, а `NULL`, потому что неизвестно, равно ли неизвестное неизвестному.

Для сравнения с `NULL` используй:

```sql
IS NULL
IS NOT NULL
```

---

## COALESCE

`COALESCE` возвращает первое не-NULL значение.

```sql
SELECT COALESCE(phone, 'Телефон не указан')
FROM users;
```

---

# 15. Агрегатные функции

Агрегатные функции считают значения по группе строк.

## COUNT

```sql
SELECT COUNT(*) FROM users;
```

---

## SUM

```sql
SELECT SUM(total) FROM orders;
```

---

## AVG

```sql
SELECT AVG(price) FROM products;
```

---

## MIN и MAX

```sql
SELECT MIN(price), MAX(price)
FROM products;
```

---

# 16. GROUP BY

`GROUP BY` группирует строки.

Пример: сумма заказов по каждому пользователю.

```sql
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id;
```

Пример с количеством заказов:

```sql
SELECT user_id, COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

---

# 17. HAVING

`WHERE` фильтрует строки до группировки.

`HAVING` фильтрует группы после группировки.

Пример: пользователи, у которых больше 3 заказов.

```sql
SELECT user_id, COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 3;
```

---

# 18. JOIN

`JOIN` нужен, чтобы получать данные из нескольких таблиц.

Создадим пример:

```sql
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL
);

CREATE TABLE orders (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),
    total numeric(10, 2) NOT NULL
);
```

---

## INNER JOIN

Возвращает только строки, где есть совпадения в обеих таблицах.

```sql
SELECT 
    users.id,
    users.name,
    orders.id AS order_id,
    orders.total
FROM users
INNER JOIN orders ON orders.user_id = users.id;
```

Если у пользователя нет заказов, он не попадёт в результат.

---

## LEFT JOIN

Возвращает все строки из левой таблицы и совпадения из правой.

```sql
SELECT 
    users.id,
    users.name,
    orders.id AS order_id,
    orders.total
FROM users
LEFT JOIN orders ON orders.user_id = users.id;
```

Если у пользователя нет заказов, он всё равно будет в результате, а поля заказа будут `NULL`.

---

## RIGHT JOIN

Возвращает все строки из правой таблицы.

```sql
SELECT *
FROM users
RIGHT JOIN orders ON orders.user_id = users.id;
```

На практике используется реже, потому что почти всегда можно переписать через `LEFT JOIN`.

---

## FULL JOIN

Возвращает все строки из обеих таблиц.

```sql
SELECT *
FROM users
FULL JOIN orders ON orders.user_id = users.id;
```

---

## CROSS JOIN

Декартово произведение: каждая строка с каждой.

```sql
SELECT *
FROM users
CROSS JOIN products;
```

Использовать осторожно.

---

# 19. Алиасы

Алиасы делают запросы короче и читаемее.

```sql
SELECT 
    u.id,
    u.name,
    o.id AS order_id,
    o.total
FROM users AS u
JOIN orders AS o ON o.user_id = u.id;
```

`AS` можно опускать:

```sql
FROM users u
```

---

# 20. Подзапросы

## Подзапрос в WHERE

Найти пользователей, у которых есть заказы:

```sql
SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

---

## EXISTS

Часто лучше использовать `EXISTS`:

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

## Подзапрос в FROM

```sql
SELECT avg_orders.total_avg
FROM (
    SELECT AVG(total) AS total_avg
    FROM orders
) AS avg_orders;
```

---

# 21. CTE — WITH-запросы

CTE делает сложные запросы понятнее.

```sql
WITH user_orders AS (
    SELECT 
        user_id,
        COUNT(*) AS orders_count,
        SUM(total) AS total_sum
    FROM orders
    GROUP BY user_id
)
SELECT 
    u.name,
    uo.orders_count,
    uo.total_sum
FROM users u
JOIN user_orders uo ON uo.user_id = u.id;
```

CTE читается как временная именованная таблица внутри запроса.

---

# 22. Оконные функции

Оконные функции позволяют считать значения по набору строк, не сворачивая результат как `GROUP BY`.

## ROW_NUMBER

```sql
SELECT 
    id,
    name,
    ROW_NUMBER() OVER (ORDER BY id) AS row_num
FROM users;
```

---

## RANK

```sql
SELECT 
    id,
    name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank
FROM players;
```

---

## SUM OVER

```sql
SELECT 
    id,
    total,
    SUM(total) OVER (ORDER BY id) AS running_total
FROM orders;
```

Это накопительная сумма.

---

## PARTITION BY

```sql
SELECT 
    user_id,
    id AS order_id,
    total,
    SUM(total) OVER (PARTITION BY user_id) AS user_total
FROM orders;
```

Считает сумму заказов отдельно для каждого пользователя.

---

# 23. Нормализация данных

Нормализация — это способ проектирования таблиц так, чтобы:

- не было лишнего дублирования;
- данные были целостными;
- изменения не приводили к противоречиям.

---

## Плохой пример

```text
orders
------------------------------------------------
id | user_name | user_email | product_name | price
```

Проблемы:

- имя пользователя повторяется в каждом заказе;
- email повторяется;
- название товара повторяется;
- если email изменился, нужно менять много строк.

---

## Хороший пример

```text
users
id | name | email

products
id | name | price

orders
id | user_id | created_at

order_items
id | order_id | product_id | quantity | price
```

Так данные разделены логически.

---

# 24. Связи между таблицами

## Один к одному

Например:

```text
users
user_profiles
```

Один пользователь имеет один профиль.

```sql
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL UNIQUE
);

CREATE TABLE user_profiles (
    user_id bigint PRIMARY KEY REFERENCES users(id),
    first_name text,
    last_name text
);
```

---

## Один ко многим

Один пользователь может иметь много заказов.

```sql
CREATE TABLE orders (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id)
);
```

---

## Многие ко многим

Например: товары и категории.

Один товар может быть в нескольких категориях.

Одна категория может содержать много товаров.

Нужна промежуточная таблица:

```sql
CREATE TABLE products (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL
);

CREATE TABLE categories (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL
);

CREATE TABLE product_categories (
    product_id bigint NOT NULL REFERENCES products(id),
    category_id bigint NOT NULL REFERENCES categories(id),
    PRIMARY KEY (product_id, category_id)
);
```

---

# 25. ON DELETE и ON UPDATE

Когда есть внешний ключ, важно определить поведение при удалении родительской записи.

## ON DELETE RESTRICT / NO ACTION

Запретить удаление, если есть связанные записи.

```sql
user_id bigint REFERENCES users(id) ON DELETE RESTRICT
```

---

## ON DELETE CASCADE

Удалить связанные записи автоматически.

```sql
user_id bigint REFERENCES users(id) ON DELETE CASCADE
```

Если удалить пользователя, удалятся его заказы.

Использовать осторожно.

---

## ON DELETE SET NULL

При удалении родителя поставить `NULL`.

```sql
user_id bigint REFERENCES users(id) ON DELETE SET NULL
```

Для этого колонка должна позволять `NULL`.

---

# 26. Индексы

Индекс — структура данных, которая ускоряет поиск.

Без индекса PostgreSQL может читать всю таблицу.

С индексом PostgreSQL может быстрее найти нужные строки.

---

## Создание индекса

```sql
CREATE INDEX idx_users_email ON users(email);
```

---

## UNIQUE INDEX

```sql
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

Но если нужна уникальность, чаще проще:

```sql
email text UNIQUE
```

---

## Когда индекс полезен

Индекс полезен для колонок, которые часто используются в:

```sql
WHERE
JOIN
ORDER BY
GROUP BY
```

Пример:

```sql
SELECT * FROM users WHERE email = 'ivan@example.com';
```

Здесь индекс по `email` может быть полезен.

---

## Когда индекс может быть вреден

Индексы:

- занимают место;
- замедляют `INSERT`;
- замедляют `UPDATE`;
- замедляют `DELETE`.

Потому что при изменении данных нужно обновлять не только таблицу, но и индексы.

---

## Индекс для внешнего ключа

PostgreSQL автоматически создаёт индекс для `PRIMARY KEY` и `UNIQUE`.

Но для `FOREIGN KEY` индекс автоматически не создаётся.

Если часто делаешь JOIN или удаляешь родительские записи, стоит создать индекс:

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

---

## Составной индекс

```sql
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

Полезен для запросов:

```sql
WHERE user_id = 1 AND status = 'paid'
```

Также может использоваться для:

```sql
WHERE user_id = 1
```

Но не всегда хорошо для:

```sql
WHERE status = 'paid'
```

Порядок колонок в составном индексе важен.

---

## Частичный индекс

Индекс только по части таблицы.

```sql
CREATE INDEX idx_orders_unpaid
ON orders(user_id)
WHERE status = 'unpaid';
```

Полезно, если часто ищешь только неоплаченные заказы.

---

## Индекс по выражению

```sql
CREATE INDEX idx_users_lower_email
ON users (lower(email));
```

Тогда запрос:

```sql
SELECT *
FROM users
WHERE lower(email) = lower('IVAN@EXAMPLE.COM');
```

может использовать индекс.

---

# 27. EXPLAIN и EXPLAIN ANALYZE

`EXPLAIN` показывает план выполнения запроса.

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

`EXPLAIN ANALYZE` реально выполняет запрос и показывает фактическое время.

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'ivan@example.com';
```

---

## Основные термины в плане

## Seq Scan

Последовательное сканирование всей таблицы.

```text
Seq Scan on users
```

Не всегда плохо. Для маленьких таблиц это нормально.

---

## Index Scan

Использование индекса.

```text
Index Scan using idx_users_email on users
```

---

## Bitmap Index Scan

Часто используется, когда нужно найти много строк через индекс.

---

## Nested Loop

Один из алгоритмов JOIN.

Хорош для маленьких наборов или когда есть хороший индекс.

---

## Hash Join

Часто хорош для больших наборов данных.

---

# 28. Транзакции

Транзакция — это группа операций, которые выполняются как единое целое.

Главный принцип: либо выполняется всё, либо ничего.

---

## Пример

Допустим, пользователь переводит деньги другому пользователю.

Нужно:

1. списать деньги с одного счёта;
2. зачислить деньги на другой счёт.

Если первая операция прошла, а вторая нет, данные испортятся.

Поэтому нужна транзакция.

---

## BEGIN, COMMIT, ROLLBACK

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Если произошла ошибка:

```sql
ROLLBACK;
```

---

## ACID

Транзакции обладают свойствами ACID.

### Atomicity — атомарность

Всё или ничего.

### Consistency — согласованность

Данные остаются корректными.

### Isolation — изоляция

Параллельные транзакции не должны ломать друг друга.

### Durability — долговечность

После `COMMIT` данные сохранены.

---

# 29. Уровни изоляции транзакций

PostgreSQL поддерживает:

```sql
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Также в SQL есть `READ UNCOMMITTED`, но в PostgreSQL он работает как `READ COMMITTED`.

---

## READ COMMITTED

Уровень по умолчанию.

Каждый SQL-запрос внутри транзакции видит только зафиксированные данные на момент начала этого конкретного запроса.

```sql
BEGIN ISOLATION LEVEL READ COMMITTED;
```

---

## REPEATABLE READ

Все запросы внутри транзакции видят снимок данных на момент начала транзакции.

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
```

---

## SERIALIZABLE

Самый строгий уровень.

PostgreSQL старается выполнить транзакции так, как будто они выполнялись последовательно.

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
```

Может чаще приводить к ошибкам сериализации, которые приложение должно повторять.

---

# 30. Блокировки

Блокировки нужны, чтобы параллельные операции не портили данные.

Пример:

```sql
BEGIN;

SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

COMMIT;
```

`FOR UPDATE` блокирует выбранные строки для изменения другими транзакциями.

---

# 31. UPSERT: INSERT ON CONFLICT

Иногда нужно:

- вставить строку;
- если такая уже есть — обновить.

Пример:

```sql
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL UNIQUE,
    name text NOT NULL
);
```

```sql
INSERT INTO users (email, name)
VALUES ('ivan@example.com', 'Ivan')
ON CONFLICT (email)
DO UPDATE SET name = EXCLUDED.name;
```

`EXCLUDED` — это данные, которые пытались вставить.

---

Если при конфликте ничего делать не нужно:

```sql
INSERT INTO users (email, name)
VALUES ('ivan@example.com', 'Ivan')
ON CONFLICT (email)
DO NOTHING;
```

---

# 32. Представления VIEW

`VIEW` — сохранённый запрос.

```sql
CREATE VIEW user_order_stats AS
SELECT 
    u.id,
    u.name,
    COUNT(o.id) AS orders_count,
    COALESCE(SUM(o.total), 0) AS total_sum
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

Теперь можно писать:

```sql
SELECT *
FROM user_order_stats;
```

---

## MATERIALIZED VIEW

Материализованное представление хранит результат физически.

```sql
CREATE MATERIALIZED VIEW user_order_stats_mv AS
SELECT 
    user_id,
    COUNT(*) AS orders_count,
    SUM(total) AS total_sum
FROM orders
GROUP BY user_id;
```

Обновить данные:

```sql
REFRESH MATERIALIZED VIEW user_order_stats_mv;
```

---

# 33. Функции

PostgreSQL позволяет создавать функции.

Пример:

```sql
CREATE OR REPLACE FUNCTION add_numbers(a int, b int)
RETURNS int
LANGUAGE sql
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
CREATE OR REPLACE FUNCTION get_discount(price numeric)
RETURNS numeric
LANGUAGE plpgsql
AS $$
BEGIN
    IF price > 10000 THEN
        RETURN price * 0.9;
    ELSE
        RETURN price;
    END IF;
END;
$$;
```

---

# 34. Триггеры

Триггер — функция, которая автоматически срабатывает при `INSERT`, `UPDATE` или `DELETE`.

Пример: автоматически обновлять `updated_at`.

```sql
CREATE TABLE posts (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title text NOT NULL,
    created_at timestamptz DEFAULT now(),
    updated_at timestamptz DEFAULT now()
);
```

Функция:

```sql
CREATE OR REPLACE FUNCTION set_updated_at()
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
CREATE TRIGGER trg_posts_updated_at
BEFORE UPDATE ON posts
FOR EACH ROW
EXECUTE FUNCTION set_updated_at();
```

---

# 35. Схемы

Схема — пространство имён внутри базы данных.

По умолчанию используется схема:

```sql
public
```

Создать схему:

```sql
CREATE SCHEMA app;
```

Создать таблицу в схеме:

```sql
CREATE TABLE app.users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL
);
```

Обращение:

```sql
SELECT *
FROM app.users;
```

---

# 36. Права доступа

## Создание роли

```sql
CREATE ROLE app_user LOGIN PASSWORD 'strong_password';
```

---

## Выдать права на подключение

```sql
GRANT CONNECT ON DATABASE shop TO app_user;
```

---

## Выдать права на схему

```sql
GRANT USAGE ON SCHEMA public TO app_user;
```

---

## Выдать права на таблицы

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO app_user;
```

---

## Права на будущие таблицы

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
```

---

## Отозвать права

```sql
REVOKE DELETE ON users FROM app_user;
```

---

# 37. Backup и Restore

## Резервная копия через pg_dump

```bash
pg_dump -U admin -d shop -f backup.sql
```

---

## Восстановление

```bash
psql -U admin -d shop -f backup.sql
```

---

## Формат custom

Создание:

```bash
pg_dump -U admin -d shop -Fc -f backup.dump
```

Восстановление:

```bash
pg_restore -U admin -d shop backup.dump
```

---

## Бэкап всех баз

```bash
pg_dumpall -U postgres > all_backup.sql
```

---

# 38. Импорт и экспорт CSV

## Экспорт

```sql
COPY users TO '/tmp/users.csv' WITH CSV HEADER;
```

На стороне клиента через psql:

```sql
\copy users TO 'users.csv' WITH CSV HEADER
```

---

## Импорт

```sql
\copy users(name, email, age) FROM 'users.csv' WITH CSV HEADER
```

---

# 39. Практическая схема интернет-магазина

Ниже пример нормальной учебной структуры.

```sql
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL UNIQUE,
    name text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE products (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL,
    description text,
    price numeric(10, 2) NOT NULL CHECK (price >= 0),
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE orders (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    status text NOT NULL DEFAULT 'new',
    total numeric(10, 2) NOT NULL DEFAULT 0 CHECK (total >= 0),
    created_at timestamptz NOT NULL DEFAULT now(),
    CHECK (status IN ('new', 'paid', 'cancelled', 'shipped'))
);

CREATE TABLE order_items (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id bigint NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id bigint NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    quantity int NOT NULL CHECK (quantity > 0),
    price numeric(10, 2) NOT NULL CHECK (price >= 0)
);
```

Индексы:

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
```

---

# 40. Примеры запросов для интернет-магазина

## Добавить пользователя

```sql
INSERT INTO users (email, name)
VALUES ('ivan@example.com', 'Ivan');
```

---

## Добавить товары

```sql
INSERT INTO products (name, price)
VALUES 
    ('Laptop', 120000),
    ('Mouse', 2500),
    ('Keyboard', 7000);
```

---

## Создать заказ

```sql
INSERT INTO orders (user_id)
VALUES (1)
RETURNING id;
```

---

## Добавить товары в заказ

```sql
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES 
    (1, 1, 1, 120000),
    (1, 2, 2, 2500);
```

---

## Пересчитать сумму заказа

```sql
UPDATE orders
SET total = (
    SELECT SUM(quantity * price)
    FROM order_items
    WHERE order_id = 1
)
WHERE id = 1;
```

---

## Получить заказы пользователя

```sql
SELECT 
    o.id,
    o.status,
    o.total,
    o.created_at
FROM orders o
WHERE o.user_id = 1
ORDER BY o.created_at DESC;
```

---

## Получить заказ с товарами

```sql
SELECT 
    o.id AS order_id,
    o.status,
    u.name AS user_name,
    p.name AS product_name,
    oi.quantity,
    oi.price,
    oi.quantity * oi.price AS item_total
FROM orders o
JOIN users u ON u.id = o.user_id
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
WHERE o.id = 1;
```

---

## Топ пользователей по сумме заказов

```sql
SELECT 
    u.id,
    u.name,
    SUM(o.total) AS total_spent
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.status = 'paid'
GROUP BY u.id, u.name
ORDER BY total_spent DESC
LIMIT 10;
```

---

# 41. Миграции

В реальных проектах структуру БД не меняют вручную хаотично.

Используют миграции.

Миграция — это файл с изменением схемы.

Пример:

```sql
-- 001_create_users.sql
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL UNIQUE,
    name text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);
```

Следующая миграция:

```sql
-- 002_add_phone_to_users.sql
ALTER TABLE users ADD COLUMN phone text;
```

Популярные инструменты:

- Flyway;
- Liquibase;
- Prisma Migrate;
- TypeORM migrations;
- Sequelize migrations;
- Alembic для Python;
- goose;
- golang-migrate.

---

# 42. Хорошие практики проектирования

## 1. Всегда добавляй PRIMARY KEY

Почти каждая таблица должна иметь первичный ключ.

---

## 2. Используй NOT NULL там, где значение обязательно

Плохо:

```sql
email text
```

Лучше:

```sql
email text NOT NULL
```

---

## 3. Используй внешние ключи

Плохо хранить `user_id`, который ни на кого не ссылается.

Лучше:

```sql
user_id bigint NOT NULL REFERENCES users(id)
```

---

## 4. Не храни списки в одной строке

Плохо:

```text
product_ids = '1,2,3,4'
```

Лучше создать отдельную таблицу связей.

---

## 5. Для денег используй numeric

Плохо:

```sql
price float
```

Лучше:

```sql
price numeric(10, 2)
```

---

## 6. Для времени часто используй timestamptz

```sql
created_at timestamptz NOT NULL DEFAULT now()
```

---

## 7. Создавай индексы осознанно

Не надо индексировать все колонки подряд.

---

## 8. Называй ограничения и индексы понятно

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

---

## 9. Не используй SELECT * в production-коде без необходимости

Лучше:

```sql
SELECT id, name, email FROM users;
```

---

## 10. Проверяй тяжёлые запросы через EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT ...
```

---

# 43. Частые ошибки новичков

## Ошибка 1. Забыли WHERE в UPDATE

```sql
UPDATE users SET is_active = false;
```

Это обновит всех пользователей.

---

## Ошибка 2. Забыли WHERE в DELETE

```sql
DELETE FROM users;
```

Это удалит всех пользователей.

---

## Ошибка 3. Используют `=` для NULL

Плохо:

```sql
WHERE deleted_at = NULL
```

Правильно:

```sql
WHERE deleted_at IS NULL
```

---

## Ошибка 4. Хранят дату строкой

Плохо:

```sql
created_at text
```

Правильно:

```sql
created_at timestamptz
```

---

## Ошибка 5. Не создают индекс на FK

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

---

## Ошибка 6. Используют float для денег

Плохо:

```sql
price double precision
```

Правильно:

```sql
price numeric(10, 2)
```

---

# 44. Что должен знать Junior+ по PostgreSQL

## SQL

Ты должен уверенно знать:

- `SELECT`;
- `INSERT`;
- `UPDATE`;
- `DELETE`;
- `WHERE`;
- `JOIN`;
- `GROUP BY`;
- `HAVING`;
- `ORDER BY`;
- `LIMIT`;
- `OFFSET`;
- подзапросы;
- CTE;
- агрегатные функции;
- оконные функции на базовом уровне.

---

## Схема БД

Ты должен понимать:

- таблицы;
- типы данных;
- первичные ключи;
- внешние ключи;
- уникальные ограничения;
- `NOT NULL`;
- `CHECK`;
- связи один-к-одному, один-ко-многим, многие-ко-многим.

---

## Индексы

Ты должен понимать:

- зачем нужны индексы;
- когда индекс помогает;
- когда индекс мешает;
- что такое составной индекс;
- что такое частичный индекс;
- почему порядок колонок важен;
- как смотреть план через `EXPLAIN`.

---

## Транзакции

Ты должен знать:

- `BEGIN`;
- `COMMIT`;
- `ROLLBACK`;
- ACID;
- базовые уровни изоляции;
- `SELECT FOR UPDATE`.

---

## Администрирование

На базовом уровне:

- создать базу;
- создать пользователя;
- выдать права;
- сделать backup;
- восстановить backup;
- импортировать CSV;
- подключиться через psql.

---

# 45. Практические задания

## Задание 1

Создай базу данных `library`.

Таблицы:

```text
authors
books
readers
loans
```

Связи:

- один автор может иметь много книг;
- один читатель может брать много книг;
- одна выдача связана с одной книгой и одним читателем.

---

## Задание 2

Напиши запросы:

1. добавить автора;
2. добавить книгу;
3. добавить читателя;
4. выдать книгу читателю;
5. вернуть книгу;
6. найти все книги конкретного автора;
7. найти всех читателей, у которых сейчас есть книги;
8. найти топ-5 самых популярных книг.

---

## Задание 3

Добавь ограничения:

- email читателя должен быть уникальным;
- дата возврата может быть `NULL`;
- название книги не может быть пустым;
- год публикации должен быть больше 0.

---

## Задание 4

Добавь индексы:

- на `books.author_id`;
- на `loans.reader_id`;
- на `loans.book_id`;
- на `readers.email`.

---

## Задание 5

Проверь запросы через:

```sql
EXPLAIN ANALYZE
```

---

# 46. Мини-шпаргалка SQL

```sql
-- создать таблицу
CREATE TABLE users (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL UNIQUE,
    name text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);

-- добавить данные
INSERT INTO users (email, name)
VALUES ('test@example.com', 'Test');

-- получить данные
SELECT id, email, name
FROM users
WHERE email = 'test@example.com';

-- обновить данные
UPDATE users
SET name = 'New Name'
WHERE id = 1;

-- удалить данные
DELETE FROM users
WHERE id = 1;

-- join
SELECT u.name, o.total
FROM users u
JOIN orders o ON o.user_id = u.id;

-- группировка
SELECT user_id, COUNT(*)
FROM orders
GROUP BY user_id;

-- транзакция
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- индекс
CREATE INDEX idx_users_email ON users(email);

-- план запроса
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'test@example.com';
```

---

# 47. Рекомендуемый порядок изучения

1. Установить PostgreSQL.
2. Научиться подключаться через `psql` или DBeaver.
3. Выучить `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
4. Понять типы данных.
5. Понять `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`.
6. Научиться делать `JOIN`.
7. Освоить `GROUP BY`, агрегаты, `HAVING`.
8. Изучить подзапросы и CTE.
9. Понять нормализацию и связи таблиц.
10. Изучить индексы.
11. Научиться читать `EXPLAIN`.
12. Освоить транзакции.
13. Научиться делать backup/restore.
14. Сделать 2–3 учебных проекта с БД.

---

# 48. Итог

Чтобы быть уверенным Junior+ по PostgreSQL, тебе нужно уметь:

- создавать нормальные таблицы;
- правильно выбирать типы данных;
- писать CRUD-запросы;
- писать JOIN-запросы;
- агрегировать данные;
- проектировать связи;
- использовать ограничения;
- понимать индексы;
- читать простые планы выполнения;
- работать с транзакциями;
- делать backup и restore;
- не ломать данные опасными запросами.

Если ты хорошо освоишь всё из этого конспекта и выполнишь практические задания, у тебя будет крепкая база PostgreSQL для работы backend-разработчиком, аналитиком или junior database developer.
