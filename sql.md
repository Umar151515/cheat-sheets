# PostgreSQL с нуля до Base/Junior+  
## Интерактивная шпаргалка-учебник на примере интернет-магазина

Мы будем строить сквозную базу данных **интернет-магазина**.

Предметная область:

- есть покупатели;
- есть товары;
- есть категории товаров;
- покупатели делают заказы;
- заказ состоит из нескольких позиций;
- у каждой позиции есть товар, количество и цена на момент покупки;
- у заказов есть статусы.

---

# Модуль 1. Введение и основы реляционных БД

## 1.1. Что такое база данных

**База данных** — это организованное хранилище данных.

Если объяснять на пальцах:

- обычный файл Excel — это тоже способ хранить данные;
- папка с документами — тоже хранилище;
- но база данных — это хранилище, которое умеет:
  - быстро искать данные;
  - безопасно изменять данные;
  - проверять правила;
  - обслуживать много пользователей одновременно;
  - защищать данные от случайной порчи;
  - связывать данные между собой.

Пример из жизни.

Представь интернет-магазин. Ему нужно хранить:

- пользователей;
- товары;
- заказы;
- оплаты;
- доставки;
- остатки на складе.

Можно хранить это в Excel, но быстро появятся проблемы:

- два менеджера одновременно изменили один файл;
- случайно удалили важную строку;
- товар в заказе ссылается на несуществующий товар;
- цена записана текстом `"сто рублей"`;
- нужно быстро найти все заказы клиента за последний месяц.

База данных решает эти проблемы.

---

## 1.2. Что такое СУБД

**СУБД** — система управления базами данных.

Это программа, которая управляет базами данных.

Примеры СУБД:

- PostgreSQL;
- MySQL;
- MariaDB;
- SQLite;
- Oracle Database;
- Microsoft SQL Server.

Если база данных — это склад, то СУБД — это управляющий складом:

- принимает запросы;
- достаёт нужные данные;
- следит за правилами;
- не даёт двум людям сломать один и тот же объект;
- ведёт журнал изменений;
- помогает восстановиться после сбоя.

---

## 1.3. Почему PostgreSQL

PostgreSQL — одна из самых мощных и популярных реляционных СУБД с открытым исходным кодом.

### Плюсы PostgreSQL

1. **Бесплатная и open-source**

Можно использовать в коммерческих проектах без покупки лицензии.

2. **Надёжная**

PostgreSQL используется в банках, маркетплейсах, аналитике, high-load сервисах.

3. **Поддерживает строгие правила данных**

Можно задать:

- первичные ключи;
- внешние ключи;
- уникальность;
- проверки;
- значения по умолчанию;
- транзакции.

4. **Хорошо работает с SQL**

PostgreSQL очень близок к стандарту SQL, но при этом имеет много мощных расширений.

5. **Много типов данных**

Есть:

- числа;
- строки;
- даты;
- JSON;
- массивы;
- UUID;
- геоданные через расширения;
- full-text search.

6. **Отлично подходит для backend-разработки**

Если ты пишешь на Java, Python, Go, Node.js, PHP, C#, PostgreSQL почти всегда хороший выбор.

---

## 1.4. Реляционная база данных

**Реляционная БД** хранит данные в виде таблиц.

Таблица похожа на лист Excel:

| id | name        | email              |
|----|-------------|--------------------|
| 1  | Иван        | ivan@example.com   |
| 2  | Мария       | maria@example.com  |

Но в отличие от Excel, таблицы в БД:

- имеют строгую структуру;
- имеют типы данных;
- могут иметь ограничения;
- могут быть связаны друг с другом.

---

## 1.5. Таблица, строка, столбец

### Таблица

**Таблица** — это набор однотипных записей.

Например, таблица `customers` хранит покупателей.

```text
customers
```

| id | first_name | last_name | email              |
|----|------------|-----------|--------------------|
| 1  | Иван       | Петров    | ivan@example.com   |
| 2  | Анна       | Смирнова  | anna@example.com   |

---

### Строка

**Строка** — одна запись в таблице.

Например:

| id | first_name | last_name | email            |
|----|------------|-----------|------------------|
| 1  | Иван       | Петров    | ivan@example.com |

Это один покупатель.

---

### Столбец

**Столбец** — конкретное свойство записи.

Например:

- `id`;
- `first_name`;
- `last_name`;
- `email`.

---

## 1.6. Типы данных в PostgreSQL

Тип данных говорит базе:

> Какие значения можно хранить в этой колонке.

Это как коробки на складе:

- в коробку для чисел нельзя положить дату;
- в коробку для даты нельзя положить email;
- в коробку для логического значения нельзя положить цену.

---

## 1.7. Основные типы данных

### INT / INTEGER

Целое число.

```sql
age INTEGER
```

Подходит для:

- возраста;
- количества товаров;
- номера этажа;
- количества попыток входа.

Пример:

```sql
quantity INTEGER
```

Возможные значения:

```text
1
5
100
-10
```

Не подходит для денег, потому что деньги часто имеют копейки.

---

### BIGINT

Большое целое число.

```sql
id BIGINT
```

Используется, когда чисел может быть очень много.

Например:

- идентификаторы пользователей;
- идентификаторы заказов;
- счётчики событий.

Для первичных ключей в реальных проектах часто используют `BIGINT`.

---

### VARCHAR(n)

Строка ограниченной длины.

```sql
email VARCHAR(255)
```

`VARCHAR(255)` означает:

> можно хранить строку длиной до 255 символов.

Подходит для:

- email;
- телефона;
- имени;
- артикула товара.

Пример:

```sql
first_name VARCHAR(100)
```

---

### TEXT

Строка без явного ограничения длины.

```sql
description TEXT
```

Подходит для:

- описания товара;
- комментария;
- текста статьи;
- адреса, если он может быть длинным.

В PostgreSQL `TEXT` часто используют спокойно. В отличие от некоторых других СУБД, `TEXT` в PostgreSQL — нормальный и удобный тип.

---

### TIMESTAMP

Дата и время.

```sql
created_at TIMESTAMP
```

Пример значения:

```text
2026-06-13 15:30:00
```

Есть два важных варианта:

```sql
TIMESTAMP WITHOUT TIME ZONE
TIMESTAMP WITH TIME ZONE
```

Чаще в backend-проектах используют:

```sql
TIMESTAMP WITH TIME ZONE
```

или коротко:

```sql
TIMESTAMPTZ
```

Он хранит момент времени с учётом часового пояса.

Пример:

```sql
created_at TIMESTAMPTZ
```

---

### NUMERIC

Точное число с дробной частью.

```sql
price NUMERIC(10, 2)
```

`NUMERIC(10, 2)` означает:

- всего 10 цифр;
- 2 цифры после запятой.

Примеры:

```text
99999999.99
1500.00
49.90
```

Для денег лучше использовать `NUMERIC`, а не `FLOAT`.

Почему?

`FLOAT` — приблизительный тип. Он может хранить `0.1 + 0.2` не идеально точно.

Для денег это плохо.

---

### BOOLEAN

Логический тип.

```sql
is_active BOOLEAN
```

Возможные значения:

```text
TRUE
FALSE
NULL
```

Примеры:

```sql
is_active BOOLEAN DEFAULT TRUE
is_deleted BOOLEAN DEFAULT FALSE
```

---

## 1.8. NULL

`NULL` означает:

> Значение неизвестно или отсутствует.

Это не `0`, не пустая строка, не `false`.

Пример:

| id | phone |
|----|-------|
| 1  | NULL  |

Это значит:

> Мы не знаем телефон пользователя.

---

## 1.9. Primary Key

**Primary Key**, или первичный ключ, — это уникальный идентификатор строки.

Пример:

| id | first_name | email            |
|----|------------|------------------|
| 1  | Иван       | ivan@example.com |
| 2  | Анна       | anna@example.com |

Здесь `id` — первичный ключ.

Он нужен, чтобы точно отличать одну строку от другой.

---

## 1.10. Почему нельзя полагаться только на имя или email

Имя не подходит:

| id | first_name |
|----|------------|
| 1  | Александр  |
| 2  | Александр  |

Email почти подходит, но может измениться.

Пользователь может сменить email, а `id` должен оставаться постоянным.

---

## 1.11. Хороший первичный ключ

В PostgreSQL современный вариант:

```sql
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

Разбор:

- `id` — имя колонки;
- `BIGINT` — большое целое число;
- `GENERATED ALWAYS AS IDENTITY` — PostgreSQL сам генерирует значение;
- `PRIMARY KEY` — это первичный ключ.

---

## 1.12. Практический пример модуля 1

Пока просто представим будущую таблицу покупателей:

```sql
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(30),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Пока не переживай, если не всё понятно. В следующем модуле разберём создание базы и таблиц.

---

## 1.13. Частые ошибки новичков

### Ошибка 1. Хранить всё текстом

Плохо:

```sql
price TEXT
created_at TEXT
is_active TEXT
```

Почему плохо:

- нельзя нормально сортировать числа;
- нельзя корректно сравнивать даты;
- нельзя проверять `TRUE` / `FALSE`.

Лучше:

```sql
price NUMERIC(10, 2)
created_at TIMESTAMPTZ
is_active BOOLEAN
```

---

### Ошибка 2. Не делать primary key

Плохо:

```sql
CREATE TABLE customers (
    first_name VARCHAR(100),
    email VARCHAR(255)
);
```

Лучше:

```sql
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(100),
    email VARCHAR(255)
);
```

---

### Ошибка 3. Использовать FLOAT для денег

Плохо:

```sql
price FLOAT
```

Лучше:

```sql
price NUMERIC(10, 2)
```

---

# Модуль 2. Первая база и DDL

DDL означает **Data Definition Language**.

Это команды, которые создают и изменяют структуру базы данных:

- `CREATE DATABASE`;
- `DROP DATABASE`;
- `CREATE TABLE`;
- `ALTER TABLE`;
- `DROP TABLE`.

Если DML — это работа с содержимым, то DDL — это работа с формой.

Аналогия:

- DDL — построить шкаф, добавить полку, убрать дверцу;
- DML — положить вещи в шкаф, переложить вещи, выбросить вещи.

---

## 2.1. Создание базы данных

Синтаксис:

```sql
CREATE DATABASE database_name;
```

Пример:

```sql
CREATE DATABASE online_shop;
```

Разбор:

- `CREATE` — создать;
- `DATABASE` — базу данных;
- `online_shop` — имя базы данных.

---

## 2.2. Удаление базы данных

Синтаксис:

```sql
DROP DATABASE database_name;
```

Пример:

```sql
DROP DATABASE online_shop;
```

Разбор:

- `DROP` — удалить объект;
- `DATABASE` — база данных;
- `online_shop` — имя базы.

⚠️ Важно: `DROP DATABASE` удаляет базу целиком со всеми таблицами и данными.

---

## 2.3. Безопасное удаление базы

```sql
DROP DATABASE IF EXISTS online_shop;
```

Разбор:

- `IF EXISTS` — удалить только если база существует.

Если базы нет, ошибки не будет.

---

## 2.4. Создание базы для нашего проекта

```sql
CREATE DATABASE online_shop;
```

Дальше нужно подключиться к базе `online_shop`.

Если ты используешь `psql`, команда подключения:

```sql
\c online_shop
```

Это не SQL-команда, а команда клиента `psql`.

---

## 2.5. Создание таблицы

Синтаксис:

```sql
CREATE TABLE table_name (
    column_name data_type constraints,
    column_name data_type constraints
);
```

Пример:

```sql
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(30),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 2.6. Разбор CREATE TABLE

```sql
CREATE TABLE customers
```

Создаём таблицу `customers`.

```sql
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

- `id` — имя колонки;
- `BIGINT` — тип;
- `GENERATED ALWAYS AS IDENTITY` — значение генерируется автоматически;
- `PRIMARY KEY` — первичный ключ.

```sql
first_name VARCHAR(100) NOT NULL
```

- `first_name` — имя;
- `VARCHAR(100)` — строка до 100 символов;
- `NOT NULL` — значение обязательно.

```sql
email VARCHAR(255) NOT NULL UNIQUE
```

- `email` обязателен;
- `UNIQUE` запрещает два одинаковых email.

```sql
is_active BOOLEAN NOT NULL DEFAULT TRUE
```

- логическое значение;
- не может быть `NULL`;
- если не передать значение, будет `TRUE`.

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
```

- дата и время с часовым поясом;
- по умолчанию текущий момент.

---

## 2.7. Ограничение NOT NULL

`NOT NULL` запрещает отсутствие значения.

```sql
first_name VARCHAR(100) NOT NULL
```

Плохо:

```sql
INSERT INTO customers (last_name, email)
VALUES ('Петров', 'ivan@example.com');
```

Ошибка, потому что не указан `first_name`.

---

## 2.8. Ограничение UNIQUE

`UNIQUE` запрещает дубли.

```sql
email VARCHAR(255) NOT NULL UNIQUE
```

Нельзя вставить двух клиентов с одинаковым email.

---

## 2.9. Ограничение DEFAULT

`DEFAULT` задаёт значение по умолчанию.

```sql
is_active BOOLEAN NOT NULL DEFAULT TRUE
```

Если при вставке не указать `is_active`, PostgreSQL поставит `TRUE`.

---

## 2.10. Ограничение CHECK

`CHECK` проверяет условие.

Пример:

```sql
price NUMERIC(10, 2) NOT NULL CHECK (price >= 0)
```

Это значит:

> Цена не может быть отрицательной.

---

## 2.11. Создаём таблицы интернет-магазина

### Таблица категорий

```sql
CREATE TABLE categories (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(150) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### Таблица товаров

```sql
CREATE TABLE products (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Пока категории с товарами не связываем. Это будет в модуле про внешние ключи.

---

### Таблица покупателей

```sql
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(30),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 2.12. ALTER TABLE

`ALTER TABLE` изменяет структуру таблицы.

---

## 2.13. Добавить колонку

Синтаксис:

```sql
ALTER TABLE table_name
ADD COLUMN column_name data_type constraints;
```

Пример:

```sql
ALTER TABLE customers
ADD COLUMN birth_date DATE;
```

Разбор:

- `ALTER TABLE customers` — изменить таблицу `customers`;
- `ADD COLUMN` — добавить колонку;
- `birth_date DATE` — колонка с датой рождения.

---

## 2.14. Удалить колонку

Синтаксис:

```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

Пример:

```sql
ALTER TABLE customers
DROP COLUMN birth_date;
```

---

## 2.15. Изменить тип колонки

Синтаксис:

```sql
ALTER TABLE table_name
ALTER COLUMN column_name TYPE new_data_type;
```

Пример:

```sql
ALTER TABLE customers
ALTER COLUMN phone TYPE VARCHAR(50);
```

Если PostgreSQL не может автоматически преобразовать тип, используется `USING`.

Пример:

```sql
ALTER TABLE products
ALTER COLUMN price TYPE NUMERIC(12, 2)
USING price::NUMERIC(12, 2);
```

---

## 2.16. Добавить ограничение

```sql
ALTER TABLE customers
ADD CONSTRAINT customers_phone_unique UNIQUE (phone);
```

Разбор:

- `ADD CONSTRAINT` — добавить ограничение;
- `customers_phone_unique` — имя ограничения;
- `UNIQUE (phone)` — телефон должен быть уникальным.

---

## 2.17. Удалить ограничение

```sql
ALTER TABLE customers
DROP CONSTRAINT customers_phone_unique;
```

---

## 2.18. Частые ошибки новичков

### Ошибка 1. Удалить базу, думая что удаляешь таблицу

```sql
DROP DATABASE online_shop;
```

Это удалит всю базу.

Удаление таблицы:

```sql
DROP TABLE customers;
```

---

### Ошибка 2. Создать колонку цены как INTEGER

Плохо:

```sql
price INTEGER
```

Тогда нельзя нормально хранить `99.90`.

Лучше:

```sql
price NUMERIC(10, 2)
```

---

### Ошибка 3. Забыть CHECK для количества

Плохо:

```sql
stock_quantity INTEGER
```

Лучше:

```sql
stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0)
```

---

# Модуль 3. Наполнение данными и DML

DML означает **Data Manipulation Language**.

Это команды для работы с данными:

- `INSERT`;
- `UPDATE`;
- `DELETE`.

---

## 3.1. INSERT INTO

`INSERT INTO` вставляет новые строки.

Синтаксис:

```sql
INSERT INTO table_name (column_1, column_2, column_3)
VALUES (value_1, value_2, value_3);
```

Разбор:

- `INSERT INTO` — вставить в таблицу;
- `table_name` — имя таблицы;
- список колонок — куда вставляем;
- `VALUES` — какие значения вставляем.

---

## 3.2. Одиночная вставка

```sql
INSERT INTO categories (name, description)
VALUES ('Электроника', 'Смартфоны, ноутбуки, наушники и другие устройства');
```

---

## 3.3. Множественная вставка

```sql
INSERT INTO categories (name, description)
VALUES
    ('Книги', 'Печатные и электронные книги'),
    ('Одежда', 'Мужская, женская и детская одежда'),
    ('Дом и кухня', 'Товары для дома, кухни и быта');
```

---

## 3.4. INSERT с RETURNING

`RETURNING` возвращает вставленные данные.

Это очень удобно в backend.

```sql
INSERT INTO customers (first_name, last_name, email, phone)
VALUES ('Иван', 'Петров', 'ivan.petrov@example.com', '+79990000001')
RETURNING id, first_name, last_name, email, created_at;
```

Зачем это нужно?

PostgreSQL сам создаёт `id`, а `RETURNING` позволяет сразу его получить.

---

## 3.5. Добавим покупателей

```sql
INSERT INTO customers (first_name, last_name, email, phone)
VALUES
    ('Иван', 'Петров', 'ivan.petrov@example.com', '+79990000001'),
    ('Анна', 'Смирнова', 'anna.smirnova@example.com', '+79990000002'),
    ('Сергей', 'Иванов', 'sergey.ivanov@example.com', '+79990000003'),
    ('Мария', 'Кузнецова', 'maria.kuznetsova@example.com', '+79990000004'),
    ('Ольга', 'Соколова', 'olga.sokolova@example.com', NULL);
```

---

## 3.6. Добавим товары

```sql
INSERT INTO products (name, description, price, stock_quantity)
VALUES
    ('Смартфон Pixel Pro', 'Смартфон с OLED-экраном и 256 ГБ памяти', 79990.00, 15),
    ('Ноутбук DevBook 14', 'Лёгкий ноутбук для разработки и работы', 129990.00, 8),
    ('Книга SQL для начинающих', 'Практическое руководство по SQL и базам данных', 1490.00, 100),
    ('Футболка PostgreSQL', 'Чёрная футболка с логотипом PostgreSQL', 1990.00, 50),
    ('Кофемолка Burr Mini', 'Компактная жерновая кофемолка для дома', 5990.00, 20);
```

---

## 3.7. UPDATE

`UPDATE` изменяет существующие строки.

Синтаксис:

```sql
UPDATE table_name
SET column_1 = value_1,
    column_2 = value_2
WHERE condition;
```

Разбор:

- `UPDATE table_name` — какую таблицу обновляем;
- `SET` — какие колонки меняем;
- `WHERE` — какие строки менять.

---

## 3.8. Обновить одного покупателя

```sql
UPDATE customers
SET phone = '+79991112233'
WHERE email = 'olga.sokolova@example.com';
```

---

## 3.9. UPDATE с RETURNING

```sql
UPDATE customers
SET is_active = FALSE
WHERE email = 'sergey.ivanov@example.com'
RETURNING id, first_name, last_name, email, is_active;
```

---

## 3.10. Самая опасная ошибка с UPDATE

Плохо:

```sql
UPDATE products
SET price = 0;
```

Это обновит **все товары**.

Правильно:

```sql
UPDATE products
SET price = 69990.00
WHERE name = 'Смартфон Pixel Pro';
```

Перед `UPDATE` полезно сначала сделать `SELECT` с тем же `WHERE`.

```sql
SELECT id, name, price
FROM products
WHERE name = 'Смартфон Pixel Pro';
```

Если выборка правильная — тогда делай `UPDATE`.

---

## 3.11. DELETE

`DELETE` удаляет строки.

Синтаксис:

```sql
DELETE FROM table_name
WHERE condition;
```

Пример:

```sql
DELETE FROM customers
WHERE email = 'sergey.ivanov@example.com';
```

---

## 3.12. DELETE с RETURNING

```sql
DELETE FROM customers
WHERE email = 'sergey.ivanov@example.com'
RETURNING id, first_name, last_name, email;
```

---

## 3.13. Опасность DELETE без WHERE

Плохо:

```sql
DELETE FROM products;
```

Это удалит все строки из таблицы `products`.

---

## 3.14. DELETE vs TRUNCATE

### DELETE

```sql
DELETE FROM products
WHERE id = 1;
```

Особенности:

- можно удалить часть строк;
- поддерживает `WHERE`;
- пишет удаление строк;
- срабатывают триггеры `DELETE`;
- можно использовать `RETURNING`.

---

### TRUNCATE

```sql
TRUNCATE TABLE products;
```

Особенности:

- удаляет все строки;
- не поддерживает `WHERE`;
- обычно быстрее;
- нельзя удалить только часть данных;
- можно сбросить identity-счётчик.

Пример со сбросом счётчика:

```sql
TRUNCATE TABLE products RESTART IDENTITY;
```

---

## 3.15. Частые ошибки новичков

### Ошибка 1. Не указывать список колонок в INSERT

Плохо:

```sql
INSERT INTO customers
VALUES (1, 'Иван', 'Петров', 'ivan@example.com', '+79990000001', TRUE, NOW());
```

Лучше:

```sql
INSERT INTO customers (first_name, last_name, email, phone)
VALUES ('Иван', 'Петров', 'ivan@example.com', '+79990000001');
```

Так код не сломается, если структура таблицы изменится.

---

### Ошибка 2. UPDATE без WHERE

```sql
UPDATE customers
SET is_active = FALSE;
```

Это отключит всех покупателей.

---

### Ошибка 3. DELETE без WHERE

```sql
DELETE FROM customers;
```

Это удалит всех покупателей.

---

# Модуль 4. SELECT — база

`SELECT` — главная команда SQL.

Она отвечает за выборку данных.

---

## 4.1. Базовый SELECT

Синтаксис:

```sql
SELECT column_1, column_2
FROM table_name;
```

Пример:

```sql
SELECT id, first_name, last_name, email
FROM customers;
```

Разбор:

- `SELECT` — какие колонки показать;
- `FROM` — из какой таблицы.

---

## 4.2. SELECT всех колонок

```sql
SELECT *
FROM customers;
```

`*` означает все колонки.

Для обучения удобно. В production-коде лучше явно перечислять колонки.

---

## 4.3. AS — псевдонимы

```sql
SELECT
    first_name AS имя,
    last_name AS фамилия,
    email AS почта
FROM customers;
```

`AS` переименовывает колонку в результате.

Можно без `AS`, но с `AS` понятнее:

```sql
SELECT
    first_name имя,
    last_name фамилия,
    email почта
FROM customers;
```

Лучше писать явно:

```sql
SELECT
    first_name AS имя,
    last_name AS фамилия,
    email AS почта
FROM customers;
```

---

## 4.4. WHERE

`WHERE` фильтрует строки.

Синтаксис:

```sql
SELECT columns
FROM table_name
WHERE condition;
```

Пример:

```sql
SELECT id, name, price
FROM products
WHERE price > 5000;
```

---

## 4.5. Операторы сравнения

### Равно

```sql
SELECT id, name, price
FROM products
WHERE name = 'Книга SQL для начинающих';
```

---

### Не равно

В PostgreSQL можно использовать:

```sql
SELECT id, name, price
FROM products
WHERE name != 'Книга SQL для начинающих';
```

Также стандартный вариант:

```sql
SELECT id, name, price
FROM products
WHERE name <> 'Книга SQL для начинающих';
```

---

### Больше

```sql
SELECT id, name, price
FROM products
WHERE price > 10000;
```

---

### Меньше

```sql
SELECT id, name, price
FROM products
WHERE price < 10000;
```

---

### Больше или равно

```sql
SELECT id, name, price
FROM products
WHERE price >= 5990.00;
```

---

### Меньше или равно

```sql
SELECT id, name, price
FROM products
WHERE price <= 5990.00;
```

---

## 4.6. IN

`IN` проверяет, входит ли значение в список.

```sql
SELECT id, name, price
FROM products
WHERE name IN ('Смартфон Pixel Pro', 'Ноутбук DevBook 14');
```

Аналог через `OR`:

```sql
SELECT id, name, price
FROM products
WHERE name = 'Смартфон Pixel Pro'
   OR name = 'Ноутбук DevBook 14';
```

---

## 4.7. BETWEEN

`BETWEEN` проверяет диапазон.

```sql
SELECT id, name, price
FROM products
WHERE price BETWEEN 1000 AND 10000;
```

Важно: `BETWEEN` включает границы.

Это то же самое:

```sql
SELECT id, name, price
FROM products
WHERE price >= 1000
  AND price <= 10000;
```

---

## 4.8. LIKE

`LIKE` ищет по шаблону с учётом регистра.

```sql
SELECT id, name
FROM products
WHERE name LIKE '%SQL%';
```

Символы:

- `%` — любое количество символов;
- `_` — один любой символ.

Пример:

```sql
SELECT id, name
FROM products
WHERE name LIKE 'Книга%';
```

Найдёт товары, которые начинаются с `Книга`.

---

## 4.9. ILIKE

`ILIKE` — поиск без учёта регистра. Это удобная фишка PostgreSQL.

```sql
SELECT id, name
FROM products
WHERE name ILIKE '%sql%';
```

Найдёт:

- `SQL`;
- `sql`;
- `Sql`;
- `sQl`.

---

## 4.10. AND, OR, NOT

### AND

Все условия должны быть истинны.

```sql
SELECT id, name, price, stock_quantity
FROM products
WHERE price > 1000
  AND stock_quantity > 10;
```

---

### OR

Хотя бы одно условие должно быть истинно.

```sql
SELECT id, name, price
FROM products
WHERE price < 2000
   OR stock_quantity > 80;
```

---

### NOT

Инвертирует условие.

```sql
SELECT id, name, is_active
FROM products
WHERE NOT is_active;
```

То же самое:

```sql
SELECT id, name, is_active
FROM products
WHERE is_active = FALSE;
```

---

## 4.11. Скобки в условиях

Опасно:

```sql
SELECT id, name, price, stock_quantity
FROM products
WHERE price < 2000
   OR price > 100000
  AND stock_quantity > 0;
```

`AND` имеет более высокий приоритет, чем `OR`.

Лучше писать явно:

```sql
SELECT id, name, price, stock_quantity
FROM products
WHERE (price < 2000 OR price > 100000)
  AND stock_quantity > 0;
```

---

## 4.12. ORDER BY

Сортировка.

```sql
SELECT id, name, price
FROM products
ORDER BY price ASC;
```

- `ASC` — по возрастанию;
- `DESC` — по убыванию.

```sql
SELECT id, name, price
FROM products
ORDER BY price DESC;
```

---

## 4.13. Сортировка по нескольким колонкам

```sql
SELECT id, name, price, stock_quantity
FROM products
ORDER BY price DESC, stock_quantity ASC;
```

Сначала сортировка по цене от дорогих к дешёвым. Если цены одинаковые — по количеству от меньшего к большему.

---

## 4.14. LIMIT

Ограничивает количество строк.

```sql
SELECT id, name, price
FROM products
ORDER BY price DESC
LIMIT 3;
```

Покажет 3 самых дорогих товара.

---

## 4.15. OFFSET

Пропускает строки.

```sql
SELECT id, name, price
FROM products
ORDER BY id ASC
LIMIT 2 OFFSET 2;
```

Это похоже на страницы:

- `LIMIT 2 OFFSET 0` — первая страница;
- `LIMIT 2 OFFSET 2` — вторая страница;
- `LIMIT 2 OFFSET 4` — третья страница.

---

## 4.16. Частые ошибки новичков

### Ошибка 1. Сравнивать NULL через `=`

Плохо:

```sql
SELECT id, first_name, phone
FROM customers
WHERE phone = NULL;
```

Правильно:

```sql
SELECT id, first_name, phone
FROM customers
WHERE phone IS NULL;
```

Для не `NULL`:

```sql
SELECT id, first_name, phone
FROM customers
WHERE phone IS NOT NULL;
```

---

### Ошибка 2. Забывать кавычки вокруг строк

Плохо:

```sql
SELECT id, name
FROM products
WHERE name = Книга SQL для начинающих;
```

Правильно:

```sql
SELECT id, name
FROM products
WHERE name = 'Книга SQL для начинающих';
```

---

### Ошибка 3. Путать порядок выполнения

Логический порядок примерно такой:

1. `FROM`
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. `SELECT`
6. `ORDER BY`
7. `LIMIT`

Поэтому алиас из `SELECT` не всегда доступен в `WHERE`.

---

# Модуль 5. Связи между таблицами и Foreign Keys

Пока наши таблицы существуют отдельно.

Но в реальной базе данные связаны.

Например:

- товар принадлежит категории;
- заказ принадлежит покупателю;
- заказ содержит товары.

---

## 5.1. Типы связей

### Один-к-одному

Одна строка в таблице A соответствует одной строке в таблице B.

Пример:

- `users`;
- `user_profiles`.

Один пользователь — один профиль.

```text
users 1 ─── 1 user_profiles
```

---

### Один-ко-многим

Одна строка в таблице A соответствует многим строкам в таблице B.

Пример:

- один покупатель может иметь много заказов;
- один заказ принадлежит одному покупателю.

```text
customers 1 ─── N orders
```

---

### Многие-ко-многим

Много строк в таблице A связано со многими строками в таблице B.

Пример:

- один заказ содержит много товаров;
- один товар может быть в разных заказах.

Нужна промежуточная таблица.

```text
orders 1 ─── N order_items N ─── 1 products
```

---

## 5.2. Foreign Key

**Foreign Key**, внешний ключ, — это колонка, которая ссылается на primary key другой таблицы.

Пример:

```sql
customer_id BIGINT NOT NULL REFERENCES customers(id)
```

Это значит:

> В `customer_id` можно записать только такой id, который существует в `customers.id`.

---

## 5.3. Добавим связь товара с категорией

Сейчас таблица `products` без категории.

Добавим колонку:

```sql
ALTER TABLE products
ADD COLUMN category_id BIGINT;
```

Теперь добавим внешний ключ:

```sql
ALTER TABLE products
ADD CONSTRAINT products_category_id_fkey
FOREIGN KEY (category_id)
REFERENCES categories(id);
```

Разбор:

- `ALTER TABLE products` — меняем таблицу товаров;
- `ADD CONSTRAINT products_category_id_fkey` — добавляем ограничение с именем;
- `FOREIGN KEY (category_id)` — колонка является внешним ключом;
- `REFERENCES categories(id)` — ссылается на `categories.id`.

---

## 5.4. Обновим товары, назначим категории

Сначала посмотрим категории:

```sql
SELECT id, name
FROM categories
ORDER BY id;
```

Предположим, данные такие:

| id | name         |
|----|--------------|
| 1  | Электроника  |
| 2  | Книги        |
| 3  | Одежда       |
| 4  | Дом и кухня  |

Назначим категории:

```sql
UPDATE products
SET category_id = 1
WHERE name IN ('Смартфон Pixel Pro', 'Ноутбук DevBook 14');

UPDATE products
SET category_id = 2
WHERE name = 'Книга SQL для начинающих';

UPDATE products
SET category_id = 3
WHERE name = 'Футболка PostgreSQL';

UPDATE products
SET category_id = 4
WHERE name = 'Кофемолка Burr Mini';
```

Теперь можно сделать колонку обязательной:

```sql
ALTER TABLE products
ALTER COLUMN category_id SET NOT NULL;
```

---

## 5.5. Создадим таблицу заказов

```sql
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'new',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT orders_customer_id_fkey
        FOREIGN KEY (customer_id)
        REFERENCES customers(id),
    CONSTRAINT orders_status_check
        CHECK (status IN ('new', 'paid', 'shipped', 'completed', 'cancelled'))
);
```

---

## 5.6. Создадим таблицу позиций заказа

```sql
CREATE TABLE order_items (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10, 2) NOT NULL CHECK (unit_price >= 0),
    CONSTRAINT order_items_order_id_fkey
        FOREIGN KEY (order_id)
        REFERENCES orders(id),
    CONSTRAINT order_items_product_id_fkey
        FOREIGN KEY (product_id)
        REFERENCES products(id),
    CONSTRAINT order_items_order_product_unique
        UNIQUE (order_id, product_id)
);
```

Почему есть `unit_price`, если цена есть в `products`?

Потому что цена товара может измениться завтра, а в заказе должна остаться цена на момент покупки.

---

## 5.7. ON DELETE

Когда родительская строка удаляется, нужно решить, что делать с дочерними.

Пример:

- удаляем покупателя;
- что делать с его заказами?

---

## 5.8. ON DELETE RESTRICT / NO ACTION

По умолчанию PostgreSQL не даст удалить родителя, если на него есть ссылки.

```sql
DELETE FROM customers
WHERE id = 1;
```

Если у покупателя есть заказы, будет ошибка.

Это защита от потери данных.

---

## 5.9. ON DELETE CASCADE

Удалить дочерние строки автоматически.

Пример:

```sql
CREATE TABLE customer_sessions (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    session_token VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT customer_sessions_customer_id_fkey
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
        ON DELETE CASCADE
);
```

Если удалить покупателя, его сессии удалятся автоматически.

Подходит для:

- временных токенов;
- сессий;
- логов, которые не нужны без владельца.

Опасно для:

- заказов;
- платежей;
- финансовых операций.

---

## 5.10. ON DELETE SET NULL

При удалении родителя поставить `NULL`.

Пример:

```sql
CREATE TABLE customer_notes (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT,
    note TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT customer_notes_customer_id_fkey
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
        ON DELETE SET NULL
);
```

Если покупателя удалить, заметка останется, но `customer_id` станет `NULL`.

---

## 5.11. Практика: создадим заказы

Сначала посмотрим клиентов и товары:

```sql
SELECT id, first_name, last_name, email
FROM customers
ORDER BY id;

SELECT id, name, price
FROM products
ORDER BY id;
```

Создадим заказ для клиента с `id = 1`:

```sql
INSERT INTO orders (customer_id, status)
VALUES (1, 'new')
RETURNING id, customer_id, status, created_at;
```

Предположим, вернулся `id = 1`.

Добавим позиции:

```sql
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES
    (1, 1, 1, 79990.00),
    (1, 3, 2, 1490.00);
```

Создадим второй заказ:

```sql
INSERT INTO orders (customer_id, status)
VALUES (2, 'paid')
RETURNING id, customer_id, status, created_at;
```

Предположим, вернулся `id = 2`.

```sql
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES
    (2, 2, 1, 129990.00),
    (2, 5, 1, 5990.00);
```

---

## 5.12. Как внешний ключ защищает данные

Попробуем вставить заказ для несуществующего клиента:

```sql
INSERT INTO orders (customer_id, status)
VALUES (999999, 'new');
```

PostgreSQL выдаст ошибку, потому что клиента `999999` нет.

Это хорошо. База защищает нас от мусора.

---

## 5.13. Частые ошибки новичков

### Ошибка 1. Не создавать внешние ключи

Плохо:

```sql
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL
);
```

Так можно вставить заказ на несуществующего клиента.

Лучше:

```sql
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    CONSTRAINT orders_customer_id_fkey
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

---

### Ошибка 2. Ставить CASCADE везде

`ON DELETE CASCADE` удобно, но может случайно удалить полмагазина данных.

Для заказов чаще безопаснее запрещать удаление покупателя, если у него есть заказы.

---

# Модуль 6. JOIN — объединение таблиц

`JOIN` нужен, чтобы получать данные из нескольких таблиц.

---

## 6.1. Зачем нужен JOIN

У нас есть `orders`:

| id | customer_id | status |
|----|-------------|--------|
| 1  | 1           | new    |
| 2  | 2           | paid   |

И `customers`:

| id | first_name | email                    |
|----|------------|--------------------------|
| 1  | Иван       | ivan.petrov@example.com  |
| 2  | Анна       | anna.smirnova@example.com |

Если хотим увидеть заказ вместе с именем покупателя, нужен `JOIN`.

---

## 6.2. Как JOIN работает на пальцах

```sql
orders.customer_id = customers.id
```

PostgreSQL берёт строку заказа и ищет клиента, у которого `customers.id` равен `orders.customer_id`.

---

## 6.3. INNER JOIN

Возвращает только строки, где есть совпадение в обеих таблицах.

```text
A ∩ B
```

```sql
SELECT
    orders.id AS order_id,
    orders.status AS order_status,
    customers.first_name,
    customers.last_name,
    customers.email
FROM orders
INNER JOIN customers
    ON orders.customer_id = customers.id;
```

Разбор:

- `FROM orders` — главная таблица;
- `INNER JOIN customers` — присоединяем клиентов;
- `ON orders.customer_id = customers.id` — условие связи.

---

## 6.4. Алиасы таблиц

Чтобы запрос был короче:

```sql
SELECT
    o.id AS order_id,
    o.status AS order_status,
    c.first_name,
    c.last_name,
    c.email
FROM orders AS o
INNER JOIN customers AS c
    ON o.customer_id = c.id;
```

- `orders AS o` — таблица `orders` теперь коротко `o`;
- `customers AS c` — таблица `customers` теперь коротко `c`.

---

## 6.5. LEFT JOIN

Возвращает все строки из левой таблицы и совпадения из правой.

Если совпадения нет — справа будет `NULL`.

```text
A + совпавшая часть B
```

Пример: показать все категории и товары в них.

```sql
SELECT
    c.id AS category_id,
    c.name AS category_name,
    p.id AS product_id,
    p.name AS product_name,
    p.price
FROM categories AS c
LEFT JOIN products AS p
    ON p.category_id = c.id
ORDER BY c.id, p.id;
```

Если в категории нет товаров, категория всё равно появится.

---

## 6.6. RIGHT JOIN

Возвращает все строки из правой таблицы и совпадения из левой.

```sql
SELECT
    c.id AS category_id,
    c.name AS category_name,
    p.id AS product_id,
    p.name AS product_name
FROM products AS p
RIGHT JOIN categories AS c
    ON p.category_id = c.id
ORDER BY c.id, p.id;
```

На практике `RIGHT JOIN` используют редко. Почти всегда можно переписать через `LEFT JOIN`, поменяв таблицы местами.

---

## 6.7. FULL OUTER JOIN

Возвращает все строки из обеих таблиц:

- совпали — соединяет;
- не совпали слева — справа `NULL`;
- не совпали справа — слева `NULL`.

```sql
SELECT
    c.id AS category_id,
    c.name AS category_name,
    p.id AS product_id,
    p.name AS product_name
FROM categories AS c
FULL OUTER JOIN products AS p
    ON p.category_id = c.id
ORDER BY c.id, p.id;
```

---

## 6.8. JOIN трёх и более таблиц

Покажем заказы с покупателями, товарами и суммами позиций.

```sql
SELECT
    o.id AS order_id,
    o.status AS order_status,
    o.created_at AS order_created_at,
    c.first_name,
    c.last_name,
    c.email,
    p.name AS product_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS item_total
FROM orders AS o
INNER JOIN customers AS c
    ON o.customer_id = c.id
INNER JOIN order_items AS oi
    ON oi.order_id = o.id
INNER JOIN products AS p
    ON oi.product_id = p.id
ORDER BY o.id, oi.id;
```

Логика:

```text
orders -> customers
orders -> order_items
order_items -> products
```

---

## 6.9. Найти покупателей без заказов

```sql
SELECT
    c.id,
    c.first_name,
    c.last_name,
    c.email
FROM customers AS c
LEFT JOIN orders AS o
    ON o.customer_id = c.id
WHERE o.id IS NULL;
```

Это классический паттерн:

> LEFT JOIN + WHERE right_table.id IS NULL

---

## 6.10. Частые ошибки новичков

### Ошибка 1. Забыть ON

Плохо:

```sql
SELECT
    o.id,
    c.email
FROM orders AS o
INNER JOIN customers AS c;
```

Так получится некорректное соединение или ошибка в зависимости от синтаксиса. Если сделать `CROSS JOIN`, будет каждая строка с каждой.

---

### Ошибка 2. Фильтровать правую таблицу в WHERE после LEFT JOIN

Допустим, хотим все категории и только активные товары.

Опасно:

```sql
SELECT
    c.name AS category_name,
    p.name AS product_name
FROM categories AS c
LEFT JOIN products AS p
    ON p.category_id = c.id
WHERE p.is_active = TRUE;
```

Так `LEFT JOIN` фактически превращается в `INNER JOIN`, потому что строки без товара имеют `p.is_active = NULL`.

Лучше:

```sql
SELECT
    c.name AS category_name,
    p.name AS product_name
FROM categories AS c
LEFT JOIN products AS p
    ON p.category_id = c.id
   AND p.is_active = TRUE;
```

---

# Модуль 7. Агрегатные функции и GROUP BY

Агрегаты считают данные по множеству строк.

---

## 7.1. COUNT

Количество строк.

```sql
SELECT COUNT(*) AS total_customers
FROM customers;
```

`COUNT(*)` считает все строки.

---

## 7.2. COUNT(column)

```sql
SELECT COUNT(phone) AS customers_with_phone
FROM customers;
```

`COUNT(phone)` считает только строки, где `phone IS NOT NULL`.

---

## 7.3. SUM

Сумма.

```sql
SELECT SUM(stock_quantity) AS total_stock
FROM products;
```

---

## 7.4. AVG

Среднее значение.

```sql
SELECT AVG(price) AS average_price
FROM products;
```

---

## 7.5. MIN и MAX

```sql
SELECT
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM products;
```

---

## 7.6. GROUP BY

`GROUP BY` группирует строки.

Аналогия:

Ты раскладываешь чеки по папкам:

- папка "Иван";
- папка "Анна";
- папка "Мария".

Потом считаешь сумму по каждой папке.

---

## 7.7. Количество товаров по категориям

```sql
SELECT
    c.name AS category_name,
    COUNT(p.id) AS product_count
FROM categories AS c
LEFT JOIN products AS p
    ON p.category_id = c.id
GROUP BY c.id, c.name
ORDER BY c.name;
```

Почему `GROUP BY c.id, c.name`?

Потому что в `SELECT` есть обычная колонка `c.name`, не агрегат. PostgreSQL должен понимать, по каким группам её показывать.

---

## 7.8. Сумма каждого заказа

```sql
SELECT
    o.id AS order_id,
    o.status,
    SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders AS o
INNER JOIN order_items AS oi
    ON oi.order_id = o.id
GROUP BY o.id, o.status
ORDER BY o.id;
```

---

## 7.9. Статистика по покупателям

```sql
SELECT
    c.id AS customer_id,
    c.first_name,
    c.last_name,
    COUNT(o.id) AS orders_count,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS total_spent
FROM customers AS c
LEFT JOIN orders AS o
    ON o.customer_id = c.id
LEFT JOIN order_items AS oi
    ON oi.order_id = o.id
GROUP BY c.id, c.first_name, c.last_name
ORDER BY total_spent DESC;
```

`COALESCE` возвращает первое не-NULL значение.

Если у клиента нет заказов, `SUM(...)` будет `NULL`, а мы хотим `0`.

```sql
COALESCE(SUM(oi.quantity * oi.unit_price), 0)
```

---

## 7.10. WHERE vs HAVING

Главный затык новичков.

### WHERE

Фильтрует строки **до группировки**.

```sql
SELECT
    status,
    COUNT(*) AS orders_count
FROM orders
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY status;
```

Сначала берём только заказы за последние 30 дней, потом группируем.

---

### HAVING

Фильтрует группы **после группировки**.

```sql
SELECT
    o.customer_id,
    COUNT(o.id) AS orders_count
FROM orders AS o
GROUP BY o.customer_id
HAVING COUNT(o.id) >= 2;
```

Сначала группируем заказы по клиенту, потом оставляем только тех, у кого заказов минимум 2.

---

## 7.11. WHERE и HAVING вместе

```sql
SELECT
    o.customer_id,
    COUNT(o.id) AS paid_orders_count
FROM orders AS o
WHERE o.status = 'paid'
GROUP BY o.customer_id
HAVING COUNT(o.id) >= 1;
```

Логика:

1. `WHERE o.status = 'paid'` — берём только оплаченные заказы.
2. `GROUP BY o.customer_id` — группируем по клиенту.
3. `HAVING COUNT(o.id) >= 1` — оставляем клиентов с минимум одним оплаченным заказом.

---

## 7.12. Частые ошибки новичков

### Ошибка 1. Использовать агрегат в WHERE

Плохо:

```sql
SELECT
    customer_id,
    COUNT(*) AS orders_count
FROM orders
WHERE COUNT(*) > 1
GROUP BY customer_id;
```

Правильно:

```sql
SELECT
    customer_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

---

### Ошибка 2. Выбрать колонку без GROUP BY

Плохо:

```sql
SELECT
    status,
    customer_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY status;
```

`customer_id` не агрегирован и не указан в `GROUP BY`.

Правильно:

```sql
SELECT
    status,
    customer_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY status, customer_id;
```

---

# Модуль 8. Подзапросы и CTE

Подзапрос — запрос внутри другого запроса.

---

## 8.1. Подзапрос в WHERE

Найти товары дороже средней цены.

```sql
SELECT
    id,
    name,
    price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
)
ORDER BY price DESC;
```

Внутренний запрос:

```sql
SELECT AVG(price)
FROM products
```

возвращает одно число.

Внешний запрос сравнивает `price` с этим числом.

---

## 8.2. Подзапрос в SELECT

Показать каждый товар и среднюю цену по всем товарам.

```sql
SELECT
    id,
    name,
    price,
    (
        SELECT AVG(price)
        FROM products
    ) AS average_product_price
FROM products
ORDER BY price DESC;
```

---

## 8.3. Подзапрос в FROM

Подзапрос в `FROM` становится временной таблицей.

```sql
SELECT
    order_totals.order_id,
    order_totals.order_total
FROM (
    SELECT
        oi.order_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM order_items AS oi
    GROUP BY oi.order_id
) AS order_totals
WHERE order_totals.order_total > 10000
ORDER BY order_totals.order_total DESC;
```

Важно: подзапрос в `FROM` должен иметь алиас:

```sql
) AS order_totals
```

---

## 8.4. EXISTS

`EXISTS` проверяет, существует ли хотя бы одна строка.

Найти клиентов, у которых есть заказы:

```sql
SELECT
    c.id,
    c.first_name,
    c.last_name,
    c.email
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.id
);
```

`SELECT 1` означает:

> Нам не важны данные, важен факт существования строки.

---

## 8.5. NOT EXISTS

Найти клиентов без заказов:

```sql
SELECT
    c.id,
    c.first_name,
    c.last_name,
    c.email
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.id
);
```

---

## 8.6. ANY

`ANY` сравнивает значение хотя бы с одним значением из набора.

Найти товары дороже хотя бы одного товара из категории "Книги":

```sql
SELECT
    p.id,
    p.name,
    p.price
FROM products AS p
WHERE p.price > ANY (
    SELECT p2.price
    FROM products AS p2
    INNER JOIN categories AS c2
        ON p2.category_id = c2.id
    WHERE c2.name = 'Книги'
);
```

Если в категории "Книги" есть товар за `1490`, то товары дороже `1490` подойдут.

---

## 8.7. ALL

`ALL` сравнивает значение со всеми значениями из набора.

Найти товары дороже всех товаров из категории "Книги":

```sql
SELECT
    p.id,
    p.name,
    p.price
FROM products AS p
WHERE p.price > ALL (
    SELECT p2.price
    FROM products AS p2
    INNER JOIN categories AS c2
        ON p2.category_id = c2.id
    WHERE c2.name = 'Книги'
);
```

---

## 8.8. CTE — Common Table Expression

CTE — временный именованный результат запроса.

Синтаксис:

```sql
WITH cte_name AS (
    SELECT columns
    FROM table_name
)
SELECT columns
FROM cte_name;
```

---

## 8.9. Зачем нужен CTE

Без CTE сложный запрос превращается в матрёшку.

CTE позволяет писать шагами:

1. сначала считаем суммы заказов;
2. потом присоединяем клиентов;
3. потом фильтруем.

---

## 8.10. Пример CTE

```sql
WITH order_totals AS (
    SELECT
        oi.order_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM order_items AS oi
    GROUP BY oi.order_id
)
SELECT
    o.id AS order_id,
    o.status,
    c.first_name,
    c.last_name,
    order_totals.order_total
FROM orders AS o
INNER JOIN customers AS c
    ON o.customer_id = c.id
INNER JOIN order_totals
    ON order_totals.order_id = o.id
WHERE order_totals.order_total > 10000
ORDER BY order_totals.order_total DESC;
```

---

## 8.11. Несколько CTE

```sql
WITH order_totals AS (
    SELECT
        oi.order_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM order_items AS oi
    GROUP BY oi.order_id
),
customer_totals AS (
    SELECT
        o.customer_id,
        COUNT(o.id) AS orders_count,
        SUM(order_totals.order_total) AS total_spent
    FROM orders AS o
    INNER JOIN order_totals
        ON order_totals.order_id = o.id
    GROUP BY o.customer_id
)
SELECT
    c.id AS customer_id,
    c.first_name,
    c.last_name,
    c.email,
    customer_totals.orders_count,
    customer_totals.total_spent
FROM customer_totals
INNER JOIN customers AS c
    ON customer_totals.customer_id = c.id
ORDER BY customer_totals.total_spent DESC;
```

---

## 8.12. Частые ошибки новичков

### Ошибка 1. Подзапрос возвращает много строк там, где нужна одна

Плохо:

```sql
SELECT
    id,
    name,
    price
FROM products
WHERE price > (
    SELECT price
    FROM products
);
```

Внутренний запрос возвращает много цен.

Правильно использовать агрегат:

```sql
SELECT
    id,
    name,
    price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

Или `ANY` / `ALL`.

---

### Ошибка 2. Не дать алиас подзапросу в FROM

Плохо:

```sql
SELECT order_id
FROM (
    SELECT order_id
    FROM order_items
);
```

Правильно:

```sql
SELECT order_items_subquery.order_id
FROM (
    SELECT order_id
    FROM order_items
) AS order_items_subquery;
```

---

# Модуль 9. Встроенные фишки PostgreSQL

---

## 9.1. Работа со строками

### CONCAT

Склеивает строки.

```sql
SELECT
    id,
    CONCAT(first_name, ' ', last_name) AS full_name
FROM customers;
```

---

### Оператор `||`

В PostgreSQL строки можно склеивать так:

```sql
SELECT
    id,
    first_name || ' ' || last_name AS full_name
FROM customers;
```

Но если одно значение `NULL`, результат может стать `NULL`.

Безопаснее:

```sql
SELECT
    id,
    CONCAT(first_name, ' ', last_name) AS full_name
FROM customers;
```

---

### SUBSTR / SUBSTRING

Берёт часть строки.

```sql
SELECT
    email,
    SUBSTR(email, 1, 5) AS first_five_chars
FROM customers;
```

---

### UPPER

Переводит в верхний регистр.

```sql
SELECT
    first_name,
    UPPER(first_name) AS first_name_upper
FROM customers;
```

---

### LOWER

Переводит в нижний регистр.

```sql
SELECT
    email,
    LOWER(email) AS email_lower
FROM customers;
```

Полезно для нормализации email.

---

## 9.2. Работа с датами и временем

### NOW()

Текущая дата и время.

```sql
SELECT NOW() AS current_datetime;
```

---

### INTERVAL

Интервал времени.

```sql
SELECT NOW() - INTERVAL '7 days' AS seven_days_ago;
```

Примеры:

```sql
SELECT NOW() + INTERVAL '1 hour' AS plus_one_hour;

SELECT NOW() + INTERVAL '30 minutes' AS plus_thirty_minutes;

SELECT NOW() - INTERVAL '1 month' AS one_month_ago;
```

---

## 9.3. Заказы за последние 7 дней

```sql
SELECT
    id,
    customer_id,
    status,
    created_at
FROM orders
WHERE created_at >= NOW() - INTERVAL '7 days'
ORDER BY created_at DESC;
```

---

## 9.4. EXTRACT

Достаёт часть даты.

```sql
SELECT
    id,
    created_at,
    EXTRACT(YEAR FROM created_at) AS order_year,
    EXTRACT(MONTH FROM created_at) AS order_month,
    EXTRACT(DAY FROM created_at) AS order_day
FROM orders;
```

---

## 9.5. DATE_TRUNC

Обрезает дату до нужной точности.

```sql
SELECT
    DATE_TRUNC('day', created_at) AS order_day,
    COUNT(*) AS orders_count
FROM orders
GROUP BY DATE_TRUNC('day', created_at)
ORDER BY order_day;
```

Примеры точности:

```sql
SELECT DATE_TRUNC('hour', NOW()) AS current_hour;

SELECT DATE_TRUNC('day', NOW()) AS current_day;

SELECT DATE_TRUNC('month', NOW()) AS current_month;

SELECT DATE_TRUNC('year', NOW()) AS current_year;
```

---

## 9.6. CASE WHEN

`CASE WHEN` — это `if-else` внутри SQL.

Синтаксис:

```sql
CASE
    WHEN condition_1 THEN result_1
    WHEN condition_2 THEN result_2
    ELSE default_result
END
```

---

## 9.7. Категории товаров по цене

```sql
SELECT
    id,
    name,
    price,
    CASE
        WHEN price < 2000 THEN 'дешёвый товар'
        WHEN price >= 2000 AND price < 20000 THEN 'средний товар'
        WHEN price >= 20000 THEN 'дорогой товар'
        ELSE 'неизвестно'
    END AS price_group
FROM products
ORDER BY price;
```

---

## 9.8. Человекочитаемый статус заказа

```sql
SELECT
    id,
    status,
    CASE
        WHEN status = 'new' THEN 'Новый заказ'
        WHEN status = 'paid' THEN 'Оплачен'
        WHEN status = 'shipped' THEN 'Отправлен'
        WHEN status = 'completed' THEN 'Завершён'
        WHEN status = 'cancelled' THEN 'Отменён'
        ELSE 'Неизвестный статус'
    END AS status_title
FROM orders
ORDER BY id;
```

---

## 9.9. CASE внутри агрегатов

Посчитать количество заказов по статусам в одну строку:

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(CASE WHEN status = 'new' THEN 1 ELSE 0 END) AS new_orders,
    SUM(CASE WHEN status = 'paid' THEN 1 ELSE 0 END) AS paid_orders,
    SUM(CASE WHEN status = 'shipped' THEN 1 ELSE 0 END) AS shipped_orders,
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) AS completed_orders,
    SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled_orders
FROM orders;
```

В PostgreSQL есть ещё красивый вариант через `FILTER`:

```sql
SELECT
    COUNT(*) AS total_orders,
    COUNT(*) FILTER (WHERE status = 'new') AS new_orders,
    COUNT(*) FILTER (WHERE status = 'paid') AS paid_orders,
    COUNT(*) FILTER (WHERE status = 'shipped') AS shipped_orders,
    COUNT(*) FILTER (WHERE status = 'completed') AS completed_orders,
    COUNT(*) FILTER (WHERE status = 'cancelled') AS cancelled_orders
FROM orders;
```

---

## 9.10. Частые ошибки новичков

### Ошибка 1. Путать NOW() и строку

Плохо:

```sql
SELECT 'NOW()';
```

Это просто текст `NOW()`.

Правильно:

```sql
SELECT NOW();
```

---

### Ошибка 2. Забыть END в CASE

Плохо:

```sql
SELECT
    CASE
        WHEN price > 10000 THEN 'дорого'
        ELSE 'нормально'
FROM products;
```

Правильно:

```sql
SELECT
    CASE
        WHEN price > 10000 THEN 'дорого'
        ELSE 'нормально'
    END AS price_label
FROM products;
```

---

# Итоговая схема нашей базы

```text
categories
    id PK
    name
    description
    created_at

products
    id PK
    category_id FK -> categories.id
    name
    description
    price
    stock_quantity
    is_active
    created_at

customers
    id PK
    first_name
    last_name
    email
    phone
    is_active
    created_at

orders
    id PK
    customer_id FK -> customers.id
    status
    created_at

order_items
    id PK
    order_id FK -> orders.id
    product_id FK -> products.id
    quantity
    unit_price
```

Связи:

```text
categories 1 ─── N products

customers 1 ─── N orders

orders 1 ─── N order_items

products 1 ─── N order_items
```

---

# Полный учебный SQL-скрипт для повторения

```sql
DROP DATABASE IF EXISTS online_shop;

CREATE DATABASE online_shop;
```

Подключись к базе `online_shop`, затем выполни:

```sql
CREATE TABLE categories (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(150) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(30),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE products (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    category_id BIGINT NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT products_category_id_fkey
        FOREIGN KEY (category_id)
        REFERENCES categories(id)
);

CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'new',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT orders_customer_id_fkey
        FOREIGN KEY (customer_id)
        REFERENCES customers(id),
    CONSTRAINT orders_status_check
        CHECK (status IN ('new', 'paid', 'shipped', 'completed', 'cancelled'))
);

CREATE TABLE order_items (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10, 2) NOT NULL CHECK (unit_price >= 0),
    CONSTRAINT order_items_order_id_fkey
        FOREIGN KEY (order_id)
        REFERENCES orders(id),
    CONSTRAINT order_items_product_id_fkey
        FOREIGN KEY (product_id)
        REFERENCES products(id),
    CONSTRAINT order_items_order_product_unique
        UNIQUE (order_id, product_id)
);

INSERT INTO categories (name, description)
VALUES
    ('Электроника', 'Смартфоны, ноутбуки, наушники и другие устройства'),
    ('Книги', 'Печатные и электронные книги'),
    ('Одежда', 'Мужская, женская и детская одежда'),
    ('Дом и кухня', 'Товары для дома, кухни и быта');

INSERT INTO customers (first_name, last_name, email, phone)
VALUES
    ('Иван', 'Петров', 'ivan.petrov@example.com', '+79990000001'),
    ('Анна', 'Смирнова', 'anna.smirnova@example.com', '+79990000002'),
    ('Сергей', 'Иванов', 'sergey.ivanov@example.com', '+79990000003'),
    ('Мария', 'Кузнецова', 'maria.kuznetsova@example.com', '+79990000004'),
    ('Ольга', 'Соколова', 'olga.sokolova@example.com', NULL);

INSERT INTO products (category_id, name, description, price, stock_quantity)
VALUES
    (1, 'Смартфон Pixel Pro', 'Смартфон с OLED-экраном и 256 ГБ памяти', 79990.00, 15),
    (1, 'Ноутбук DevBook 14', 'Лёгкий ноутбук для разработки и работы', 129990.00, 8),
    (2, 'Книга SQL для начинающих', 'Практическое руководство по SQL и базам данных', 1490.00, 100),
    (3, 'Футболка PostgreSQL', 'Чёрная футболка с логотипом PostgreSQL', 1990.00, 50),
    (4, 'Кофемолка Burr Mini', 'Компактная жерновая кофемолка для дома', 5990.00, 20);

INSERT INTO orders (customer_id, status)
VALUES
    (1, 'new'),
    (2, 'paid'),
    (1, 'paid'),
    (3, 'cancelled');

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES
    (1, 1, 1, 79990.00),
    (1, 3, 2, 1490.00),
    (2, 2, 1, 129990.00),
    (2, 5, 1, 5990.00),
    (3, 4, 3, 1990.00),
    (4, 3, 1, 1490.00);
```

---

# Мини-чеклист Junior SQL/PostgreSQL

Ты должен уверенно понимать и уметь:

- создать базу данных;
- создать таблицы;
- выбрать правильные типы данных;
- объяснить `PRIMARY KEY`;
- объяснить `FOREIGN KEY`;
- использовать `NOT NULL`, `UNIQUE`, `DEFAULT`, `CHECK`;
- вставлять данные через `INSERT`;
- обновлять через `UPDATE` с `WHERE`;
- удалять через `DELETE` с `WHERE`;
- понимать отличие `DELETE` и `TRUNCATE`;
- писать `SELECT`;
- фильтровать через `WHERE`;
- использовать `IN`, `BETWEEN`, `LIKE`, `ILIKE`;
- сортировать через `ORDER BY`;
- ограничивать через `LIMIT` и `OFFSET`;
- соединять таблицы через `JOIN`;
- понимать `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`;
- группировать через `GROUP BY`;
- фильтровать группы через `HAVING`;
- писать подзапросы;
- использовать `EXISTS`;
- писать CTE через `WITH`;
- работать со строками;
- работать с датами;
- использовать `CASE WHEN`.

Если всё это понятно — у тебя уже крепкая база уровня **Base/Junior** по SQL и PostgreSQL.
