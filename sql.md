# 🗄️ ПОЛНАЯ ШПАРГАЛКА ПО SQL

## 📋 ОГЛАВЛЕНИЕ
1. [Типы данных в SQL](#типы-данных-в-sql)
2. [Ограничения (Constraints)](#ограничения-constraints)
3. [DDL - Data Definition Language](#ddl---data-definition-language)
4. [DML - Data Manipulation Language](#dml---data-manipulation-language)
5. [DQL - Data Query Language](#dql---data-query-language)
6. [JOIN - Соединения таблиц](#join---соединения-таблиц)
7. [Подзапросы](#подзапросы)
8. [Транзакции](#транзакции)
9. [Индексы](#индексы)
10. [Практические примеры](#практические-примеры)

---

## ТИПЫ ДАННЫХ В SQL

### **ЧИСЛОВЫЕ ТИПЫ ДАННЫХ**

#### **ЦЕЛЫЕ ЧИСЛА**
```sql
-- TINYINT: от -128 до 127 (или 0 до 255 для UNSIGNED)
CREATE TABLE example_tinyint (
    age TINYINT,                    -- от -128 до 127
    positive_age TINYINT UNSIGNED   -- от 0 до 255
);

-- SMALLINT: от -32,768 до 32,767
CREATE TABLE example_smallint (
    product_count SMALLINT,
    max_users SMALLINT UNSIGNED     -- от 0 до 65,535
);

-- MEDIUMINT: от -8,388,608 до 8,388,607
CREATE TABLE example_mediumint (
    city_population MEDIUMINT UNSIGNED  -- от 0 до 16,777,215
);

-- INT (INTEGER): от -2,147,483,648 до 2,147,483,647
CREATE TABLE example_int (
    user_id INT PRIMARY KEY,
    company_id INT UNSIGNED         -- от 0 до 4,294,967,295
);

-- BIGINT: очень большие числа
CREATE TABLE example_bigint (
    global_id BIGINT,               -- от -9,223,372,036,854,775,808 до 9,223,372,036,854,775,807
    website_visits BIGINT UNSIGNED  -- от 0 до 18,446,744,073,709,551,615
);
```

#### **ЧИСЛА С ПЛАВАЮЩЕЙ ТОЧКОЙ**
```sql
-- FLOAT: приблизительные числа, 4 байта
CREATE TABLE example_float (
    temperature FLOAT,              -- примерно 7 цифр точности
    probability FLOAT(5,2)          -- всего 5 цифр, 2 после запятой
);

-- DOUBLE: более точные числа, 8 байт
CREATE TABLE example_double (
    scientific_value DOUBLE,        -- примерно 15 цифр точности
    precise_measurement DOUBLE(10,4)-- всего 10 цифр, 4 после запятой
);

-- DECIMAL/NUMERIC: точные числа (для денег)
CREATE TABLE example_decimal (
    -- DECIMAL(общее_количество_цифр, цифр_после_запятой)
    price DECIMAL(10, 2),           -- всего 10 цифр, 2 после запятой: 99999999.99
    tax_rate DECIMAL(5, 4),         -- всего 5 цифр, 4 после запятой: 9.9999
    salary NUMERIC(8, 2)            -- синоним DECIMAL
);

-- ПРИМЕР ИСПОЛЬЗОВАНИЯ:
INSERT INTO example_decimal VALUES 
(12345678.99, 0.2050, 50000.00),
(99999999.99, 0.9999, 99999.99);
```

### **СТРОКОВЫЕ ТИПЫ ДАННЫХ**

#### **СТРОКИ ФИКСИРОВАННОЙ ДЛИНЫ**
```sql
-- CHAR: всегда занимает указанное количество символов
CREATE TABLE example_char (
    country_code CHAR(2),           -- ровно 2 символа: 'US', 'RU'
    gender CHAR(1),                 -- ровно 1 символ: 'M', 'F'
    fixed_id CHAR(10)               -- ровно 10 символов, дополняется пробелами
);

-- ПРИМЕРЫ:
INSERT INTO example_char VALUES 
('US', 'M', 'ID12345678'),         -- сохранится как есть
('RU', 'F', 'ID999');              -- сохранится как 'ID999     ' (с пробелами)
```

#### **СТРОКИ ПЕРЕМЕННОЙ ДЛИНЫ**
```sql
-- VARCHAR: занимает только необходимое место
CREATE TABLE example_varchar (
    username VARCHAR(50),           -- до 50 символов
    email VARCHAR(100),             -- до 100 символов
    description VARCHAR(255)        -- до 255 символов
);

-- ПРИМЕРЫ:
INSERT INTO example_varchar VALUES 
('john_doe', 'john@example.com', 'Это описание пользователя'),
('alice', 'alice@test.org', 'Короткое описание'); -- займет только необходимое место
```

#### **БОЛЬШИЕ ТЕКСТОВЫЕ ПОЛЯ**
```sql
-- TEXT: до 65,535 символов
CREATE TABLE example_text (
    article_content TEXT,           -- до 64KB
    json_data TEXT                  -- для хранения JSON
);

-- MEDIUMTEXT: до 16,777,215 символов
CREATE TABLE example_mediumtext (
    book_content MEDIUMTEXT,        -- до 16MB
    large_json MEDIUMTEXT
);

-- LONGTEXT: до 4,294,967,295 символов
CREATE TABLE example_longtext (
    encyclopedia LONGTEXT,          -- до 4GB
    huge_xml_data LONGTEXT
);
```

#### **БИНАРНЫЕ ДАННЫЕ**
```sql
-- BLOB: бинарные данные до 64KB
CREATE TABLE example_blob (
    small_image BLOB,
    pdf_file BLOB
);

-- LONGBLOB: большие бинарные данные до 4GB
CREATE TABLE example_longblob (
    video_file LONGBLOB,
    backup_file LONGBLOB
);
```

### **ТИПЫ ДАННЫХ ДАТЫ И ВРЕМЕНИ**

```sql
-- DATE: только дата
CREATE TABLE example_date (
    birth_date DATE,                -- '1990-05-15'
    event_date DATE
);

-- TIME: только время
CREATE TABLE example_time (
    start_time TIME,                -- '14:30:00'
    duration TIME                   -- '02:45:30'
);

-- DATETIME: дата и время
CREATE TABLE example_datetime (
    created_at DATETIME,            -- '2023-12-25 14:30:00'
    updated_at DATETIME
);

-- TIMESTAMP: автоматическая метка времени
CREATE TABLE example_timestamp (
    id INT,
    created_ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- YEAR: год
CREATE TABLE example_year (
    graduation_year YEAR,           -- 2023
    foundation_year YEAR(4)         -- 1990 (4 цифры)
);
```

### **ПРОЧИЕ ТИПЫ ДАННЫХ**

```sql
-- BOOLEAN: логический тип
CREATE TABLE example_boolean (
    is_active BOOLEAN,              -- TRUE/FALSE
    is_verified BOOLEAN DEFAULT FALSE
);

-- ENUM: перечисление
CREATE TABLE example_enum (
    status ENUM('active', 'inactive', 'pending'),
    priority ENUM('low', 'medium', 'high', 'critical')
);

-- SET: множество значений
CREATE TABLE example_set (
    permissions SET('read', 'write', 'execute', 'delete'),
    tags SET('urgent', 'important', 'follow_up')
);

-- JSON: хранение JSON данных
CREATE TABLE example_json (
    user_preferences JSON,
    config_data JSON
);
```

---

## ОГРАНИЧЕНИЯ (CONSTRAINTS)

### **PRIMARY KEY - ПЕРВИЧНЫЙ КЛЮЧ**

```sql
-- ПРОСТОЙ ПЕРВИЧНЫЙ КЛЮЧ
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- СОСТАВНОЙ ПЕРВИЧНЫЙ КЛЮЧ
CREATE TABLE course_registrations (
    student_id INT,
    course_id INT,
    registration_date DATE,
    PRIMARY KEY (student_id, course_id)  -- комбинация должна быть уникальной
);

-- С АВТОИНКРЕМЕНТОМ
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(200) NOT NULL
);

-- ПРИ ВНЕСЕНИИ ИЗМЕНЕНИЙ
ALTER TABLE employees ADD PRIMARY KEY (emp_id);
ALTER TABLE orders DROP PRIMARY KEY;
```

### **FOREIGN KEY - ВНЕШНИЙ КЛЮЧ**

```sql
-- БАЗОВЫЙ ВНЕШНИЙ КЛЮЧ
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(dept_id)
);

-- С ОПЦИЯМИ УДАЛЕНИЯ И ОБНОВЛЕНИЯ
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE      -- удалить заказы при удалении клиента
        ON UPDATE CASCADE      -- обновить customer_id при изменении в customers
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
        ON DELETE RESTRICT    -- запретить удаление если есть связанные записи
        ON UPDATE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
        ON DELETE SET NULL    -- установить NULL при удалении продукта
        ON UPDATE CASCADE
);

-- ДОБАВЛЕНИЕ ВНЕШНЕГО КЛЮЧА К СУЩЕСТВУЮЩЕЙ ТАБЛИЦЕ
ALTER TABLE employees
ADD CONSTRAINT fk_emp_dept
FOREIGN KEY (department_id) REFERENCES departments(dept_id)
ON DELETE SET NULL
ON UPDATE CASCADE;
```

### **NOT NULL - ЗАПРЕТ NULL ЗНАЧЕНИЙ**

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,           -- обязательно к заполнению
    email VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20),                -- может быть NULL
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- ИЗМЕНЕНИЕ СУЩЕСТВУЮЩЕГО СТОЛБЦА
ALTER TABLE users MODIFY COLUMN phone_number VARCHAR(20) NOT NULL;
ALTER TABLE products MODIFY COLUMN price DECIMAL(10,2) NOT NULL DEFAULT 0;
```

### **UNIQUE - УНИКАЛЬНЫЕ ЗНАЧЕНИЯ**

```sql
CREATE TABLE companies (
    company_id INT PRIMARY KEY,
    company_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,               -- уникальный email
    tax_id VARCHAR(20) UNIQUE,               -- уникальный налоговый номер
    website VARCHAR(100)                     -- не уникальный
);

-- СОСТАВНОЙ УНИКАЛЬНЫЙ КЛЮЧ
CREATE TABLE user_contacts (
    user_id INT,
    contact_type VARCHAR(20),
    contact_value VARCHAR(100),
    UNIQUE KEY unique_user_contact (user_id, contact_type)  -- комбинация должна быть уникальной
);

-- ДОБАВЛЕНИЕ УНИКАЛЬНОГО ОГРАНИЧЕНИЯ
ALTER TABLE employees ADD CONSTRAINT uk_employee_email UNIQUE (email);
ALTER TABLE products ADD UNIQUE (product_code);
```

### **CHECK - ПРОВЕРКА УСЛОВИЙ**

```sql
-- ПРОВЕРКА ЗНАЧЕНИЙ ПРИ СОЗДАНИИ ТАБЛИЦЫ
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    salary DECIMAL(10,2) CHECK (salary >= 0),              -- зарплата не отрицательная
    age INT CHECK (age >= 18 AND age <= 65),               -- возраст от 18 до 65
    department VARCHAR(50) CHECK (department IN ('IT', 'HR', 'Finance', 'Marketing')),
    hire_date DATE CHECK (hire_date >= '2000-01-01')      -- дата найма после 2000 года
);

-- СЛОЖНЫЕ ПРОВЕРКИ
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    discount DECIMAL(10,2),
    stock_quantity INT,
    CONSTRAINT chk_valid_price CHECK (price > 0),
    CONSTRAINT chk_valid_discount CHECK (discount >= 0 AND discount <= price),
    CONSTRAINT chk_non_negative_stock CHECK (stock_quantity >= 0)
);

-- ДОБАВЛЕНИЕ ПРОВЕРКИ К СУЩЕСТВУЮЩЕЙ ТАБЛИЦЕ
ALTER TABLE employees
ADD CONSTRAINT chk_valid_salary
CHECK (salary >= 0 AND salary <= 1000000);

ALTER TABLE orders
ADD CONSTRAINT chk_order_date
CHECK (order_date <= CURDATE());
```

### **DEFAULT - ЗНАЧЕНИЯ ПО УМОЛЧАНИЮ**

```sql
CREATE TABLE website_users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    registration_date DATETIME DEFAULT CURRENT_TIMESTAMP,  -- текущая дата/время
    last_login DATETIME DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,                       -- TRUE по умолчанию
    login_count INT DEFAULT 0,                            -- 0 по умолчанию
    status VARCHAR(20) DEFAULT 'active',                  -- 'active' по умолчанию
    preferences JSON DEFAULT JSON_OBJECT('theme', 'light', 'language', 'en')
);

-- ИЗМЕНЕНИЕ ЗНАЧЕНИЯ ПО УМОЛЧАНИЮ
ALTER TABLE employees 
ALTER COLUMN salary SET DEFAULT 50000;

ALTER TABLE products 
ALTER COLUMN created_at SET DEFAULT CURRENT_TIMESTAMP;
```

---

## DDL - DATA DEFINITION LANGUAGE

### **СОЗДАНИЕ БАЗЫ ДАННЫХ**

```sql
-- ПРОСТОЕ СОЗДАНИЕ БАЗЫ ДАННЫХ
CREATE DATABASE company;

-- СОЗДАНИЕ С УКАЗАНИЕМ КОДИРОВКИ
CREATE DATABASE company
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- СОЗДАНИЕ С ПРОВЕРКОЙ СУЩЕСТВОВАНИЯ
CREATE DATABASE IF NOT EXISTS company;

-- ПОСМОТРЕТЬ ВСЕ БАЗЫ ДАННЫХ
SHOW DATABASES;

-- ВЫБРАТЬ БАЗУ ДАННЫХ ДЛЯ РАБОТЫ
USE company;

-- УДАЛЕНИЕ БАЗЫ ДАННЫХ
DROP DATABASE company;
DROP DATABASE IF EXISTS old_company;
```

### **СОЗДАНИЕ ТАБЛИЦ**

```sql
-- БАЗОВАЯ ТАБЛИЦА СОТРУДНИКОВ
CREATE TABLE employees (
    employee_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2) CHECK (salary >= 0),
    department_id INT,
    manager_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- ТАБЛИЦА ОТДЕЛОВ
CREATE TABLE departments (
    department_id INT AUTO_INCREMENT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE,
    manager_id INT,
    budget DECIMAL(12,2) DEFAULT 0,
    location VARCHAR(100)
);

-- ТАБЛИЦА ПРОЕКТОВ
CREATE TABLE projects (
    project_id INT AUTO_INCREMENT PRIMARY KEY,
    project_name VARCHAR(200) NOT NULL,
    description TEXT,
    start_date DATE,
    end_date DATE,
    budget DECIMAL(12,2),
    status ENUM('planned', 'active', 'completed', 'cancelled') DEFAULT 'planned',
    CHECK (end_date IS NULL OR end_date >= start_date)
);

-- ТАБЛИЦА СВЯЗИ МНОГИЕ-КО-МНОГИМ
CREATE TABLE employee_projects (
    employee_id INT,
    project_id INT,
    role VARCHAR(50),
    assignment_date DATE DEFAULT (CURDATE()),
    hours_worked DECIMAL(5,2) DEFAULT 0,
    PRIMARY KEY (employee_id, project_id),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id) ON DELETE CASCADE,
    FOREIGN KEY (project_id) REFERENCES projects(project_id) ON DELETE CASCADE
);
```

### **ИЗМЕНЕНИЕ СТРУКТУРЫ ТАБЛИЦ**

```sql
-- ДОБАВЛЕНИЕ НОВОГО СТОЛБЦА
ALTER TABLE employees 
ADD COLUMN middle_name VARCHAR(50);

ALTER TABLE employees 
ADD COLUMN emergency_contact VARCHAR(100) AFTER last_name;

-- УДАЛЕНИЕ СТОЛБЦА
ALTER TABLE employees 
DROP COLUMN middle_name;

-- ИЗМЕНЕНИЕ ТИПА ДАННЫХ СТОЛБЦА
ALTER TABLE employees 
MODIFY COLUMN phone VARCHAR(30);

ALTER TABLE employees 
MODIFY COLUMN salary DECIMAL(12,2);

-- ИЗМЕНЕНИЕ ИМЕНИ СТОЛБЦА
ALTER TABLE employees 
CHANGE COLUMN phone phone_number VARCHAR(20);

-- ДОБАВЛЕНИЕ ОГРАНИЧЕНИЙ
ALTER TABLE employees 
ADD CONSTRAINT fk_employee_department 
FOREIGN KEY (department_id) REFERENCES departments(department_id);

ALTER TABLE employees 
ADD CONSTRAINT fk_employee_manager 
FOREIGN KEY (manager_id) REFERENCES employees(employee_id);

-- УДАЛЕНИЕ ОГРАНИЧЕНИЙ
ALTER TABLE employees 
DROP FOREIGN KEY fk_employee_department;

-- ДОБАВЛЕНИЕ ИНДЕКСА
ALTER TABLE employees 
ADD INDEX idx_last_name (last_name);

ALTER TABLE employees 
ADD INDEX idx_department_salary (department_id, salary DESC);
```

### **УДАЛЕНИЕ ТАБЛИЦ**

```sql
-- БАЗОВОЕ УДАЛЕНИЕ
DROP TABLE employees;

-- БЕЗОПАСНОЕ УДАЛЕНИЕ (ЕСЛИ СУЩЕСТВУЕТ)
DROP TABLE IF EXISTS old_employees;

-- УДАЛЕНИЕ С ОПЦИЯМИ
DROP TABLE temporary_data CASCADE;  -- удалить зависимости

-- ОЧИСТКА ДАННЫХ БЕЗ УДАЛЕНИЯ ТАБЛИЦЫ
TRUNCATE TABLE log_entries;         -- быстро, нельзя откатить
TRUNCATE TABLE audit_trail RESTART IDENTITY;  -- сброс автоинкремента
```

---

## DML - DATA MANIPULATION LANGUAGE

### **INSERT - ДОБАВЛЕНИЕ ДАННЫХ**

```sql
-- ВСТАВКА ОДНОЙ ЗАПИСИ (ВСЕ СТОЛБЦЫ)
INSERT INTO employees VALUES (
    NULL, 'Иван', 'Петров', 'ivan.petrov@company.com', 
    '+7-999-123-45-67', '2023-01-15', 75000.00, 1, NULL, 
    CURRENT_TIMESTAMP, CURRENT_TIMESTAMP
);

-- ВСТАВКА ОДНОЙ ЗАПИСИ (КОНКРЕТНЫЕ СТОЛБЦЫ)
INSERT INTO employees 
    (first_name, last_name, email, phone, hire_date, salary, department_id)
VALUES 
    ('Мария', 'Сидорова', 'maria.sidorova@company.com', 
     '+7-999-123-45-68', '2023-02-20', 80000.00, 2);

-- ВСТАВКА НЕСКОЛЬКИХ ЗАПИСЕЙ ОДНОВРЕМЕННО
INSERT INTO employees 
    (first_name, last_name, email, hire_date, salary, department_id) 
VALUES 
    ('Алексей', 'Козлов', 'alexey.kozlov@company.com', '2023-03-10', 65000.00, 1),
    ('Ольга', 'Новикова', 'olga.novikova@company.com', '2023-03-15', 70000.00, 2),
    ('Дмитрий', 'Васильев', 'dmitry.vasilev@company.com', '2023-04-01', 90000.00, 3);

-- ВСТАВКА ДАННЫХ ИЗ ДРУГОЙ ТАБЛИЦЫ
INSERT INTO employees_archive 
SELECT * FROM employees 
WHERE hire_date < '2020-01-01';

-- ВСТАВКА С ВОЗВРАТОМ ДАННЫХ (PostgreSQL)
INSERT INTO employees 
    (first_name, last_name, email, salary)
VALUES 
    ('Петр', 'Семенов', 'petr.semenov@company.com', 85000.00)
RETURNING employee_id, first_name, last_name;

-- ВСТАВКА С ОБРАБОТКОЙ ДУБЛИКАТОВ (MySQL)
INSERT INTO employees 
    (first_name, last_name, email, salary)
VALUES 
    ('Иван', 'Петров', 'ivan.petrov@company.com', 75000.00)
ON DUPLICATE KEY UPDATE 
    salary = VALUES(salary),
    updated_at = CURRENT_TIMESTAMP;
```

### **UPDATE - ОБНОВЛЕНИЕ ДАННЫХ**

```sql
-- ОБНОВЛЕНИЕ ВСЕХ ЗАПИСЕЙ
UPDATE employees SET salary = salary * 1.05;  -- повышение всем на 5%

-- ОБНОВЛЕНИЕ С УСЛОВИЕМ
UPDATE employees 
SET salary = salary * 1.10 
WHERE department_id = 1;  -- IT отделу +10%

-- ОБНОВЛЕНИЕ НЕСКОЛЬКИХ СТОЛБЦОВ
UPDATE employees 
SET 
    salary = 82000.00,
    department_id = 3,
    phone = '+7-999-999-99-99'
WHERE employee_id = 5;

-- ОБНОВЛЕНИЕ С ИСПОЛЬЗОВАНИЕМ ПОДЗАПРОСА
UPDATE employees 
SET salary = (
    SELECT AVG(salary) FROM employees WHERE department_id = 2
) 
WHERE department_id = 1;

-- ОБНОВЛЕНИЕ С JOIN
UPDATE employees e
JOIN departments d ON e.department_id = d.department_id
SET e.salary = e.salary * 1.15
WHERE d.department_name = 'IT';

-- ОБНОВЛЕНИЕ С ВОЗВРАТОМ ДАННЫХ (PostgreSQL)
UPDATE employees 
SET salary = salary + 5000
WHERE employee_id = 10
RETURNING employee_id, first_name, last_name, salary;

-- ОБНОВЛЕНИЕ С ИСПОЛЬЗОВАНИЕМ CASE
UPDATE employees 
SET salary = CASE 
    WHEN department_id = 1 THEN salary * 1.10
    WHEN department_id = 2 THEN salary * 1.08
    ELSE salary * 1.05
END;
```

### **DELETE - УДАЛЕНИЕ ДАННЫХ**

```sql
-- УДАЛЕНИЕ ВСЕХ ЗАПИСЕЙ (ОСТОРОЖНО!)
DELETE FROM employees;

-- УДАЛЕНИЕ С УСЛОВИЕМ
DELETE FROM employees 
WHERE salary < 30000;

-- УДАЛЕНИЕ С ПОДЗАПРОСОМ
DELETE FROM employees 
WHERE department_id IN (
    SELECT department_id FROM departments WHERE location = 'Moscow'
);

-- УДАЛЕНИЕ С ИСПОЛЬЗОВАНИЕМ JOIN
DELETE e FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.budget < 100000;

-- УДАЛЕНИЕ С ВОЗВРАТОМ ДАННЫХ (PostgreSQL)
DELETE FROM employees 
WHERE employee_id = 15
RETURNING *;

-- УДАЛЕНИЕ С ОГРАНИЧЕНИЕМ КОЛИЧЕСТВА
DELETE FROM log_entries 
ORDER BY created_at 
LIMIT 1000;  -- удалить 1000 самых старых записей
```

### **MERGE/UPSERT - ОБЪЕДИНЕНИЕ ДАННЫХ**

```sql
-- MySQL (INSERT ... ON DUPLICATE KEY UPDATE)
INSERT INTO employees 
    (employee_id, first_name, last_name, email, salary)
VALUES 
    (1, 'Иван', 'Петров', 'new_email@company.com', 80000.00)
ON DUPLICATE KEY UPDATE 
    first_name = VALUES(first_name),
    last_name = VALUES(last_name),
    email = VALUES(email),
    salary = VALUES(salary),
    updated_at = CURRENT_TIMESTAMP;

-- PostgreSQL (INSERT ... ON CONFLICT)
INSERT INTO employees 
    (employee_id, first_name, last_name, email, salary)
VALUES 
    (1, 'Иван', 'Петров', 'new_email@company.com', 80000.00)
ON CONFLICT (employee_id) 
DO UPDATE SET 
    first_name = EXCLUDED.first_name,
    last_name = EXCLUDED.last_name,
    email = EXCLUDED.email,
    salary = EXCLUDED.salary,
    updated_at = CURRENT_TIMESTAMP;

-- SQL Server (MERGE)
MERGE INTO employees AS target
USING (VALUES (1, 'Иван', 'Петров', 'new_email@company.com', 80000.00)) 
       AS source (employee_id, first_name, last_name, email, salary)
ON target.employee_id = source.employee_id
WHEN MATCHED THEN
    UPDATE SET 
        first_name = source.first_name,
        last_name = source.last_name,
        email = source.email,
        salary = source.salary,
        updated_at = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, email, salary)
    VALUES (source.employee_id, source.first_name, source.last_name, source.email, source.salary);
```

---

## DQL - DATA QUERY LANGUAGE

### **SELECT - БАЗОВЫЕ ЗАПРОСЫ**

```sql
-- ВЫБРАТЬ ВСЕ СТОЛБЦЫ ИЗ ТАБЛИЦЫ
SELECT * FROM employees;

-- ВЫБРАТЬ КОНКРЕТНЫЕ СТОЛБЦЫ
SELECT 
    employee_id,
    first_name,
    last_name,
    salary
FROM employees;

-- ВЫБРАТЬ С ВЫЧИСЛЯЕМЫМИ СТОЛБЦАМИ
SELECT 
    first_name,
    last_name,
    salary,
    salary * 12 AS annual_salary,              -- годовая зарплата
    salary * 0.87 AS net_salary,               -- зарплата после налогов (13%)
    CONCAT(first_name, ' ', last_name) AS full_name  -- полное имя
FROM employees;

-- ВЫБРАТЬ УНИКАЛЬНЫЕ ЗНАЧЕНИЯ
SELECT DISTINCT department_id FROM employees;

SELECT DISTINCT 
    department_id, 
    EXTRACT(YEAR FROM hire_date) AS hire_year 
FROM employees;

-- ОГРАНИЧЕНИЕ КОЛИЧЕСТВА ЗАПИСЕЙ
SELECT * FROM employees LIMIT 10;                     -- MySQL, PostgreSQL
SELECT TOP 10 * FROM employees;                       -- SQL Server
SELECT * FROM employees WHERE ROWNUM <= 10;           -- Oracle

-- ПРОПУСК ЗАПИСЕЙ (ПАГИНАЦИЯ)
SELECT * FROM employees LIMIT 10 OFFSET 20;           -- записи 21-30
SELECT * FROM employees ORDER BY hire_date LIMIT 10 OFFSET 30; -- записи 31-40

-- SQL Server (альтернатива)
SELECT * FROM employees 
ORDER BY employee_id 
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```

### **WHERE - ФИЛЬТРАЦИЯ ДАННЫХ**

```sql
-- ОПЕРАТОРЫ СРАВНЕНИЯ
SELECT * FROM employees WHERE salary = 50000;
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE salary < 50000;
SELECT * FROM employees WHERE salary >= 50000;
SELECT * FROM employees WHERE salary <= 50000;
SELECT * FROM employees WHERE salary <> 50000;  -- не равно
SELECT * FROM employees WHERE salary != 50000;  -- не равно (альтернатива)

-- ЛОГИЧЕСКИЕ ОПЕРАТОРЫ
SELECT * FROM employees 
WHERE salary > 40000 AND department_id = 1;

SELECT * FROM employees 
WHERE salary > 60000 OR department_id = 2;

SELECT * FROM employees 
WHERE NOT department_id = 3;

-- КОМБИНАЦИИ ЛОГИЧЕСКИХ ОПЕРАТОРОВ
SELECT * FROM employees 
WHERE (department_id = 1 OR department_id = 2) 
  AND salary BETWEEN 40000 AND 80000;

-- BETWEEN (ВКЛЮЧАЕТ ГРАНИЧНЫЕ ЗНАЧЕНИЯ)
SELECT * FROM employees 
WHERE salary BETWEEN 40000 AND 60000;

SELECT * FROM employees 
WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31';

-- IN (ВХОЖДЕНИЕ В СПИСОК)
SELECT * FROM employees 
WHERE department_id IN (1, 2, 3);

SELECT * FROM employees 
WHERE last_name IN ('Петров', 'Сидорова', 'Козлов');

-- NOT IN (НЕ ВХОДИТ В СПИСОК)
SELECT * FROM employees 
WHERE department_id NOT IN (1, 2);

-- LIKE (ПОИСК ПО ШАБЛОНУ)
SELECT * FROM employees WHERE first_name LIKE 'И%';      -- начинается на И
SELECT * FROM employees WHERE last_name LIKE '%ов';      -- заканчивается на ов
SELECT * FROM employees WHERE email LIKE '%@gmail.com';  -- email gmail
SELECT * FROM employees WHERE first_name LIKE '_а%';     -- второй символ 'а'
SELECT * FROM employees WHERE name LIKE 'Ал_кс%';        -- третий символ любой

-- REGEXP (РЕГУЛЯРНЫЕ ВЫРАЖЕНИЯ)
SELECT * FROM employees WHERE first_name REGEXP '^Иван|Петр';
SELECT * FROM employees WHERE email REGEXP '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$';

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE email IS NOT NULL;
```

### **GROUP BY - ГРУППИРОВКА ДАННЫХ**

```sql
-- ПРОСТАЯ ГРУППИРОВКА
SELECT 
    department_id,
    COUNT(*) as employee_count
FROM employees
GROUP BY department_id;

-- ГРУППИРОВКА ПО НЕСКОЛЬКИМ СТОЛБЦАМ
SELECT 
    department_id,
    EXTRACT(YEAR FROM hire_date) as hire_year,
    COUNT(*) as hired_count
FROM employees
GROUP BY department_id, EXTRACT(YEAR FROM hire_date);

-- ГРУППИРОВКА С АГРЕГАТНЫМИ ФУНКЦИЯМИ
SELECT 
    department_id,
    COUNT(*) as total_employees,
    AVG(salary) as average_salary,
    MAX(salary) as max_salary,
    MIN(salary) as min_salary,
    SUM(salary) as total_salary
FROM employees
GROUP BY department_id;

-- ГРУППИРОВКА С ФИЛЬТРАЦИЕЙ HAVING
SELECT 
    department_id,
    AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 50000;

-- ГРУППИРОВКА С WHERE И HAVING
SELECT 
    department_id,
    COUNT(*) as high_earners
FROM employees
WHERE salary > 40000
GROUP BY department_id
HAVING COUNT(*) >= 3;
```

### **АГРЕГАТНЫЕ ФУНКЦИИ**

```sql
-- ОСНОВНЫЕ АГРЕГАТНЫЕ ФУНКЦИИ
SELECT 
    COUNT(*) as total_employees,                    -- общее количество
    COUNT(DISTINCT department_id) as unique_departments, -- уникальные отделы
    AVG(salary) as average_salary,                  -- средняя зарплата
    MAX(salary) as max_salary,                      -- максимальная зарплата
    MIN(salary) as min_salary,                      -- минимальная зарплата
    SUM(salary) as total_salary_budget              -- сумма всех зарплат
FROM employees;

-- АГРЕГАТНЫЕ ФУНКЦИИ С ГРУППИРОВКОЙ
SELECT 
    department_id,
    COUNT(*) as emp_count,
    ROUND(AVG(salary), 2) as avg_salary,           -- округление до 2 знаков
    FORMAT(SUM(salary), 2) as total_salary,        -- форматирование числа
    MAX(hire_date) as latest_hire,
    MIN(hire_date) as earliest_hire
FROM employees
GROUP BY department_id;

-- УСЛОВНЫЕ АГРЕГАЦИИ
SELECT 
    department_id,
    COUNT(*) as total_employees,
    -- количество сотрудников с зарплатой > 50000
    SUM(CASE WHEN salary > 50000 THEN 1 ELSE 0 END) as high_earners,
    -- средняя зарплата сотрудников, нанятых в 2023
    AVG(CASE WHEN EXTRACT(YEAR FROM hire_date) = 2023 THEN salary END) as avg_2023_salary,
    -- процент высокооплачиваемых сотрудников
    ROUND(
        SUM(CASE WHEN salary > 50000 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 
        2
    ) as high_earners_percent
FROM employees
GROUP BY department_id;
```

### **ORDER BY - СОРТИРОВКА РЕЗУЛЬТАТОВ**

```sql
-- ПРОСТАЯ СОРТИРОВКА
SELECT * FROM employees ORDER BY last_name;           -- по возрастанию (ASC)
SELECT * FROM employees ORDER BY salary DESC;         -- по убыванию

-- СОРТИРОВКА ПО НЕСКОЛЬКИМ СТОЛБЦАМ
SELECT * FROM employees 
ORDER BY department_id ASC, salary DESC;

-- СОРТИРОВКА ПО ВЫЧИСЛЯЕМОМУ СТОЛБЦУ
SELECT 
    employee_id,
    first_name,
    last_name,
    salary,
    salary * 12 AS annual_salary 
FROM employees 
ORDER BY annual_salary DESC;

-- СОРТИРОВКА С УСЛОВИЯМИ
SELECT 
    employee_id,
    first_name,
    last_name,
    department_id,
    salary
FROM employees
ORDER BY 
    CASE 
        WHEN department_id = 1 THEN 1  -- IT первый
        WHEN department_id = 2 THEN 2  -- HR второй
        ELSE 3                         -- остальные
    END,
    salary DESC;

-- СОРТИРОВКА ПО ПОЗИЦИИ СТОЛБЦА (НЕ РЕКОМЕНДУЕТСЯ)
SELECT 
    employee_id,
    first_name,
    last_name,
    salary 
FROM employees 
ORDER BY 4 DESC;  -- сортировка по salary (4-й столбец)
```

---

## JOIN - СОЕДИНЕНИЯ ТАБЛИЦ

### **INNER JOIN**

```sql
-- БАЗОВЫЙ INNER JOIN
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- INNER JOIN С НЕСКОЛЬКИМИ УСЛОВИЯМИ
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    p.project_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN employee_projects ep ON e.employee_id = ep.employee_id
INNER JOIN projects p ON ep.project_id = p.project_id
WHERE p.status = 'active';

-- INNER JOIN С АГРЕГАТНЫМИ ФУНКЦИЯМИ
SELECT 
    d.department_name,
    COUNT(e.employee_id) as employee_count,
    AVG(e.salary) as avg_salary
FROM departments d
INNER JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;
```

### **LEFT JOIN**

```sql
-- БАЗОВЫЙ LEFT JOIN
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- LEFT JOIN С ФИЛЬТРАЦИЕЙ ПО ПРАВОЙ ТАБЛИЦЕ
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;  -- сотрудники без отдела

-- LEFT JOIN С АГРЕГАЦИЕЙ
SELECT 
    d.department_name,
    COUNT(e.employee_id) as employee_count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;
```

### ▶**RIGHT JOIN**

```sql
-- БАЗОВЫЙ RIGHT JOIN
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- RIGHT JOIN С ФИЛЬТРАЦИЕЙ ПО ЛЕВОЙ ТАБЛИЦЕ
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE e.employee_id IS NULL;  -- отделы без сотрудников
```

### **FULL OUTER JOIN**

```sql
-- FULL OUTER JOIN (MySQL через UNION)
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id

UNION

SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- FULL OUTER JOIN С ИДЕНТИФИКАЦИЕЙ ТИПА ЗАПИСИ
SELECT 
    COALESCE(e.first_name, 'НЕТ СОТРУДНИКА') as first_name,
    COALESCE(e.last_name, 'НЕТ СОТРУДНИКА') as last_name,
    COALESCE(d.department_name, 'НЕТ ОТДЕЛА') as department_name,
    CASE 
        WHEN e.employee_id IS NULL THEN 'ОТДЕЛ БЕЗ СОТРУДНИКОВ'
        WHEN d.department_id IS NULL THEN 'СОТРУДНИК БЕЗ ОТДЕЛА'
        ELSE 'СОТРУДНИК В ОТДЕЛЕ'
    END as record_type
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id

UNION

SELECT 
    COALESCE(e.first_name, 'НЕТ СОТРУДНИКА') as first_name,
    COALESCE(e.last_name, 'НЕТ СОТРУДНИКА') as last_name,
    COALESCE(d.department_name, 'НЕТ ОТДЕЛА') as department_name,
    CASE 
        WHEN e.employee_id IS NULL THEN 'ОТДЕЛ БЕЗ СОТРУДНИКОВ'
        WHEN d.department_id IS NULL THEN 'СОТРУДНИК БЕЗ ОТДЕЛА'
        ELSE 'СОТРУДНИК В ОТДЕЛЕ'
    END as record_type
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;
```

### **CROSS JOIN**

```sql
-- CROSS JOIN (ДЕКАРТОВО ПРОИЗВЕДЕНИЕ)
SELECT 
    e.first_name,
    p.project_name
FROM employees e
CROSS JOIN projects p;

-- CROSS JOIN ДЛЯ СОЗДАНИЯ КОМБИНАЦИЙ
SELECT 
    d.department_name,
    p.project_name
FROM departments d
CROSS JOIN projects p
WHERE d.location = 'Moscow' AND p.status = 'planned';
```

### **SELF JOIN**

```sql
-- SELF JOIN ДЛЯ ИЕРАРХИИ СОТРУДНИКОВ
SELECT 
    emp.first_name as employee,
    emp.last_name as employee_last_name,
    mgr.first_name as manager,
    mgr.last_name as manager_last_name
FROM employees emp
LEFT JOIN employees mgr ON emp.manager_id = mgr.employee_id;

-- SELF JOIN ДЛЯ НАХОЖДЕНИЯ КОЛЛЕГ
SELECT 
    e1.first_name as employee1,
    e1.last_name as employee1_last_name,
    e2.first_name as employee2,
    e2.last_name as employee2_last_name,
    e1.department_id
FROM employees e1
INNER JOIN employees e2 ON e1.department_id = e2.department_id
WHERE e1.employee_id < e2.employee_id;  -- избегаем дубликатов
```

---

## ПОДЗАПРОСЫ

### **ПОДЗАПРОСЫ В WHERE**

```sql
-- ПРОСТОЙ ПОДЗАПРОС
SELECT * FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);

-- ПОДЗАПРОС С IN
SELECT * FROM employees 
WHERE department_id IN (
    SELECT department_id FROM departments WHERE location = 'Moscow'
);

-- ПОДЗАПРОС С EXISTS
SELECT * FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e 
    WHERE e.department_id = d.department_id AND e.salary > 80000
);

-- ПОДЗАПРОС С NOT EXISTS
SELECT * FROM departments d
WHERE NOT EXISTS (
    SELECT 1 FROM employees e 
    WHERE e.department_id = d.department_id
);

-- ПОДЗАПРОС С ALL/ANY
SELECT * FROM employees 
WHERE salary > ALL (
    SELECT salary FROM employees WHERE department_id = 1
);

SELECT * FROM employees 
WHERE salary > ANY (
    SELECT salary FROM employees WHERE department_id = 2
);
```

### **ПОДЗАПРОСЫ В SELECT**

```sql
-- СКАЛЯРНЫЕ ПОДЗАПРОСЫ
SELECT 
    first_name,
    last_name,
    salary,
    (SELECT AVG(salary) FROM employees) as company_avg,
    salary - (SELECT AVG(salary) FROM employees) as diff_from_avg
FROM employees;

-- КОРРЕЛИРОВАННЫЕ ПОДЗАПРОСЫ
SELECT 
    e.first_name,
    e.last_name,
    e.salary,
    (SELECT AVG(salary) FROM employees e2 
     WHERE e2.department_id = e.department_id) as dept_avg,
    (SELECT COUNT(*) FROM employee_projects ep 
     WHERE ep.employee_id = e.employee_id) as project_count
FROM employees e;
```

### **ПОДЗАПРОСЫ В FROM**

```sql
-- ПОДЗАПРОС КАК ВИРТУАЛЬНАЯ ТАБЛИЦА
SELECT 
    dept_stats.department_name,
    dept_stats.avg_salary
FROM (
    SELECT 
        d.department_name,
        AVG(e.salary) as avg_salary
    FROM departments d
    JOIN employees e ON d.department_id = e.department_id
    GROUP BY d.department_name
) dept_stats
WHERE dept_stats.avg_salary > 50000;

-- СЛОЖНЫЙ ПОДЗАПРОС С АГРЕГАЦИЕЙ
SELECT 
    year_stats.hire_year,
    year_stats.total_hired,
    year_stats.avg_salary
FROM (
    SELECT 
        EXTRACT(YEAR FROM hire_date) as hire_year,
        COUNT(*) as total_hired,
        AVG(salary) as avg_salary
    FROM employees
    GROUP BY EXTRACT(YEAR FROM hire_date)
) year_stats
ORDER BY year_stats.hire_year DESC;
```

---

## ТРАНЗАКЦИИ

### **БАЗОВЫЕ ТРАНЗАКЦИИ**

```sql
-- ПРОСТАЯ ТРАНЗАКЦИЯ
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- ЕСЛИ ВСЕ ОПЕРАЦИИ УСПЕШНЫ
COMMIT;

-- ЕСЛИ ЧТО-ТО ПОШЛО НЕ ТАК
ROLLBACK;
```

### **ТРАНЗАКЦИИ С ТОЧКАМИ СОХРАНЕНИЯ**

```sql
START TRANSACTION;

-- ОПЕРАЦИЯ 1
UPDATE inventory SET quantity = quantity - 5 WHERE product_id = 101;

-- СОХРАНИТЬ ТОЧКУ
SAVEPOINT after_inventory_update;

-- ОПЕРАЦИЯ 2
UPDATE accounts SET balance = balance - 500 WHERE account_id = 1;

-- ЕСЛИ ОПЕРАЦИЯ 2 НЕ УДАЛАСЬ
ROLLBACK TO SAVEPOINT after_inventory_update;

-- ПРОДОЛЖИТЬ С ОПЕРАЦИИ 3
UPDATE orders SET status = 'completed' WHERE order_id = 1001;

COMMIT;
```

### 🏦 **ПРАКТИЧЕСКИЙ ПРИМЕР: БАНКОВСКИЙ ПЕРЕВОД**

```sql
START TRANSACTION;

-- ПРОВЕРКА ДОСТАТОЧНОСТИ СРЕДСТВ
SELECT balance INTO @current_balance 
FROM accounts WHERE account_id = 1 FOR UPDATE;

IF @current_balance >= 500 THEN
    -- СПИСАНИЕ СО СЧЕТА ОТПРАВИТЕЛЯ
    UPDATE accounts SET balance = balance - 500 WHERE account_id = 1;
    
    -- ЗАЧИСЛЕНИЕ НА СЧЕТ ПОЛУЧАТЕЛЯ
    UPDATE accounts SET balance = balance + 500 WHERE account_id = 2;
    
    -- ЗАПИСЬ В ИСТОРИЮ ОПЕРАЦИЙ
    INSERT INTO transactions (from_account, to_account, amount, timestamp)
    VALUES (1, 2, 500, NOW());
    
    COMMIT;
ELSE
    ROLLBACK;
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Недостаточно средств';
END IF;
```

---

## ИНДЕКСЫ

### **СОЗДАНИЕ ИНДЕКСОВ**

```sql
-- ПРОСТОЙ ИНДЕКС
CREATE INDEX idx_last_name ON employees(last_name);

-- УНИКАЛЬНЫЙ ИНДЕКС
CREATE UNIQUE INDEX idx_email ON employees(email);

-- СОСТАВНОЙ ИНДЕКС
CREATE INDEX idx_department_salary ON employees(department_id, salary DESC);

-- ИНДЕКС ДЛЯ ТЕКСТОВОГО ПОИСКА
CREATE FULLTEXT INDEX idx_product_description ON products(description);

-- ЧАСТИЧНЫЙ ИНДЕКС (ТОЛЬКО ДЛЯ АКТИВНЫХ ЗАПИСЕЙ)
CREATE INDEX idx_active_employees ON employees(employee_id) 
WHERE is_active = TRUE;
```

### 🗑️ **УПРАВЛЕНИЕ ИНДЕКСАМИ**

```sql
-- ПОСМОТРЕТЬ ВСЕ ИНДЕКСЫ ТАБЛИЦЫ
SHOW INDEX FROM employees;

-- УДАЛИТЬ ИНДЕКС
DROP INDEX idx_last_name ON employees;

-- ПЕРЕИМЕНОВАТЬ ИНДЕКС (MySQL 8.0+)
ALTER TABLE employees 
RENAME INDEX old_index_name TO new_index_name;

-- АНАЛИЗ ИСПОЛЬЗОВАНИЯ ИНДЕКСОВ
EXPLAIN SELECT * FROM employees WHERE last_name = 'Петров';
```

---

## ПРАКТИЧЕСКИЕ ПРИМЕРЫ

### **ТОП-5 СОТРУДНИКОВ ПО ЗАРПЛАТЕ В КАЖДОМ ОТДЕЛЕ**

```sql
WITH ranked_employees AS (
    SELECT 
        first_name,
        last_name,
        salary,
        department_id,
        ROW_NUMBER() OVER (
            PARTITION BY department_id 
            ORDER BY salary DESC
        ) as rank_in_dept
    FROM employees
    WHERE is_active = TRUE
)
SELECT 
    re.first_name,
    re.last_name,
    re.salary,
    d.department_name,
    re.rank_in_dept
FROM ranked_employees re
JOIN departments d ON re.department_id = d.department_id
WHERE re.rank_in_dept <= 5
ORDER BY d.department_name, re.rank_in_dept;
```

### **ЕЖЕМЕСЯЧНАЯ СТАТИСТИКА ПРИЕМА НА РАБОТУ**

```sql
SELECT 
    EXTRACT(YEAR FROM hire_date) as year,
    EXTRACT(MONTH FROM hire_date) as month,
    COUNT(*) as hired_count,
    AVG(salary) as avg_salary,
    SUM(salary) as total_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary,
    -- процент от общего количества
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM employees), 2) as percent_of_total
FROM employees
GROUP BY 
    EXTRACT(YEAR FROM hire_date),
    EXTRACT(MONTH FROM hire_date)
ORDER BY year DESC, month DESC;
```

### **ВЫЯВЛЕНИЕ ДУБЛИКАТОВ**

```sql
SELECT 
    email,
    COUNT(*) as duplicate_count,
    GROUP_CONCAT(employee_id) as duplicate_ids,
    GROUP_CONCAT(CONCAT(first_name, ' ', last_name)) as duplicate_names
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;
```

### **ИЕРАРХИЧЕСКИЙ ЗАПРОС ДЛЯ ОРГСТРУКТУРЫ**

```sql
WITH RECURSIVE org_chart AS (
    -- БАЗОВЫЙ СЛУЧАЙ: САМЫЕ ГЛАВНЫЕ НАЧАЛЬНИКИ
    SELECT 
        employee_id,
        first_name,
        last_name,
        manager_id,
        0 as level,
        CAST(CONCAT(first_name, ' ', last_name) AS CHAR(1000)) as path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- РЕКУРСИВНЫЙ СЛУЧАЙ: ПОДЧИНЕННЫЕ
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        oc.level + 1,
        CAST(CONCAT(oc.path, ' -> ', e.first_name, ' ', e.last_name) AS CHAR(1000))
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT 
    employee_id,
    first_name,
    last_name,
    level,
    path
FROM org_chart
ORDER BY path;
```

---