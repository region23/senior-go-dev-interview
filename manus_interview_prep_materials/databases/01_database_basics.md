# Основы баз данных и SQL

В этом документе представлены основные концепции баз данных и SQL, которые часто проверяются на технических интервью в компаниях уровня Яндекс и Озон. Особое внимание уделено темам, которые вы отметили как проблемные: транзакции, индексы и репликация.

## 1. Реляционные базы данных

### 1.1 Основные концепции

**Реляционная модель данных** основана на представлении данных в виде таблиц (отношений), состоящих из строк (кортежей) и столбцов (атрибутов). Ключевые особенности:

- **Таблицы (отношения)**: Структурированные наборы данных с определенными столбцами
- **Строки (кортежи)**: Отдельные записи в таблице
- **Столбцы (атрибуты)**: Поля с определенным типом данных
- **Первичные ключи**: Уникальные идентификаторы строк
- **Внешние ключи**: Ссылки на первичные ключи других таблиц
- **Ограничения**: Правила, обеспечивающие целостность данных

**Нормализация** - процесс организации данных для минимизации избыточности и зависимостей. Основные нормальные формы:

- **1NF**: Атомарность значений (одно значение в ячейке)
- **2NF**: Все неключевые атрибуты полностью зависят от первичного ключа
- **3NF**: Нет транзитивных зависимостей неключевых атрибутов
- **BCNF**: Каждая нетривиальная функциональная зависимость ведет к суперключу
- **4NF, 5NF**: Дополнительные ограничения для многозначных зависимостей

### 1.2 Основы SQL

**SQL (Structured Query Language)** - язык для управления реляционными базами данных. Основные категории команд:

**DDL (Data Definition Language)**:
```sql
-- Создание таблицы
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Изменение таблицы
ALTER TABLE users ADD COLUMN last_login TIMESTAMP;

-- Удаление таблицы
DROP TABLE users;
```

**DML (Data Manipulation Language)**:
```sql
-- Вставка данных
INSERT INTO users (username, email) VALUES ('john_doe', 'john@example.com');

-- Обновление данных
UPDATE users SET last_login = CURRENT_TIMESTAMP WHERE username = 'john_doe';

-- Удаление данных
DELETE FROM users WHERE id = 1;
```

**DQL (Data Query Language)**:
```sql
-- Базовый SELECT
SELECT id, username, email FROM users WHERE id > 10 ORDER BY created_at DESC LIMIT 10;

-- Агрегация
SELECT COUNT(*), DATE(created_at) as date 
FROM users 
GROUP BY date 
HAVING COUNT(*) > 5;
```

**DCL (Data Control Language)**:
```sql
-- Предоставление прав
GRANT SELECT, INSERT ON users TO app_user;

-- Отзыв прав
REVOKE INSERT ON users FROM app_user;
```

**TCL (Transaction Control Language)**:
```sql
-- Начало транзакции
BEGIN;

-- Фиксация изменений
COMMIT;

-- Отмена изменений
ROLLBACK;
```

### 1.3 Соединения таблиц (JOIN)

Соединения позволяют комбинировать данные из нескольких таблиц:

**INNER JOIN**: Возвращает строки, когда есть совпадения в обеих таблицах
```sql
SELECT u.username, o.order_id, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

**LEFT JOIN**: Возвращает все строки из левой таблицы и совпадающие из правой
```sql
SELECT u.username, o.order_id, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

**RIGHT JOIN**: Возвращает все строки из правой таблицы и совпадающие из левой
```sql
SELECT u.username, o.order_id, o.amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;
```

**FULL JOIN**: Возвращает строки, когда есть совпадения в одной из таблиц
```sql
SELECT u.username, o.order_id, o.amount
FROM users u
FULL JOIN orders o ON u.id = o.user_id;
```

**CROSS JOIN**: Декартово произведение таблиц
```sql
SELECT u.username, p.product_name
FROM users u
CROSS JOIN products p;
```

**SELF JOIN**: Соединение таблицы с самой собой
```sql
SELECT e1.name as employee, e2.name as manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;
```

### 1.4 Подзапросы

Подзапросы (вложенные запросы) позволяют использовать результаты одного запроса в другом:

**Скалярные подзапросы** (возвращают одно значение):
```sql
SELECT username, email
FROM users
WHERE id = (SELECT user_id FROM orders WHERE order_id = 12345);
```

**Строчные подзапросы** (возвращают одну строку):
```sql
SELECT *
FROM products
WHERE (price, category_id) = (SELECT MAX(price), 5 FROM products);
```

**Табличные подзапросы** (возвращают таблицу):
```sql
SELECT u.username, o.total_amount
FROM users u
JOIN (
    SELECT user_id, SUM(amount) as total_amount
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id;
```

**Коррелированные подзапросы** (ссылаются на внешний запрос):
```sql
SELECT username, email
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id AND o.amount > 1000
);
```

### 1.5 Оконные функции

Оконные функции выполняют вычисления по набору строк, связанных с текущей строкой:

```sql
-- Ранжирование пользователей по сумме заказов
SELECT 
    username,
    total_amount,
    RANK() OVER (ORDER BY total_amount DESC) as rank,
    DENSE_RANK() OVER (ORDER BY total_amount DESC) as dense_rank,
    ROW_NUMBER() OVER (ORDER BY total_amount DESC) as row_num
FROM (
    SELECT u.username, SUM(o.amount) as total_amount
    FROM users u
    JOIN orders o ON u.id = o.user_id
    GROUP BY u.username
) t;

-- Скользящая сумма по дням
SELECT 
    date,
    daily_sales,
    SUM(daily_sales) OVER (ORDER BY date) as running_total,
    AVG(daily_sales) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as weekly_avg
FROM daily_sales_report;
```

## 2. Транзакции

### 2.1 Основы транзакций

**Транзакция** - логическая единица работы, которая должна быть выполнена полностью или не выполнена вообще. Транзакции обеспечивают **ACID** свойства:

- **Атомарность (Atomicity)**: Транзакция выполняется полностью или не выполняется вообще
- **Согласованность (Consistency)**: Транзакция переводит БД из одного согласованного состояния в другое
- **Изолированность (Isolation)**: Результаты незавершенной транзакции не видны другим транзакциям
- **Долговечность (Durability)**: После завершения транзакции изменения сохраняются даже при сбоях

**Пример транзакции в PostgreSQL**:
```sql
BEGIN;

-- Снятие денег с одного счета
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Проверка, что баланс не стал отрицательным
SELECT balance FROM accounts WHERE id = 1;
-- Если баланс отрицательный, выполняем ROLLBACK

-- Пополнение другого счета
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

### 2.2 Уровни изоляции транзакций

Уровни изоляции определяют, как транзакции взаимодействуют друг с другом:

**READ UNCOMMITTED**: Транзакция может видеть незафиксированные изменения других транзакций (грязное чтение)
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
BEGIN;
-- операции
COMMIT;
```

**READ COMMITTED**: Транзакция видит только зафиксированные изменения других транзакций (предотвращает грязное чтение)
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN;
-- операции
COMMIT;
```

**REPEATABLE READ**: Гарантирует, что повторное чтение одних и тех же данных даст одинаковый результат (предотвращает неповторяемое чтение)
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN;
-- операции
COMMIT;
```

**SERIALIZABLE**: Самый строгий уровень, транзакции выполняются так, как если бы они выполнялись последовательно (предотвращает фантомное чтение)
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN;
-- операции
COMMIT;
```

### 2.3 Проблемы параллельного выполнения транзакций

При параллельном выполнении транзакций могут возникать следующие проблемы:

- **Грязное чтение (Dirty Read)**: Транзакция читает данные, измененные другой незавершенной транзакцией
- **Неповторяемое чтение (Non-repeatable Read)**: Повторное чтение одних и тех же данных в рамках одной транзакции дает разные результаты
- **Фантомное чтение (Phantom Read)**: Транзакция повторно выполняет запрос и получает другой набор строк
- **Потерянное обновление (Lost Update)**: Одна транзакция перезаписывает изменения другой транзакции

**Таблица уровней изоляции и проблем**:

| Уровень изоляции | Грязное чтение | Неповторяемое чтение | Фантомное чтение |
|------------------|----------------|----------------------|------------------|
| READ UNCOMMITTED | Возможно       | Возможно             | Возможно         |
| READ COMMITTED   | Невозможно     | Возможно             | Возможно         |
| REPEATABLE READ  | Невозможно     | Невозможно           | Возможно*        |
| SERIALIZABLE     | Невозможно     | Невозможно           | Невозможно       |

*В PostgreSQL REPEATABLE READ также предотвращает фантомное чтение.

### 2.4 Блокировки

Блокировки - механизм, обеспечивающий изоляцию транзакций:

**Типы блокировок**:
- **Разделяемые (Shared, S)**: Для чтения данных, несколько транзакций могут иметь S-блокировку
- **Исключительные (Exclusive, X)**: Для изменения данных, только одна транзакция может иметь X-блокировку
- **Намерение разделения (Intent Shared, IS)**: Указывает намерение получить S-блокировку на нижележащие объекты
- **Намерение исключения (Intent Exclusive, IX)**: Указывает намерение получить X-блокировку на нижележащие объекты

**Явные блокировки в PostgreSQL**:
```sql
-- Блокировка таблицы для чтения
LOCK TABLE users IN SHARE MODE;

-- Блокировка таблицы для записи
LOCK TABLE users IN EXCLUSIVE MODE;

-- Блокировка строки для обновления
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Блокировка строки для чтения
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
```

**Взаимоблокировки (Deadlocks)** возникают, когда две или более транзакций ждут освобождения ресурсов, заблокированных друг другом:
```
Транзакция 1: Блокирует ресурс A, ждет ресурс B
Транзакция 2: Блокирует ресурс B, ждет ресурс A
```

СУБД обычно обнаруживают взаимоблокировки и прерывают одну из транзакций.

## 3. Индексы

### 3.1 Основы индексирования

**Индекс** - структура данных, ускоряющая поиск записей в таблице. Индексы работают аналогично указателю в книге, позволяя быстро находить нужные данные без полного сканирования таблицы.

**Преимущества индексов**:
- Ускорение операций поиска, сортировки и соединения
- Обеспечение уникальности значений
- Оптимизация запросов с условиями WHERE, ORDER BY, GROUP BY

**Недостатки индексов**:
- Дополнительное использование дискового пространства
- Замедление операций вставки, обновления и удаления
- Необходимость обслуживания (перестроение, дефрагментация)

### 3.2 Типы индексов

**B-Tree индексы** - наиболее распространенный тип, подходит для большинства случаев:
```sql
-- Создание B-Tree индекса
CREATE INDEX idx_users_email ON users(email);

-- Составной индекс
CREATE INDEX idx_users_name_email ON users(last_name, first_name, email);
```

**Hash индексы** - эффективны только для операций точного соответствия (=):
```sql
-- Создание Hash индекса в PostgreSQL
CREATE INDEX idx_users_email_hash ON users USING HASH (email);
```

**GiST индексы** (Generalized Search Tree) - для полнотекстового поиска, геоданных:
```sql
-- Индекс для геоданных
CREATE INDEX idx_locations_position ON locations USING GIST (position);
```

**GIN индексы** (Generalized Inverted Index) - для массивов, JSON, полнотекстового поиска:
```sql
-- Индекс для массивов
CREATE INDEX idx_articles_tags ON articles USING GIN (tags);

-- Индекс для JSON
CREATE INDEX idx_users_metadata ON users USING GIN (metadata jsonb_path_ops);
```

**BRIN индексы** (Block Range INdex) - для очень больших таблиц с упорядоченными данными:
```sql
-- Индекс для временных рядов
CREATE INDEX idx_logs_timestamp ON logs USING BRIN (timestamp);
```

**Частичные индексы** - индексируют только подмножество строк:
```sql
-- Индекс только для активных пользователей
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
```

**Функциональные индексы** - индексируют результат функции:
```sql
-- Индекс для поиска без учета регистра
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
```

### 3.3 Стратегии индексирования

**Выбор столбцов для индексирования**:
- Столбцы, часто используемые в условиях WHERE
- Столбцы, используемые в JOIN
- Столбцы в ORDER BY и GROUP BY
- Столбцы с высокой кардинальностью (много уникальных значений)

**Составные индексы**:
- Порядок столбцов важен: сначала столбцы для точного соответствия, затем для диапазонов
- Учитывайте селективность столбцов
- Помните о правиле "левого префикса" - индекс используется только для начальных столбцов

**Пример оптимального составного индекса**:
```sql
-- Для запроса:
SELECT * FROM users 
WHERE status = 'active' AND city = 'Moscow' 
ORDER BY created_at DESC;

-- Оптимальный индекс:
CREATE INDEX idx_users_status_city_created 
ON users(status, city, created_at DESC);
```

### 3.4 Анализ использования индексов

**EXPLAIN** - команда для анализа плана выполнения запроса:
```sql
-- Базовый EXPLAIN
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- EXPLAIN с выполнением запроса и дополнительной информацией
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';
```

**Типы сканирования**:
- **Seq Scan**: Полное сканирование таблицы (медленно для больших таблиц)
- **Index Scan**: Сканирование индекса с доступом к таблице
- **Index Only Scan**: Сканирование только индекса (быстро)
- **Bitmap Index Scan**: Создание битовой карты для доступа к таблице
- **Index Scan Backward**: Сканирование индекса в обратном порядке

**Оптимизация индексов**:
- Регулярно анализируйте таблицы: `ANALYZE table_name;`
- Перестраивайте фрагментированные индексы: `REINDEX INDEX index_name;`
- Удаляйте неиспользуемые индексы
- Мониторьте размер индексов и их использование

## 4. Репликация и шардирование

### 4.1 Репликация

**Репликация** - процесс копирования данных с одного сервера БД (мастера) на другие серверы (реплики). Основные цели репликации:

- Повышение доступности данных
- Распределение нагрузки чтения
- Географическое распределение данных
- Резервное копирование и восстановление

**Типы репликации**:

**Синхронная репликация**:
- Транзакция считается завершенной только после подтверждения от реплик
- Обеспечивает строгую согласованность данных
- Увеличивает задержку записи
- Пример в PostgreSQL:
  ```sql
  -- Настройка синхронной репликации
  ALTER SYSTEM SET synchronous_standby_names = 'replica1, replica2';
  ```

**Асинхронная репликация**:
- Мастер не ждет подтверждения от реплик
- Меньшая задержка записи
- Возможна временная несогласованность данных
- Пример в PostgreSQL:
  ```sql
  -- Настройка асинхронной репликации
  ALTER SYSTEM SET synchronous_standby_names = '';
  ```

**Полусинхронная репликация**:
- Компромисс между синхронной и асинхронной
- Мастер ждет подтверждения от части реплик
- Пример в PostgreSQL:
  ```sql
  -- Настройка полусинхронной репликации (ждем подтверждения от одной реплики)
  ALTER SYSTEM SET synchronous_standby_names = 'FIRST 1 (replica1, replica2, replica3)';
  ```

**Архитектуры репликации**:

**Master-Slave (Primary-Replica)**:
- Один мастер принимает все записи
- Реплики используются для чтения
- Простая архитектура, но мастер - единая точка отказа

**Master-Master (Multi-Master)**:
- Несколько мастеров могут принимать записи
- Сложнее в настройке и поддержке
- Требует разрешения конфликтов

**Каскадная репликация**:
- Реплики могут иметь свои реплики
- Снижает нагрузку на мастер
- Увеличивает задержку для нижних уровней

### 4.2 Шардирование

**Шардирование** - горизонтальное разделение данных между несколькими физическими базами данных (шардами). Каждый шард содержит подмножество данных.

**Преимущества шардирования**:
- Масштабирование для больших объемов данных
- Распределение нагрузки между серверами
- Повышение производительности параллельных операций
- Изоляция отказов

**Недостатки шардирования**:
- Сложность в реализации и поддержке
- Сложные запросы между шардами
- Сложность в изменении схемы шардирования
- Дополнительная логика в приложении

**Стратегии шардирования**:

**Шардирование по диапазону**:
- Разделение по диапазонам значений ключа (например, ID от 1 до 1M, от 1M до 2M и т.д.)
- Простая логика маршрутизации
- Риск неравномерного распределения данных
- Пример:
  ```
  Шард 1: user_id от 1 до 1,000,000
  Шард 2: user_id от 1,000,001 до 2,000,000
  ```

**Шардирование по хешу**:
- Распределение на основе хеш-функции от ключа
- Более равномерное распределение
- Сложнее добавлять новые шарды
- Пример:
  ```
  Шард = hash(user_id) % number_of_shards
  ```

**Шардирование по справочнику**:
- Использование таблицы маршрутизации для определения шарда
- Гибкое распределение
- Требует дополнительного хранилища и обслуживания
- Пример:
  ```
  lookup_table = {
    "user123": "shard2",
    "user456": "shard1"
  }
  ```

**Вертикальное шардирование** (фактически, не шардирование, а разделение таблиц):
- Разделение таблиц по разным серверам
- Проще в реализации
- Ограниченная масштабируемость
- Пример:
  ```
  Server 1: users, profiles
  Server 2: orders, payments
  ```

### 4.3 Согласованность данных в распределенных системах

**CAP-теорема** утверждает, что распределенная система не может одновременно обеспечить все три свойства:
- **Согласованность (Consistency)**: Все узлы видят одинаковые данные в один момент времени
- **Доступность (Availability)**: Каждый запрос получает ответ (успешный или неуспешный)
- **Устойчивость к разделению (Partition tolerance)**: Система продолжает работать при сетевых разделениях

**Модели согласованности**:

**Строгая согласованность**:
- Все изменения мгновенно видны всем узлам
- Высокая стоимость в распределенных системах
- Пример: синхронная репликация

**Последовательная согласованность**:
- Все узлы видят операции в одинаковом порядке
- Менее строгая, чем строгая согласованность
- Пример: распределенные транзакции

**Причинная согласованность**:
- Операции, связанные причинно-следственной связью, видны в правильном порядке
- Несвязанные операции могут быть видны в разном порядке
- Пример: системы с векторными часами

**Eventual consistency (Согласованность в конечном счете)**:
- Все реплики в конечном итоге сходятся к одному состоянию
- Временно могут быть несогласованными
- Пример: DNS, асинхронная репликация

## 5. Оптимизация запросов

### 5.1 Основы оптимизации

**Принципы оптимизации запросов**:
- Используйте индексы эффективно
- Минимизируйте объем возвращаемых данных
- Избегайте полного сканирования таблиц
- Оптимизируйте соединения таблиц
- Используйте подходящие типы данных

**Анализ производительности**:
```sql
-- Включение временных меток для анализа
\timing on

-- Анализ плана выполнения
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';

-- Статистика по таблице
SELECT * FROM pg_stat_user_tables WHERE relname = 'users';

-- Статистика по индексам
SELECT * FROM pg_stat_user_indexes WHERE relname = 'users';
```

### 5.2 Оптимизация SELECT запросов

**Выбор только необходимых столбцов**:
```sql
-- Плохо
SELECT * FROM users JOIN orders ON users.id = orders.user_id;

-- Хорошо
SELECT u.id, u.username, o.order_id, o.amount 
FROM users u JOIN orders o ON u.id = o.user_id;
```

**Ограничение количества строк**:
```sql
-- Использование LIMIT
SELECT * FROM logs ORDER BY created_at DESC LIMIT 100;

-- Использование WHERE для фильтрации
SELECT * FROM users WHERE created_at > CURRENT_DATE - INTERVAL '7 days';
```

**Оптимизация соединений**:
```sql
-- Использование индексов для соединений
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Выбор правильного типа соединения
SELECT u.username, COUNT(o.id) 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.username;
```

**Использование подзапросов и CTE**:
```sql
-- Common Table Expression (CTE)
WITH active_users AS (
    SELECT id, username FROM users WHERE status = 'active'
)
SELECT au.username, COUNT(o.id) as order_count
FROM active_users au
LEFT JOIN orders o ON au.id = o.user_id
GROUP BY au.username;
```

### 5.3 Оптимизация сложных запросов

**Оптимизация агрегаций**:
```sql
-- Использование индексов для GROUP BY
CREATE INDEX idx_orders_user_id_created ON orders(user_id, created_at);

-- Предварительная фильтрация перед агрегацией
SELECT user_id, SUM(amount)
FROM orders
WHERE created_at > CURRENT_DATE - INTERVAL '30 days'
GROUP BY user_id;
```

**Оптимизация сортировки**:
```sql
-- Использование индексов для ORDER BY
CREATE INDEX idx_users_created_at ON users(created_at DESC);

-- Ограничение сортировки небольшим набором
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
```

**Оптимизация DISTINCT**:
```sql
-- Использование GROUP BY вместо DISTINCT
SELECT user_id FROM orders GROUP BY user_id;

-- Использование EXISTS вместо DISTINCT
SELECT u.* FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

**Материализованные представления**:
```sql
-- Создание материализованного представления
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    DATE_TRUNC('month', created_at) as month,
    SUM(amount) as total_sales
FROM orders
GROUP BY month;

-- Обновление материализованного представления
REFRESH MATERIALIZED VIEW monthly_sales;
```

### 5.4 Антипаттерны SQL

**N+1 запросы**:
```sql
-- Плохо (псевдокод)
users = SELECT * FROM users LIMIT 10;
for each user in users:
    orders = SELECT * FROM orders WHERE user_id = user.id;

-- Хорошо
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.id IN (SELECT id FROM users LIMIT 10);
```

**Неэффективные функции в WHERE**:
```sql
-- Плохо (не использует индекс)
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';

-- Хорошо (использует функциональный индекс)
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
```

**Избыточные соединения**:
```sql
-- Плохо
SELECT u.username, o.order_id
FROM users u
JOIN profiles p ON u.id = p.user_id
JOIN orders o ON u.id = o.user_id;

-- Хорошо (если профили не нужны)
SELECT u.username, o.order_id
FROM users u
JOIN orders o ON u.id = o.user_id;
```

**Неявные преобразования типов**:
```sql
-- Плохо (преобразование типов)
SELECT * FROM users WHERE id = '42';

-- Хорошо (соответствие типов)
SELECT * FROM users WHERE id = 42;
```

## 6. Практические примеры

### 6.1 Транзакции в реальных сценариях

**Перевод денег между счетами**:
```sql
BEGIN;

-- Проверка достаточности средств
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- Если balance < 100, ROLLBACK

-- Снятие денег с первого счета
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Пополнение второго счета
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Запись в журнал транзакций
INSERT INTO transactions (from_account, to_account, amount, type)
VALUES (1, 2, 100, 'transfer');

COMMIT;
```

**Обработка заказа**:
```sql
BEGIN;

-- Создание заказа
INSERT INTO orders (user_id, total_amount, status)
VALUES (42, 150.00, 'pending')
RETURNING id INTO order_id;

-- Добавление товаров в заказ
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES 
    (order_id, 101, 2, 50.00),
    (order_id, 102, 1, 50.00);

-- Обновление остатков на складе
UPDATE inventory SET stock = stock - 2 WHERE product_id = 101;
UPDATE inventory SET stock = stock - 1 WHERE product_id = 102;

-- Проверка наличия товаров
SELECT stock FROM inventory 
WHERE product_id IN (101, 102) AND stock < 0;
-- Если есть отрицательные остатки, ROLLBACK

COMMIT;
```

### 6.2 Оптимизация реальных запросов

**Анализ продаж по категориям**:
```sql
-- Неоптимизированный запрос
SELECT 
    c.name as category,
    SUM(oi.quantity * oi.price) as total_sales
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
JOIN categories c ON p.category_id = c.id
WHERE o.created_at BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY c.name
ORDER BY total_sales DESC;

-- Оптимизированный запрос с индексами
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_products_category_id ON products(category_id);

-- Оптимизированный запрос с CTE
WITH order_data AS (
    SELECT 
        oi.product_id,
        oi.quantity * oi.price as sale_amount
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    WHERE o.created_at BETWEEN '2023-01-01' AND '2023-12-31'
)
SELECT 
    c.name as category,
    SUM(od.sale_amount) as total_sales
FROM order_data od
JOIN products p ON od.product_id = p.id
JOIN categories c ON p.category_id = c.id
GROUP BY c.name
ORDER BY total_sales DESC;
```

**Поиск активных пользователей с фильтрацией**:
```sql
-- Неоптимизированный запрос
SELECT 
    u.id, 
    u.username, 
    u.email,
    COUNT(o.id) as order_count,
    SUM(o.total_amount) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE 
    u.status = 'active' AND
    u.created_at < '2023-01-01' AND
    (o.created_at IS NULL OR o.created_at > '2023-01-01')
GROUP BY u.id, u.username, u.email
HAVING COUNT(o.id) > 0
ORDER BY total_spent DESC
LIMIT 100;

-- Оптимизированный запрос с индексами
CREATE INDEX idx_users_status_created ON users(status, created_at);
CREATE INDEX idx_orders_user_id_created ON orders(user_id, created_at);

-- Оптимизированный запрос с CTE
WITH active_users AS (
    SELECT id, username, email
    FROM users
    WHERE status = 'active' AND created_at < '2023-01-01'
),
user_orders AS (
    SELECT 
        user_id,
        COUNT(id) as order_count,
        SUM(total_amount) as total_spent
    FROM orders
    WHERE created_at > '2023-01-01'
    GROUP BY user_id
    HAVING COUNT(id) > 0
)
SELECT 
    au.id, 
    au.username, 
    au.email,
    uo.order_count,
    uo.total_spent
FROM active_users au
JOIN user_orders uo ON au.id = uo.user_id
ORDER BY uo.total_spent DESC
LIMIT 100;
```

### 6.3 Репликация и шардирование на практике

**Настройка репликации в PostgreSQL**:

На мастере:
```sql
-- postgresql.conf
wal_level = replica
max_wal_senders = 10
wal_keep_segments = 64

-- pg_hba.conf
host replication replicator 192.168.1.0/24 md5

-- Создание пользователя для репликации
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'password';

-- Создание слота репликации
SELECT pg_create_physical_replication_slot('replica1_slot');
```

На реплике:
```
-- recovery.conf
standby_mode = 'on'
primary_conninfo = 'host=192.168.1.100 port=5432 user=replicator password=password'
primary_slot_name = 'replica1_slot'
```

**Пример шардирования в приложении (псевдокод)**:
```go
// Функция определения шарда по user_id
func getShardForUser(userID int) string {
    shardNumber := userID % 4  // 4 шарда
    return fmt.Sprintf("shard_%d", shardNumber)
}

// Получение соединения с нужным шардом
func getConnectionForUser(userID int) *sql.DB {
    shardName := getShardForUser(userID)
    return shardConnections[shardName]
}

// Выполнение запроса на нужном шарде
func getUserData(userID int) (User, error) {
    db := getConnectionForUser(userID)
    var user User
    err := db.QueryRow("SELECT * FROM users WHERE id = $1", userID).Scan(&user.ID, &user.Name, &user.Email)
    return user, err
}

// Запрос данных со всех шардов
func getAllActiveUsers() ([]User, error) {
    var allUsers []User
    
    // Запрос к каждому шарду
    for _, db := range shardConnections {
        var users []User
        rows, err := db.Query("SELECT * FROM users WHERE status = 'active'")
        if err != nil {
            return nil, err
        }
        
        // Обработка результатов
        for rows.Next() {
            var user User
            err := rows.Scan(&user.ID, &user.Name, &user.Email)
            if err != nil {
                return nil, err
            }
            users = append(users, user)
        }
        
        allUsers = append(allUsers, users...)
    }
    
    return allUsers, nil
}
```

## 7. Заключение

В этом документе мы рассмотрели ключевые концепции баз данных и SQL, которые часто проверяются на технических интервью:

1. **Реляционные базы данных и SQL**: основы работы с таблицами, запросами, соединениями и подзапросами
2. **Транзакции**: ACID свойства, уровни изоляции, блокировки и проблемы параллельного выполнения
3. **Индексы**: типы индексов, стратегии индексирования и анализ использования
4. **Репликация и шардирование**: типы репликации, стратегии шардирования и согласованность данных
5. **Оптимизация запросов**: принципы оптимизации, анализ производительности и антипаттерны
6. **Практические примеры**: реальные сценарии использования транзакций, оптимизации запросов и распределенных систем

Эти знания помогут вам успешно пройти техническое интервью в компаниях уровня Яндекс и Озон, особенно в части, касающейся баз данных и SQL.
