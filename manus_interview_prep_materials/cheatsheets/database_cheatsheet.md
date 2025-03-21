# Шпаргалка по базам данных и SQL

## Основы реляционных баз данных

### Ключевые концепции
- **Таблица**: структура для хранения данных в виде строк и столбцов
- **Строка (запись)**: набор данных об одном объекте
- **Столбец (поле)**: атрибут, описывающий объект
- **Первичный ключ (Primary Key)**: уникальный идентификатор строки
- **Внешний ключ (Foreign Key)**: ссылка на первичный ключ в другой таблице
- **Индекс**: структура данных для ускорения поиска
- **Схема**: логическая структура базы данных

### Типы данных в PostgreSQL
- **Числовые**: INTEGER, BIGINT, NUMERIC, REAL, DOUBLE PRECISION
- **Символьные**: CHAR, VARCHAR, TEXT
- **Дата и время**: DATE, TIME, TIMESTAMP, INTERVAL
- **Логические**: BOOLEAN
- **Бинарные**: BYTEA
- **JSON**: JSON, JSONB
- **Массивы**: INTEGER[], TEXT[], и т.д.
- **Специальные**: UUID, CIDR, INET, MAC

### Нормализация
- **1NF**: атомарность значений, отсутствие повторяющихся групп
- **2NF**: соответствие 1NF + отсутствие частичных зависимостей
- **3NF**: соответствие 2NF + отсутствие транзитивных зависимостей
- **BCNF**: соответствие 3NF + все детерминанты являются потенциальными ключами
- **4NF**: соответствие BCNF + отсутствие многозначных зависимостей
- **5NF**: соответствие 4NF + отсутствие зависимостей соединения

## SQL: Основные операции

### SELECT: Выборка данных
```sql
-- Базовый SELECT
SELECT column1, column2 FROM table_name;

-- SELECT с условием
SELECT * FROM employees WHERE department = 'IT';

-- SELECT с сортировкой
SELECT * FROM products ORDER BY price DESC;

-- SELECT с ограничением количества строк
SELECT * FROM logs LIMIT 10;

-- SELECT с пропуском строк
SELECT * FROM logs OFFSET 10 LIMIT 10;

-- SELECT с группировкой
SELECT department, COUNT(*) FROM employees GROUP BY department;

-- SELECT с фильтрацией групп
SELECT department, COUNT(*) 
FROM employees 
GROUP BY department 
HAVING COUNT(*) > 5;

-- SELECT с подзапросом
SELECT * FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### INSERT: Добавление данных
```sql
-- Добавление одной строки
INSERT INTO employees (name, department, salary) 
VALUES ('John Doe', 'IT', 75000);

-- Добавление нескольких строк
INSERT INTO employees (name, department, salary) 
VALUES 
  ('Jane Smith', 'HR', 65000),
  ('Bob Johnson', 'Finance', 80000);

-- Добавление с SELECT
INSERT INTO employee_archive 
SELECT * FROM employees WHERE end_date < CURRENT_DATE;
```

### UPDATE: Изменение данных
```sql
-- Обновление всех строк
UPDATE employees SET salary = salary * 1.05;

-- Обновление с условием
UPDATE employees 
SET salary = salary * 1.1 
WHERE department = 'IT';

-- Обновление нескольких столбцов
UPDATE employees 
SET salary = salary * 1.1, updated_at = CURRENT_TIMESTAMP 
WHERE id = 123;

-- Обновление с подзапросом
UPDATE products 
SET price = price * 1.1 
WHERE category_id IN (SELECT id FROM categories WHERE name = 'Electronics');
```

### DELETE: Удаление данных
```sql
-- Удаление всех строк
DELETE FROM temporary_logs;

-- Удаление с условием
DELETE FROM employees WHERE end_date < CURRENT_DATE;

-- Удаление с подзапросом
DELETE FROM orders 
WHERE customer_id IN (SELECT id FROM customers WHERE status = 'inactive');
```

### TRUNCATE: Быстрое удаление всех данных
```sql
TRUNCATE TABLE logs;
```

## SQL: Соединения таблиц

### INNER JOIN: Внутреннее соединение
```sql
-- Соединение двух таблиц
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- Соединение трех таблиц
SELECT e.name, d.name AS department, p.name AS project
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
INNER JOIN project_assignments pa ON e.id = pa.employee_id
INNER JOIN projects p ON pa.project_id = p.id;
```

### LEFT JOIN: Левое внешнее соединение
```sql
-- Все сотрудники, включая тех, у кого нет отдела
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

### RIGHT JOIN: Правое внешнее соединение
```sql
-- Все отделы, включая те, где нет сотрудников
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

### FULL JOIN: Полное внешнее соединение
```sql
-- Все сотрудники и все отделы
SELECT e.name, d.name AS department
FROM employees e
FULL JOIN departments d ON e.department_id = d.id;
```

### CROSS JOIN: Перекрестное соединение
```sql
-- Все возможные комбинации сотрудников и отделов
SELECT e.name, d.name AS department
FROM employees e
CROSS JOIN departments d;
```

### SELF JOIN: Соединение таблицы с самой собой
```sql
-- Сотрудники и их менеджеры
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

## SQL: Агрегатные функции

### Основные агрегатные функции
```sql
-- COUNT: Количество строк
SELECT COUNT(*) FROM employees;
SELECT COUNT(DISTINCT department) FROM employees;

-- SUM: Сумма значений
SELECT SUM(salary) FROM employees;

-- AVG: Среднее значение
SELECT AVG(salary) FROM employees;

-- MIN/MAX: Минимальное/максимальное значение
SELECT MIN(salary), MAX(salary) FROM employees;

-- GROUP_CONCAT (в PostgreSQL: string_agg): Объединение строк
SELECT department, string_agg(name, ', ') FROM employees GROUP BY department;
```

### Группировка и агрегация
```sql
-- Группировка по одному столбцу
SELECT department, COUNT(*), AVG(salary)
FROM employees
GROUP BY department;

-- Группировка по нескольким столбцам
SELECT department, job_title, COUNT(*), AVG(salary)
FROM employees
GROUP BY department, job_title;

-- Фильтрация групп с HAVING
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- Сортировка результатов агрегации
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;
```

## SQL: Подзапросы

### Скалярные подзапросы (возвращают одно значение)
```sql
-- Сотрудники с зарплатой выше средней
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Подзапросы, возвращающие список
```sql
-- Сотрудники из отделов с высокой средней зарплатой
SELECT name, department_id
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING AVG(salary) > 70000
);
```

### Коррелированные подзапросы
```sql
-- Сотрудники с зарплатой выше средней в своем отделе
SELECT e1.name, e1.salary, e1.department_id
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

### Подзапросы в SELECT
```sql
-- Список отделов с количеством сотрудников
SELECT 
    d.name,
    (SELECT COUNT(*) FROM employees e WHERE e.department_id = d.id) AS employee_count
FROM departments d;
```

### Подзапросы в FROM
```sql
-- Средняя зарплата по отделам с количеством сотрудников
SELECT 
    dept_stats.department,
    dept_stats.avg_salary,
    dept_stats.employee_count
FROM (
    SELECT 
        d.name AS department,
        AVG(e.salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees e
    JOIN departments d ON e.department_id = d.id
    GROUP BY d.name
) AS dept_stats
WHERE dept_stats.employee_count > 5;
```

## SQL: Оконные функции

### Базовые оконные функции
```sql
-- ROW_NUMBER: Номер строки
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;

-- RANK: Ранг с пропусками
SELECT 
    name,
    department,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;

-- DENSE_RANK: Ранг без пропусков
SELECT 
    name,
    department,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;

-- NTILE: Разделение на группы
SELECT 
    name,
    department,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS salary_quartile
FROM employees;
```

### Разделение на секции (PARTITION BY)
```sql
-- Ранжирование в пределах отдела
SELECT 
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_salary_rank
FROM employees;

-- Процент от максимальной зарплаты в отделе
SELECT 
    name,
    department,
    salary,
    salary / MAX(salary) OVER (PARTITION BY department) * 100 AS percent_of_max
FROM employees;
```

### Агрегатные функции в окне
```sql
-- Отклонение от средней зарплаты в отделе
SELECT 
    name,
    department,
    salary,
    salary - AVG(salary) OVER (PARTITION BY department) AS salary_diff_from_avg
FROM employees;

-- Накопительная сумма
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;

-- Скользящее среднее
SELECT 
    date,
    amount,
    AVG(amount) OVER (
        ORDER BY date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3days
FROM transactions;
```

### Функции смещения
```sql
-- LAG: Предыдущее значение
SELECT 
    date,
    amount,
    LAG(amount) OVER (ORDER BY date) AS prev_amount,
    amount - LAG(amount) OVER (ORDER BY date) AS amount_change
FROM transactions;

-- LEAD: Следующее значение
SELECT 
    date,
    amount,
    LEAD(amount) OVER (ORDER BY date) AS next_amount
FROM transactions;

-- FIRST_VALUE: Первое значение в окне
SELECT 
    date,
    amount,
    FIRST_VALUE(amount) OVER (
        PARTITION BY EXTRACT(MONTH FROM date) 
        ORDER BY date
    ) AS first_amount_in_month
FROM transactions;
```

## SQL: Транзакции

### Основы транзакций
```sql
-- Начало транзакции
BEGIN;

-- Выполнение операций
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Проверка условия
SELECT balance FROM accounts WHERE id = 1;
-- Если баланс отрицательный, откатываем транзакцию
-- ROLLBACK;

-- Фиксация транзакции
COMMIT;
```

### Уровни изоляции транзакций
```sql
-- Установка уровня изоляции для текущей транзакции
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- операции
COMMIT;

-- Доступные уровни изоляции:
-- READ UNCOMMITTED: Чтение незафиксированных данных
-- READ COMMITTED: Чтение только зафиксированных данных
-- REPEATABLE READ: Повторное чтение возвращает те же данные
-- SERIALIZABLE: Полная изоляция транзакций
```

### Блокировки
```sql
-- Явная блокировка таблицы
BEGIN;
LOCK TABLE accounts IN EXCLUSIVE MODE;
-- операции
COMMIT;

-- Блокировка строк при SELECT
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- операции с заблокированной строкой
COMMIT;

-- Блокировка строк при SELECT без ожидания
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
-- операции с заблокированной строкой
COMMIT;
```

## SQL: Индексы

### Типы индексов в PostgreSQL
- **B-tree**: стандартный индекс для операций сравнения
- **Hash**: для операций равенства
- **GiST**: обобщенное поисковое дерево для сложных типов данных
- **SP-GiST**: разделяемое пространственное дерево
- **GIN**: инвертированный индекс для составных значений
- **BRIN**: блочный индекс для больших таблиц с упорядоченными данными

### Создание индексов
```sql
-- Простой индекс
CREATE INDEX idx_employees_name ON employees(name);

-- Составной индекс
CREATE INDEX idx_employees_dept_name ON employees(department_id, name);

-- Уникальный индекс
CREATE UNIQUE INDEX idx_employees_email ON employees(email);

-- Частичный индекс
CREATE INDEX idx_active_employees ON employees(name) WHERE status = 'active';

-- Функциональный индекс
CREATE INDEX idx_employees_lower_name ON employees(LOWER(name));

-- Индекс для полнотекстового поиска
CREATE INDEX idx_products_description ON products USING GIN(to_tsvector('english', description));
```

### Управление индексами
```sql
-- Просмотр индексов таблицы
SELECT * FROM pg_indexes WHERE tablename = 'employees';

-- Удаление индекса
DROP INDEX idx_employees_name;

-- Перестроение индекса
REINDEX INDEX idx_employees_name;
REINDEX TABLE employees;
```

### Рекомендации по индексам
1. Индексируйте столбцы, используемые в условиях WHERE, JOIN и ORDER BY
2. Используйте составные индексы для часто встречающихся комбинаций условий
3. Учитывайте порядок столбцов в составном индексе
4. Избегайте индексирования столбцов с низкой кардинальностью
5. Используйте частичные индексы для фильтрации по часто используемым условиям
6. Регулярно анализируйте и перестраивайте индексы

## SQL: Оптимизация запросов

### EXPLAIN: Анализ плана выполнения
```sql
-- Базовый EXPLAIN
EXPLAIN SELECT * FROM employees WHERE department_id = 5;

-- EXPLAIN с выполнением запроса
EXPLAIN ANALYZE SELECT * FROM employees WHERE department_id = 5;

-- Подробный вывод
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT * FROM employees WHERE department_id = 5;
```

### Типичные операции в плане выполнения
- **Seq Scan**: последовательное сканирование таблицы
- **Index Scan**: сканирование индекса с доступом к таблице
- **Index Only Scan**: сканирование только индекса
- **Bitmap Heap Scan**: двухфазное сканирование с использованием битовой карты
- **Nested Loop**: вложенный цикл для соединения таблиц
- **Hash Join**: соединение с использованием хеш-таблицы
- **Merge Join**: соединение отсортированных таблиц

### Оптимизация запросов
```sql
-- Использование индексов
CREATE INDEX idx_employees_dept ON employees(department_id);

-- Переписывание запросов
-- Вместо:
SELECT * FROM employees WHERE LOWER(name) = 'john smith';
-- Используйте:
SELECT * FROM employees WHERE name ILIKE 'john smith';

-- Избегайте функций на индексированных столбцах
-- Вместо:
SELECT * FROM employees WHERE EXTRACT(YEAR FROM hire_date) = 2022;
-- Используйте:
SELECT * FROM employees WHERE hire_date BETWEEN '2022-01-01' AND '2022-12-31';

-- Используйте EXISTS вместо IN для больших подзапросов
-- Вместо:
SELECT * FROM departments 
WHERE id IN (SELECT department_id FROM employees WHERE salary > 100000);
-- Используйте:
SELECT * FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.department_id = d.id AND e.salary > 100000);
```

### Денормализация для производительности
```sql
-- Добавление избыточных данных для ускорения запросов
ALTER TABLE orders ADD COLUMN customer_name VARCHAR(100);

-- Материализованные представления
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS total_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- Обновление материализованного представления
REFRESH MATERIALIZED VIEW monthly_sales;
```

## SQL: Репликация и шардирование

### Репликация в PostgreSQL
- **Физическая репликация**: побитовое копирование данных
  - Streaming Replication: непрерывная передача WAL
  - Cascading Replication: репликация через промежуточные серверы
- **Логическая репликация**: репликация на уровне изменений строк
  - Позволяет выборочную репликацию таблиц
  - Поддерживает репликацию между разными версиями PostgreSQL

### Настройка физической репликации
```sql
-- На мастере:
-- postgresql.conf
wal_level = replica
max_wal_senders = 10
wal_keep_segments = 32

-- pg_hba.conf
host replication replicator 192.168.1.0/24 md5

-- Создание пользователя для репликации
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'password';

-- На реплике:
-- Создание базовой копии
pg_basebackup -h master_host -D /var/lib/postgresql/data -U replicator -P -v

-- recovery.conf
standby_mode = 'on'
primary_conninfo = 'host=master_host port=5432 user=replicator password=password'
trigger_file = '/tmp/postgresql.trigger'
```

### Настройка логической репликации
```sql
-- На издателе:
-- postgresql.conf
wal_level = logical
max_replication_slots = 10
max_wal_senders = 10

-- Создание публикации
CREATE PUBLICATION my_publication FOR TABLE employees, departments;

-- На подписчике:
-- Создание подписки
CREATE SUBSCRIPTION my_subscription
CONNECTION 'host=publisher_host port=5432 dbname=mydb user=replicator password=password'
PUBLICATION my_publication;
```

### Шардирование
- **Вертикальное шардирование**: разделение таблиц по разным серверам
- **Горизонтальное шардирование**: разделение строк одной таблицы по разным серверам

#### Стратегии шардирования
1. **По диапазону**: разделение по диапазонам значений ключа
2. **По хешу**: распределение по хешу ключа
3. **По списку**: распределение по спискам значений
4. **По функции**: произвольная функция распределения

#### Реализация шардирования в PostgreSQL
- **Секционирование таблиц**: разделение таблицы на подтаблицы
```sql
-- Декларативное секционирование
CREATE TABLE orders (
    id SERIAL,
    customer_id INTEGER,
    order_date DATE,
    amount NUMERIC,
    PRIMARY KEY (id, order_date)
) PARTITION BY RANGE (order_date);

-- Создание секций
CREATE TABLE orders_2022 PARTITION OF orders
    FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');

CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
```

- **Foreign Data Wrappers**: доступ к данным на удаленных серверах
```sql
-- Создание внешнего сервера
CREATE SERVER shard1 FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'shard1_host', port '5432', dbname 'mydb');

-- Создание пользовательского сопоставления
CREATE USER MAPPING FOR current_user SERVER shard1
OPTIONS (user 'remote_user', password 'password');

-- Создание внешней таблицы
CREATE FOREIGN TABLE orders_shard1 (
    id INTEGER,
    customer_id INTEGER,
    order_date DATE,
    amount NUMERIC
) SERVER shard1 OPTIONS (table_name 'orders');

-- Создание представления для объединения шардов
CREATE VIEW orders AS
    SELECT * FROM orders_shard1
    UNION ALL
    SELECT * FROM orders_shard2;
```

## NoSQL базы данных

### Типы NoSQL баз данных
1. **Key-Value**: Redis, DynamoDB
2. **Document**: MongoDB, CouchDB
3. **Column-Family**: Cassandra, HBase
4. **Graph**: Neo4j, JanusGraph

### Redis: Key-Value хранилище
```
# Установка значения
SET user:1:name "John Doe"

# Получение значения
GET user:1:name

# Хеш-таблицы
HSET user:1 name "John Doe" email "john@example.com" age 30
HGET user:1 name
HGETALL user:1

# Списки
LPUSH notifications:user:1 "New message"
RPUSH notifications:user:1 "Friend request"
LRANGE notifications:user:1 0 -1

# Множества
SADD tags:post:1 "golang" "database" "nosql"
SMEMBERS tags:post:1
SINTER tags:post:1 tags:post:2

# Упорядоченные множества
ZADD leaderboard 100 "user:1" 85 "user:2" 95 "user:3"
ZRANGE leaderboard 0 -1 WITHSCORES
ZREVRANGE leaderboard 0 2 WITHSCORES
```

### MongoDB: Document хранилище
```javascript
// Создание коллекции
db.createCollection("employees")

// Вставка документа
db.employees.insertOne({
    name: "John Doe",
    department: "IT",
    salary: 75000,
    skills: ["Python", "JavaScript", "Docker"],
    contact: {
        email: "john@example.com",
        phone: "123-456-7890"
    }
})

// Вставка нескольких документов
db.employees.insertMany([
    { name: "Jane Smith", department: "HR", salary: 65000 },
    { name: "Bob Johnson", department: "Finance", salary: 80000 }
])

// Поиск документов
db.employees.find({ department: "IT" })
db.employees.find({ salary: { $gt: 70000 } })
db.employees.find({ "skills": "Python" })

// Проекция (выбор полей)
db.employees.find({ department: "IT" }, { name: 1, salary: 1, _id: 0 })

// Сортировка
db.employees.find().sort({ salary: -1 })

// Лимит и пропуск
db.employees.find().skip(10).limit(5)

// Обновление документа
db.employees.updateOne(
    { name: "John Doe" },
    { $set: { salary: 80000 } }
)

// Обновление нескольких документов
db.employees.updateMany(
    { department: "IT" },
    { $inc: { salary: 5000 } }
)

// Удаление документа
db.employees.deleteOne({ name: "John Doe" })

// Удаление нескольких документов
db.employees.deleteMany({ department: "IT" })

// Агрегация
db.employees.aggregate([
    { $match: { department: "IT" } },
    { $group: { _id: null, avgSalary: { $avg: "$salary" } } }
])

// Индексы
db.employees.createIndex({ name: 1 })
db.employees.createIndex({ department: 1, salary: -1 })
```

### Cassandra: Column-Family хранилище
```cql
-- Создание пространства ключей
CREATE KEYSPACE my_keyspace
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};

-- Использование пространства ключей
USE my_keyspace;

-- Создание таблицы
CREATE TABLE employees (
    id UUID PRIMARY KEY,
    name TEXT,
    department TEXT,
    salary INT,
    hire_date TIMESTAMP
);

-- Вставка данных
INSERT INTO employees (id, name, department, salary, hire_date)
VALUES (uuid(), 'John Doe', 'IT', 75000, '2022-01-15');

-- Выборка данных
SELECT * FROM employees WHERE id = 123e4567-e89b-12d3-a456-426614174000;

-- Создание индекса
CREATE INDEX ON employees (department);

-- Выборка с использованием индекса
SELECT * FROM employees WHERE department = 'IT';

-- Обновление данных
UPDATE employees
SET salary = 80000
WHERE id = 123e4567-e89b-12d3-a456-426614174000;

-- Удаление данных
DELETE FROM employees
WHERE id = 123e4567-e89b-12d3-a456-426614174000;
```

## Практические задачи для собеседования

### Задача 1: Найти дубликаты
```sql
-- Найти дубликаты в таблице
SELECT name, email, COUNT(*)
FROM users
GROUP BY name, email
HAVING COUNT(*) > 1;
```

### Задача 2: Найти второе наибольшее значение
```sql
-- Вариант 1: с использованием LIMIT/OFFSET
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Вариант 2: с использованием подзапроса
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Вариант 3: с использованием оконных функций
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
    FROM employees
) ranked
WHERE rnk = 2;
```

### Задача 3: Найти отделы с зарплатой выше средней
```sql
SELECT d.name, AVG(e.salary) as avg_salary
FROM departments d
JOIN employees e ON d.id = e.department_id
GROUP BY d.name
HAVING AVG(e.salary) > (SELECT AVG(salary) FROM employees);
```

### Задача 4: Найти сотрудников без руководителя
```sql
SELECT name
FROM employees
WHERE manager_id IS NULL
OR manager_id NOT IN (SELECT id FROM employees);
```

### Задача 5: Найти пользователей, сделавших заказы в определенный период
```sql
SELECT DISTINCT u.name
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.order_date BETWEEN '2023-01-01' AND '2023-01-31';
```

### Задача 6: Найти товары, которые никогда не заказывали
```sql
SELECT p.name
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
WHERE oi.id IS NULL;
```

### Задача 7: Найти пользователей с заказами на определенную сумму
```sql
SELECT u.name, SUM(o.amount) as total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.name
HAVING SUM(o.amount) > 1000;
```

### Задача 8: Найти самый популярный товар в каждой категории
```sql
WITH ranked_products AS (
    SELECT 
        p.name,
        p.category_id,
        COUNT(oi.id) as order_count,
        RANK() OVER (PARTITION BY p.category_id ORDER BY COUNT(oi.id) DESC) as rnk
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.name, p.category_id
)
SELECT c.name as category, rp.name as product, rp.order_count
FROM ranked_products rp
JOIN categories c ON rp.category_id = c.id
WHERE rp.rnk = 1;
```

### Задача 9: Найти пользователей, которые сделали заказы всех товаров в категории
```sql
SELECT u.name
FROM users u
WHERE NOT EXISTS (
    SELECT p.id
    FROM products p
    WHERE p.category_id = 5
    AND NOT EXISTS (
        SELECT 1
        FROM orders o
        JOIN order_items oi ON o.id = oi.order_id
        WHERE o.user_id = u.id
        AND oi.product_id = p.id
    )
);
```

### Задача 10: Найти среднее время между заказами для каждого пользователя
```sql
SELECT 
    u.name,
    AVG(next_order_date - o.order_date) as avg_days_between_orders
FROM (
    SELECT 
        user_id,
        order_date,
        LEAD(order_date) OVER (PARTITION BY user_id ORDER BY order_date) as next_order_date
    FROM orders
) o
JOIN users u ON o.user_id = u.id
WHERE next_order_date IS NOT NULL
GROUP BY u.name;
```

## Советы для собеседования по базам данных

### Ключевые темы для подготовки
1. **Основы SQL**: SELECT, JOIN, агрегатные функции, подзапросы
2. **Индексы**: типы, создание, использование, оптимизация
3. **Транзакции**: ACID, уровни изоляции, блокировки
4. **Нормализация**: формы нормализации, денормализация
5. **Оптимизация запросов**: EXPLAIN, планы выполнения, переписывание запросов
6. **Репликация и шардирование**: типы, настройка, использование
7. **NoSQL**: типы баз данных, сценарии использования, сравнение с SQL

### Типичные вопросы на собеседовании
1. **Что такое индекс и как он работает?**
   - Индекс - структура данных для ускорения поиска в таблице
   - B-tree индексы хранят отсортированные ключи с указателями на строки
   - Ускоряют поиск, но замедляют вставку и обновление

2. **Объясните разницу между INNER JOIN и LEFT JOIN.**
   - INNER JOIN возвращает строки, где есть совпадения в обеих таблицах
   - LEFT JOIN возвращает все строки из левой таблицы и совпадающие из правой

3. **Что такое транзакция и какие свойства ACID она обеспечивает?**
   - Атомарность: все или ничего
   - Согласованность: сохранение целостности данных
   - Изолированность: транзакции не влияют друг на друга
   - Долговечность: изменения сохраняются после фиксации

4. **Какие уровни изоляции транзакций существуют и чем они отличаются?**
   - READ UNCOMMITTED: возможно чтение "грязных" данных
   - READ COMMITTED: чтение только зафиксированных данных
   - REPEATABLE READ: повторное чтение возвращает те же данные
   - SERIALIZABLE: полная изоляция транзакций

5. **Как оптимизировать медленный запрос?**
   - Анализ плана выполнения с EXPLAIN
   - Добавление подходящих индексов
   - Переписывание запроса (избегание функций на индексированных столбцах)
   - Денормализация для сложных запросов
   - Материализованные представления

6. **Когда использовать NoSQL вместо SQL?**
   - Большие объемы данных с простой структурой
   - Горизонтальное масштабирование
   - Гибкая схема данных
   - Высокая пропускная способность записи

7. **Как реализовать шардирование базы данных?**
   - Выбор ключа шардирования
   - Определение стратегии (по диапазону, хешу, списку)
   - Настройка маршрутизации запросов
   - Обработка запросов, затрагивающих несколько шардов

8. **Как обеспечить высокую доступность базы данных?**
   - Репликация (Master-Slave, Master-Master)
   - Автоматическое переключение при отказе (Failover)
   - Распределение по нескольким дата-центрам
   - Регулярное резервное копирование

### Практические советы
1. **Объясняйте свой ход мыслей**: проговаривайте, как вы подходите к решению задачи
2. **Начинайте с простого решения**: сначала напишите работающий запрос, затем оптимизируйте
3. **Обсуждайте компромиссы**: каждое решение имеет плюсы и минусы
4. **Задавайте уточняющие вопросы**: убедитесь, что вы правильно поняли задачу
5. **Используйте правильную терминологию**: демонстрируйте знание предметной области
