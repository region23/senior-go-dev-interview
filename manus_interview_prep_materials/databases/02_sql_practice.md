# Практические задачи по SQL и базам данных

В этом документе представлены практические задачи по SQL и базам данных, которые помогут вам подготовиться к соответствующей части технического интервью в Яндексе и Озоне. Каждая задача включает описание, SQL-запросы и пояснения.

## Схема базы данных для задач

Для решения задач будем использовать следующую схему базы данных интернет-магазина:

```sql
-- Пользователи
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    last_login TIMESTAMP
);

-- Категории товаров
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_id INTEGER REFERENCES categories(id),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Товары
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    category_id INTEGER NOT NULL REFERENCES categories(id),
    stock INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);

-- Заказы
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Товары в заказах
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(id),
    product_id INTEGER NOT NULL REFERENCES products(id),
    quantity INTEGER NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    UNIQUE (order_id, product_id)
);

-- Отзывы на товары
CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products(id),
    user_id INTEGER NOT NULL REFERENCES users(id),
    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (product_id, user_id)
);
```

## Задача 1: Базовые запросы SELECT

### 1.1 Выборка данных с условиями

**Задача**: Найти всех активных пользователей, зарегистрированных в 2023 году, отсортированных по дате регистрации.

```sql
SELECT id, username, email, created_at
FROM users
WHERE 
    status = 'active' AND 
    created_at BETWEEN '2023-01-01' AND '2023-12-31'
ORDER BY created_at;
```

**Задача**: Найти все товары в категории "Электроника" с ценой от 1000 до 5000, отсортированные по цене по убыванию.

```sql
SELECT p.id, p.name, p.price, p.stock
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE 
    c.name = 'Электроника' AND 
    p.price BETWEEN 1000 AND 5000 AND
    p.is_active = TRUE
ORDER BY p.price DESC;
```

### 1.2 Агрегация данных

**Задача**: Посчитать количество заказов и общую сумму продаж по месяцам за 2023 год.

```sql
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_sales
FROM orders
WHERE created_at BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY month
ORDER BY month;
```

**Задача**: Найти среднюю оценку и количество отзывов для каждого товара, у которого более 5 отзывов.

```sql
SELECT 
    p.id,
    p.name,
    COUNT(r.id) AS review_count,
    ROUND(AVG(r.rating), 2) AS avg_rating
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name
HAVING COUNT(r.id) > 5
ORDER BY avg_rating DESC, review_count DESC;
```

## Задача 2: Сложные JOIN-запросы

### 2.1 Многотабличные соединения

**Задача**: Получить список товаров с их категориями, количеством продаж и средней оценкой.

```sql
SELECT 
    p.id,
    p.name,
    c.name AS category,
    COALESCE(SUM(oi.quantity), 0) AS total_sold,
    COALESCE(ROUND(AVG(r.rating), 2), 0) AS avg_rating
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN reviews r ON p.id = r.product_id
WHERE p.is_active = TRUE
GROUP BY p.id, p.name, c.name
ORDER BY total_sold DESC, avg_rating DESC;
```

**Задача**: Найти пользователей, которые сделали заказы, но не оставили ни одного отзыва.

```sql
SELECT DISTINCT
    u.id,
    u.username,
    u.email
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.id NOT IN (
    SELECT DISTINCT user_id FROM reviews
)
ORDER BY u.id;
```

### 2.2 Самосоединения (Self Joins)

**Задача**: Получить иерархию категорий (родительская категория и дочерние категории).

```sql
SELECT 
    c1.id,
    c1.name,
    c2.id AS parent_id,
    c2.name AS parent_name
FROM categories c1
LEFT JOIN categories c2 ON c1.parent_id = c2.id
ORDER BY c2.name, c1.name;
```

**Задача**: Найти все пары товаров из одной категории с разницей в цене менее 10%.

```sql
SELECT 
    p1.id AS product1_id,
    p1.name AS product1_name,
    p1.price AS product1_price,
    p2.id AS product2_id,
    p2.name AS product2_name,
    p2.price AS product2_price,
    c.name AS category
FROM products p1
JOIN products p2 ON 
    p1.category_id = p2.category_id AND 
    p1.id < p2.id AND
    ABS(p1.price - p2.price) / p1.price < 0.1
JOIN categories c ON p1.category_id = c.id
WHERE p1.is_active = TRUE AND p2.is_active = TRUE
ORDER BY c.name, p1.id, p2.id;
```

## Задача 3: Подзапросы и CTE

### 3.1 Подзапросы

**Задача**: Найти товары, цена которых выше средней цены в их категории.

```sql
SELECT 
    p.id,
    p.name,
    p.price,
    c.name AS category,
    (
        SELECT AVG(price) 
        FROM products 
        WHERE category_id = p.category_id
    ) AS avg_category_price
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.price > (
    SELECT AVG(price) 
    FROM products 
    WHERE category_id = p.category_id
)
ORDER BY c.name, p.price DESC;
```

**Задача**: Найти пользователей, которые сделали заказы на сумму выше средней суммы заказа.

```sql
SELECT 
    u.id,
    u.username,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username
HAVING SUM(o.total_amount) > (
    SELECT AVG(total_amount) FROM orders
)
ORDER BY total_spent DESC;
```

### 3.2 Common Table Expressions (CTE)

**Задача**: Найти топ-5 категорий по объему продаж за последний месяц.

```sql
WITH monthly_sales AS (
    SELECT 
        p.category_id,
        SUM(oi.quantity * oi.price) AS sales_amount
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON oi.product_id = p.id
    WHERE o.created_at >= CURRENT_DATE - INTERVAL '1 month'
    GROUP BY p.category_id
)
SELECT 
    c.id,
    c.name,
    ms.sales_amount
FROM categories c
JOIN monthly_sales ms ON c.id = ms.category_id
ORDER BY ms.sales_amount DESC
LIMIT 5;
```

**Задача**: Найти пользователей, которые покупали товары из всех категорий верхнего уровня.

```sql
WITH top_categories AS (
    SELECT id FROM categories WHERE parent_id IS NULL
),
user_categories AS (
    SELECT 
        o.user_id,
        p.category_id
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON oi.product_id = p.id
    GROUP BY o.user_id, p.category_id
)
SELECT 
    u.id,
    u.username,
    u.email,
    COUNT(DISTINCT uc.category_id) AS category_count
FROM users u
JOIN user_categories uc ON u.id = uc.user_id
GROUP BY u.id, u.username, u.email
HAVING COUNT(DISTINCT uc.category_id) = (SELECT COUNT(*) FROM top_categories);
```

## Задача 4: Оконные функции

### 4.1 Ранжирование

**Задача**: Ранжировать товары по цене в каждой категории.

```sql
SELECT 
    p.id,
    p.name,
    p.price,
    c.name AS category,
    RANK() OVER (PARTITION BY p.category_id ORDER BY p.price DESC) AS price_rank
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.is_active = TRUE
ORDER BY c.name, price_rank;
```

**Задача**: Найти товары, которые входят в топ-3 по продажам в своей категории.

```sql
WITH product_sales AS (
    SELECT 
        p.id,
        p.name,
        p.category_id,
        SUM(oi.quantity) AS total_sold,
        RANK() OVER (PARTITION BY p.category_id ORDER BY SUM(oi.quantity) DESC) AS sales_rank
    FROM products p
    LEFT JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.id, p.name, p.category_id
)
SELECT 
    ps.id,
    ps.name,
    c.name AS category,
    ps.total_sold,
    ps.sales_rank
FROM product_sales ps
JOIN categories c ON ps.category_id = c.id
WHERE ps.sales_rank <= 3
ORDER BY c.name, ps.sales_rank;
```

### 4.2 Агрегация в окне

**Задача**: Рассчитать долю продаж каждого товара в его категории.

```sql
WITH category_sales AS (
    SELECT 
        p.category_id,
        SUM(oi.quantity * oi.price) AS total_category_sales
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.category_id
),
product_sales AS (
    SELECT 
        p.id,
        p.name,
        p.category_id,
        SUM(oi.quantity * oi.price) AS product_sales
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.id, p.name, p.category_id
)
SELECT 
    ps.id,
    ps.name,
    c.name AS category,
    ps.product_sales,
    cs.total_category_sales,
    ROUND((ps.product_sales / cs.total_category_sales) * 100, 2) AS sales_percentage
FROM product_sales ps
JOIN categories c ON ps.category_id = c.id
JOIN category_sales cs ON ps.category_id = cs.category_id
ORDER BY c.name, sales_percentage DESC;
```

**Задача**: Рассчитать скользящую сумму продаж по дням за последний месяц.

```sql
WITH daily_sales AS (
    SELECT 
        DATE(o.created_at) AS sale_date,
        SUM(o.total_amount) AS daily_amount
    FROM orders o
    WHERE o.created_at >= CURRENT_DATE - INTERVAL '1 month'
    GROUP BY DATE(o.created_at)
)
SELECT 
    sale_date,
    daily_amount,
    SUM(daily_amount) OVER (ORDER BY sale_date) AS running_total,
    SUM(daily_amount) OVER (
        ORDER BY sale_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_7day_sum
FROM daily_sales
ORDER BY sale_date;
```

## Задача 5: Транзакции и изменение данных

### 5.1 Транзакции

**Задача**: Создать транзакцию для оформления нового заказа с проверкой наличия товаров.

```sql
BEGIN;

-- Создаем новый заказ
INSERT INTO orders (user_id, total_amount, status)
VALUES (1, 0, 'pending')
RETURNING id INTO @order_id;

-- Добавляем товары в заказ
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES 
    (@order_id, 101, 2, (SELECT price FROM products WHERE id = 101)),
    (@order_id, 102, 1, (SELECT price FROM products WHERE id = 102));

-- Проверяем наличие товаров
SELECT 
    p.id, 
    p.name, 
    p.stock, 
    oi.quantity,
    CASE WHEN p.stock < oi.quantity THEN 'Недостаточно на складе' ELSE 'OK' END AS status
FROM order_items oi
JOIN products p ON oi.product_id = p.id
WHERE oi.order_id = @order_id AND p.stock < oi.quantity;

-- Если есть товары с недостаточным количеством, отменяем транзакцию
-- В реальном коде здесь будет условие
-- IF EXISTS (SELECT 1 FROM ... WHERE p.stock < oi.quantity) THEN ROLLBACK;

-- Обновляем остатки на складе
UPDATE products p
SET stock = p.stock - oi.quantity
FROM order_items oi
WHERE p.id = oi.product_id AND oi.order_id = @order_id;

-- Обновляем сумму заказа
UPDATE orders
SET total_amount = (
    SELECT SUM(quantity * price) 
    FROM order_items 
    WHERE order_id = @order_id
)
WHERE id = @order_id;

COMMIT;
```

**Задача**: Создать транзакцию для обновления цен товаров с учетом истории изменений.

```sql
-- Предположим, у нас есть таблица для хранения истории цен
CREATE TABLE IF NOT EXISTS price_history (
    id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products(id),
    old_price DECIMAL(10, 2) NOT NULL,
    new_price DECIMAL(10, 2) NOT NULL,
    changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    reason VARCHAR(200)
);

BEGIN;

-- Сохраняем текущие цены в историю
INSERT INTO price_history (product_id, old_price, new_price, reason)
SELECT 
    id, 
    price, 
    price * 1.10, -- Увеличиваем цену на 10%
    'Плановое повышение цен'
FROM products
WHERE category_id = 5; -- Для товаров определенной категории

-- Обновляем цены
UPDATE products
SET 
    price = price * 1.10,
    updated_at = CURRENT_TIMESTAMP
WHERE category_id = 5;

COMMIT;
```

### 5.2 Сложные обновления и вставки

**Задача**: Обновить статус заказов на "просрочен" для заказов, которые находятся в статусе "в обработке" более 3 дней.

```sql
UPDATE orders
SET 
    status = 'overdue',
    updated_at = CURRENT_TIMESTAMP
WHERE 
    status = 'processing' AND
    created_at < CURRENT_TIMESTAMP - INTERVAL '3 days';
```

**Задача**: Вставить данные о новых товарах из временной таблицы, обновив существующие товары.

```sql
-- Предположим, у нас есть временная таблица с новыми товарами
CREATE TEMPORARY TABLE temp_products (
    external_id VARCHAR(50),
    name VARCHAR(200),
    description TEXT,
    price DECIMAL(10, 2),
    category_name VARCHAR(100),
    stock INTEGER
);

-- Вставляем новые категории, если они не существуют
INSERT INTO categories (name)
SELECT DISTINCT tp.category_name
FROM temp_products tp
LEFT JOIN categories c ON tp.category_name = c.name
WHERE c.id IS NULL;

-- Обновляем существующие товары
UPDATE products p
SET 
    name = tp.name,
    description = tp.description,
    price = tp.price,
    stock = tp.stock,
    updated_at = CURRENT_TIMESTAMP
FROM temp_products tp
JOIN categories c ON tp.category_name = c.name
WHERE p.external_id = tp.external_id;

-- Вставляем новые товары
INSERT INTO products (external_id, name, description, price, category_id, stock)
SELECT 
    tp.external_id,
    tp.name,
    tp.description,
    tp.price,
    c.id,
    tp.stock
FROM temp_products tp
JOIN categories c ON tp.category_name = c.name
LEFT JOIN products p ON tp.external_id = p.external_id
WHERE p.id IS NULL;
```

## Задача 6: Индексы и оптимизация

### 6.1 Создание индексов

**Задача**: Создать индексы для оптимизации часто используемых запросов.

```sql
-- Индекс для поиска товаров по названию (часто используется в поиске)
CREATE INDEX idx_products_name ON products USING GIN (to_tsvector('russian', name));

-- Индекс для фильтрации по категории и сортировке по цене
CREATE INDEX idx_products_category_price ON products(category_id, price);

-- Индекс для фильтрации по статусу и дате создания заказа
CREATE INDEX idx_orders_status_created ON orders(status, created_at);

-- Индекс для поиска заказов пользователя
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Составной индекс для часто используемого соединения
CREATE INDEX idx_order_items_order_product ON order_items(order_id, product_id);

-- Индекс для поиска отзывов по товару
CREATE INDEX idx_reviews_product_id ON reviews(product_id);
```

### 6.2 Анализ и оптимизация запросов

**Задача**: Проанализировать и оптимизировать медленный запрос.

Исходный запрос:
```sql
-- Медленный запрос для поиска популярных товаров
SELECT 
    p.name,
    c.name AS category,
    COUNT(oi.id) AS order_count,
    SUM(oi.quantity) AS total_quantity,
    AVG(r.rating) AS avg_rating
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN reviews r ON p.id = r.product_id
WHERE p.is_active = TRUE
GROUP BY p.id, p.name, c.name
ORDER BY total_quantity DESC
LIMIT 100;
```

Анализ с EXPLAIN:
```sql
EXPLAIN ANALYZE
SELECT 
    p.name,
    c.name AS category,
    COUNT(oi.id) AS order_count,
    SUM(oi.quantity) AS total_quantity,
    AVG(r.rating) AS avg_rating
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN reviews r ON p.id = r.product_id
WHERE p.is_active = TRUE
GROUP BY p.id, p.name, c.name
ORDER BY total_quantity DESC
LIMIT 100;
```

Оптимизированный запрос:
```sql
-- Оптимизированный запрос с использованием CTE
WITH product_orders AS (
    SELECT 
        product_id,
        COUNT(id) AS order_count,
        SUM(quantity) AS total_quantity
    FROM order_items
    GROUP BY product_id
),
product_ratings AS (
    SELECT 
        product_id,
        AVG(rating) AS avg_rating
    FROM reviews
    GROUP BY product_id
)
SELECT 
    p.name,
    c.name AS category,
    COALESCE(po.order_count, 0) AS order_count,
    COALESCE(po.total_quantity, 0) AS total_quantity,
    COALESCE(pr.avg_rating, 0) AS avg_rating
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN product_orders po ON p.id = po.product_id
LEFT JOIN product_ratings pr ON p.id = pr.product_id
WHERE p.is_active = TRUE
ORDER BY total_quantity DESC NULLS LAST
LIMIT 100;
```

Необходимые индексы:
```sql
CREATE INDEX idx_order_items_product_quantity ON order_items(product_id, quantity);
CREATE INDEX idx_reviews_product_rating ON reviews(product_id, rating);
```

## Задача 7: Представления и хранимые процедуры

### 7.1 Представления (Views)

**Задача**: Создать представление для отчета по продажам товаров.

```sql
CREATE VIEW product_sales_report AS
SELECT 
    p.id AS product_id,
    p.name AS product_name,
    c.name AS category,
    COUNT(DISTINCT o.id) AS order_count,
    SUM(oi.quantity) AS total_quantity,
    SUM(oi.quantity * oi.price) AS total_revenue,
    COALESCE(AVG(r.rating), 0) AS avg_rating,
    COUNT(r.id) AS review_count
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.id
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name, c.name;
```

**Задача**: Создать представление для анализа активности пользователей.

```sql
CREATE VIEW user_activity_report AS
SELECT 
    u.id AS user_id,
    u.username,
    u.email,
    u.created_at AS registration_date,
    u.last_login,
    COUNT(DISTINCT o.id) AS order_count,
    COALESCE(SUM(o.total_amount), 0) AS total_spent,
    COUNT(DISTINCT r.id) AS review_count,
    CASE 
        WHEN COUNT(DISTINCT o.id) = 0 THEN 'Неактивный'
        WHEN COUNT(DISTINCT o.id) <= 3 THEN 'Низкая активность'
        WHEN COUNT(DISTINCT o.id) <= 10 THEN 'Средняя активность'
        ELSE 'Высокая активность'
    END AS activity_level
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LEFT JOIN reviews r ON u.id = r.user_id
GROUP BY u.id, u.username, u.email, u.created_at, u.last_login;
```

### 7.2 Хранимые процедуры и функции

**Задача**: Создать функцию для расчета рекомендаций товаров на основе истории покупок.

```sql
CREATE OR REPLACE FUNCTION get_product_recommendations(user_id_param INTEGER, limit_param INTEGER DEFAULT 5)
RETURNS TABLE (
    product_id INTEGER,
    product_name VARCHAR,
    category VARCHAR,
    relevance_score NUMERIC
) AS $$
BEGIN
    RETURN QUERY
    WITH user_categories AS (
        -- Категории, которые покупал пользователь
        SELECT 
            p.category_id,
            COUNT(*) AS purchase_count
        FROM orders o
        JOIN order_items oi ON o.id = oi.order_id
        JOIN products p ON oi.product_id = p.id
        WHERE o.user_id = user_id_param
        GROUP BY p.category_id
    ),
    user_purchased_products AS (
        -- Товары, которые уже покупал пользователь
        SELECT DISTINCT oi.product_id
        FROM orders o
        JOIN order_items oi ON o.id = oi.order_id
        WHERE o.user_id = user_id_param
    )
    SELECT 
        p.id AS product_id,
        p.name AS product_name,
        c.name AS category,
        (
            -- Релевантность на основе категорий пользователя и рейтинга товара
            COALESCE(uc.purchase_count, 0) * 0.5 +
            COALESCE(AVG(r.rating), 3) * 0.3 +
            COUNT(DISTINCT oi.order_id) * 0.2
        ) AS relevance_score
    FROM products p
    JOIN categories c ON p.category_id = c.id
    LEFT JOIN user_categories uc ON p.category_id = uc.category_id
    LEFT JOIN reviews r ON p.id = r.product_id
    LEFT JOIN order_items oi ON p.id = oi.product_id
    WHERE 
        p.is_active = TRUE AND
        p.id NOT IN (SELECT product_id FROM user_purchased_products)
    GROUP BY p.id, p.name, c.name, uc.purchase_count
    ORDER BY relevance_score DESC
    LIMIT limit_param;
END;
$$ LANGUAGE plpgsql;

-- Использование функции
SELECT * FROM get_product_recommendations(1, 10);
```

**Задача**: Создать процедуру для автоматического обновления статусов заказов.

```sql
CREATE OR REPLACE PROCEDURE update_order_statuses()
AS $$
DECLARE
    updated_count INTEGER;
BEGIN
    -- Обновляем просроченные заказы
    UPDATE orders
    SET 
        status = 'overdue',
        updated_at = CURRENT_TIMESTAMP
    WHERE 
        status = 'processing' AND
        created_at < CURRENT_TIMESTAMP - INTERVAL '3 days';
    
    GET DIAGNOSTICS updated_count = ROW_COUNT;
    RAISE NOTICE 'Updated % overdue orders', updated_count;
    
    -- Автоматически завершаем доставленные заказы
    UPDATE orders
    SET 
        status = 'completed',
        updated_at = CURRENT_TIMESTAMP
    WHERE 
        status = 'delivered' AND
        updated_at < CURRENT_TIMESTAMP - INTERVAL '14 days';
    
    GET DIAGNOSTICS updated_count = ROW_COUNT;
    RAISE NOTICE 'Completed % delivered orders', updated_count;
    
    -- Отмечаем отмененные заказы как архивные
    UPDATE orders
    SET 
        status = 'archived',
        updated_at = CURRENT_TIMESTAMP
    WHERE 
        status = 'cancelled' AND
        updated_at < CURRENT_TIMESTAMP - INTERVAL '30 days';
    
    GET DIAGNOSTICS updated_count = ROW_COUNT;
    RAISE NOTICE 'Archived % cancelled orders', updated_count;
END;
$$ LANGUAGE plpgsql;

-- Вызов процедуры
CALL update_order_statuses();
```

## Задача 8: Работа с транзакциями и блокировками

### 8.1 Управление транзакциями

**Задача**: Реализовать транзакцию для перемещения товаров между складами с проверкой доступности.

```sql
-- Предположим, у нас есть таблица складов и остатков
CREATE TABLE IF NOT EXISTS warehouses (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    location VARCHAR(200)
);

CREATE TABLE IF NOT EXISTS inventory (
    warehouse_id INTEGER NOT NULL REFERENCES warehouses(id),
    product_id INTEGER NOT NULL REFERENCES products(id),
    quantity INTEGER NOT NULL DEFAULT 0,
    PRIMARY KEY (warehouse_id, product_id)
);

-- Транзакция для перемещения товаров
BEGIN;

-- Блокируем строки в таблице inventory для предотвращения гонок
SELECT quantity 
FROM inventory 
WHERE warehouse_id = 1 AND product_id = 101
FOR UPDATE;

SELECT quantity 
FROM inventory 
WHERE warehouse_id = 2 AND product_id = 101
FOR UPDATE;

-- Проверяем наличие достаточного количества товара на исходном складе
SELECT 
    CASE 
        WHEN quantity >= 10 THEN TRUE
        ELSE FALSE
    END AS has_enough_stock
INTO @has_enough
FROM inventory
WHERE warehouse_id = 1 AND product_id = 101;

-- Если товара достаточно, выполняем перемещение
IF @has_enough THEN
    -- Уменьшаем количество на исходном складе
    UPDATE inventory
    SET quantity = quantity - 10
    WHERE warehouse_id = 1 AND product_id = 101;
    
    -- Увеличиваем количество на целевом складе
    INSERT INTO inventory (warehouse_id, product_id, quantity)
    VALUES (2, 101, 10)
    ON CONFLICT (warehouse_id, product_id)
    DO UPDATE SET quantity = inventory.quantity + 10;
    
    -- Записываем операцию в журнал
    INSERT INTO inventory_movements (
        from_warehouse_id, 
        to_warehouse_id, 
        product_id, 
        quantity, 
        moved_at
    )
    VALUES (1, 2, 101, 10, CURRENT_TIMESTAMP);
    
    COMMIT;
ELSE
    -- Если товара недостаточно, отменяем транзакцию
    ROLLBACK;
END IF;
```

### 8.2 Обработка конкурентных транзакций

**Задача**: Реализовать безопасное обновление остатков товаров при оформлении заказа.

```sql
-- Функция для безопасного обновления остатков
CREATE OR REPLACE FUNCTION process_order_items(order_id_param INTEGER)
RETURNS BOOLEAN AS $$
DECLARE
    item RECORD;
    current_stock INTEGER;
    success BOOLEAN := TRUE;
BEGIN
    -- Перебираем все товары в заказе
    FOR item IN 
        SELECT product_id, quantity 
        FROM order_items 
        WHERE order_id = order_id_param
    LOOP
        -- Блокируем строку товара для обновления
        SELECT stock INTO current_stock
        FROM products
        WHERE id = item.product_id
        FOR UPDATE;
        
        -- Проверяем наличие достаточного количества
        IF current_stock >= item.quantity THEN
            -- Обновляем остаток
            UPDATE products
            SET stock = stock - item.quantity
            WHERE id = item.product_id;
        ELSE
            -- Если товара недостаточно, отмечаем неудачу
            success := FALSE;
            EXIT; -- Выходим из цикла
        END IF;
    END LOOP;
    
    -- Обновляем статус заказа в зависимости от результата
    IF success THEN
        UPDATE orders
        SET status = 'processing', updated_at = CURRENT_TIMESTAMP
        WHERE id = order_id_param;
    ELSE
        UPDATE orders
        SET status = 'failed', updated_at = CURRENT_TIMESTAMP
        WHERE id = order_id_param;
    END IF;
    
    RETURN success;
END;
$$ LANGUAGE plpgsql;

-- Использование функции в транзакции
BEGIN;
SELECT process_order_items(12345);
COMMIT;
```

## Задача 9: Работа с JSON и массивами

### 9.1 Запросы с JSON

**Задача**: Работа с JSON-полями для хранения дополнительных свойств товаров.

```sql
-- Добавим JSON-поле в таблицу товаров
ALTER TABLE products ADD COLUMN properties JSONB;

-- Обновление JSON-свойств товара
UPDATE products
SET properties = jsonb_build_object(
    'color', 'red',
    'size', 'M',
    'material', 'cotton',
    'dimensions', jsonb_build_object(
        'width', 50,
        'height', 70,
        'depth', 10
    )
)
WHERE id = 101;

-- Поиск товаров по JSON-свойствам
SELECT id, name, properties->>'color' AS color
FROM products
WHERE 
    properties->>'color' = 'red' AND
    (properties->>'size' = 'M' OR properties->>'size' = 'L');

-- Поиск товаров с определенными размерами
SELECT id, name, 
    properties->'dimensions'->>'width' AS width,
    properties->'dimensions'->>'height' AS height
FROM products
WHERE 
    (properties->'dimensions'->>'width')::INTEGER > 40 AND
    (properties->'dimensions'->>'height')::INTEGER > 60;

-- Индекс для эффективного поиска по JSON
CREATE INDEX idx_products_properties ON products USING GIN (properties);
```

### 9.2 Запросы с массивами

**Задача**: Работа с массивами для хранения тегов товаров.

```sql
-- Добавим поле для массива тегов
ALTER TABLE products ADD COLUMN tags TEXT[];

-- Обновление тегов товара
UPDATE products
SET tags = ARRAY['summer', 'discount', 'new_collection']
WHERE id = 101;

-- Поиск товаров по тегам (содержит все указанные теги)
SELECT id, name, tags
FROM products
WHERE tags @> ARRAY['summer', 'discount'];

-- Поиск товаров по тегам (содержит любой из указанных тегов)
SELECT id, name, tags
FROM products
WHERE tags && ARRAY['summer', 'winter'];

-- Поиск товаров с определенным количеством тегов
SELECT id, name, tags, array_length(tags, 1) AS tag_count
FROM products
WHERE array_length(tags, 1) > 2;

-- Индекс для эффективного поиска по массиву
CREATE INDEX idx_products_tags ON products USING GIN (tags);
```

## Задача 10: Сложные аналитические запросы

### 10.1 Когортный анализ

**Задача**: Провести когортный анализ пользователей по месяцам регистрации.

```sql
WITH user_cohorts AS (
    -- Определяем когорту для каждого пользователя (месяц регистрации)
    SELECT 
        id,
        DATE_TRUNC('month', created_at) AS cohort_month
    FROM users
),
user_activities AS (
    -- Определяем активность пользователей по месяцам
    SELECT 
        u.id AS user_id,
        uc.cohort_month,
        DATE_TRUNC('month', o.created_at) AS activity_month,
        COUNT(DISTINCT o.id) AS order_count,
        SUM(o.total_amount) AS total_spent
    FROM users u
    JOIN user_cohorts uc ON u.id = uc.id
    JOIN orders o ON u.id = o.user_id
    GROUP BY u.id, uc.cohort_month, DATE_TRUNC('month', o.created_at)
),
cohort_size AS (
    -- Определяем размер каждой когорты
    SELECT 
        cohort_month,
        COUNT(id) AS user_count
    FROM user_cohorts
    GROUP BY cohort_month
)
SELECT 
    ua.cohort_month,
    cs.user_count,
    ua.activity_month,
    EXTRACT(MONTH FROM AGE(ua.activity_month, ua.cohort_month)) AS month_number,
    COUNT(DISTINCT ua.user_id) AS active_users,
    ROUND(COUNT(DISTINCT ua.user_id)::NUMERIC / cs.user_count * 100, 2) AS retention_rate,
    SUM(ua.order_count) AS total_orders,
    SUM(ua.total_spent) AS total_revenue,
    ROUND(SUM(ua.total_spent) / COUNT(DISTINCT ua.user_id), 2) AS arpu
FROM user_activities ua
JOIN cohort_size cs ON ua.cohort_month = cs.cohort_month
GROUP BY ua.cohort_month, cs.user_count, ua.activity_month, month_number
ORDER BY ua.cohort_month, month_number;
```

### 10.2 RFM-анализ

**Задача**: Провести RFM-анализ клиентов (Recency, Frequency, Monetary).

```sql
WITH user_rfm AS (
    SELECT 
        u.id AS user_id,
        u.username,
        -- Recency: дни с последнего заказа
        EXTRACT(DAY FROM CURRENT_TIMESTAMP - MAX(o.created_at)) AS recency,
        -- Frequency: количество заказов
        COUNT(o.id) AS frequency,
        -- Monetary: общая сумма заказов
        COALESCE(SUM(o.total_amount), 0) AS monetary
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id, u.username
),
rfm_scores AS (
    SELECT 
        user_id,
        username,
        recency,
        frequency,
        monetary,
        -- Разбиваем на квинтили (1-5)
        NTILE(5) OVER (ORDER BY recency DESC) AS r_score,
        NTILE(5) OVER (ORDER BY frequency) AS f_score,
        NTILE(5) OVER (ORDER BY monetary) AS m_score
    FROM user_rfm
)
SELECT 
    user_id,
    username,
    recency,
    frequency,
    monetary,
    r_score,
    f_score,
    m_score,
    -- Объединяем оценки в одно число
    r_score * 100 + f_score * 10 + m_score AS rfm_score,
    -- Определяем сегмент клиента
    CASE 
        WHEN r_score >= 4 AND f_score >= 4 AND m_score >= 4 THEN 'VIP'
        WHEN r_score >= 3 AND f_score >= 3 AND m_score >= 3 THEN 'Loyal'
        WHEN r_score >= 3 AND f_score >= 1 AND m_score >= 2 THEN 'Potential Loyal'
        WHEN r_score >= 4 AND f_score <= 2 AND m_score <= 2 THEN 'New'
        WHEN r_score <= 2 AND f_score >= 3 AND m_score >= 3 THEN 'At Risk'
        WHEN r_score <= 2 AND f_score <= 2 AND m_score <= 2 THEN 'Hibernating'
        ELSE 'Regular'
    END AS customer_segment
FROM rfm_scores
ORDER BY rfm_score DESC;
```

## Заключение

В этом документе мы рассмотрели 10 практических задач по SQL и базам данных, которые часто встречаются на технических интервью в компаниях уровня Яндекс и Озон. Каждая задача включает описание, SQL-запросы и пояснения.

Основные темы, которые были охвачены:
1. Базовые запросы SELECT с условиями и агрегацией
2. Сложные JOIN-запросы и самосоединения
3. Подзапросы и Common Table Expressions (CTE)
4. Оконные функции для ранжирования и агрегации
5. Транзакции и изменение данных
6. Индексы и оптимизация запросов
7. Представления и хранимые процедуры
8. Работа с транзакциями и блокировками
9. Работа с JSON и массивами
10. Сложные аналитические запросы (когортный анализ, RFM-анализ)

Эти задачи помогут вам подготовиться к техническому интервью и улучшить ваши навыки работы с SQL и базами данных.
