# sql_learning

--Задания разбиты по темам: от простого SELECT до оконных функций.

--Ниже инструкция как развернуть Postgresql, создать и заполнить таблицы

-- =============================================


--Блок 1. Базовый SELECT, WHERE, ORDER BY (1–10)

--1. Вывести всех клиентов с их именами и городами.

--sql
SELECT first_name, last_name, city FROM customers;

--2. Вывести названия и цены всех товаров дороже 20 000.

--sql
SELECT name, price FROM products WHERE price > 20000;

--3. Вывести товары с ценой от 1 000 до 5 000, отсортированные по цене возрастанию.

--sql
SELECT name, price FROM products WHERE price BETWEEN 1000 AND 5000 ORDER BY price;

--4. Найти клиентов из Москвы.

--sql
SELECT * FROM customers WHERE city = 'Москва';

--5. Вывести клиентов из России или Германии.

--sql
SELECT * FROM customers WHERE country IN ('Россия', 'Germany');

--6. Найти всех клиентов, чей email заканчивается на mail.com.

--sql
SELECT * FROM customers WHERE email LIKE '%mail.com';

--7. Вывести товары, которых на складе меньше 10 штук.

--sql
SELECT name, stock FROM products WHERE stock < 10;

--8. Вывести клиентов, отсортированных по дате регистрации (сначала новые).

--sql
SELECT * FROM customers ORDER BY signup_date DESC;

--9. Вывести 5 самых дорогих товаров.

--sql
SELECT name, price FROM products ORDER BY price DESC LIMIT 5;

--10. Найти клиентов, родившихся в 1990-х (1990-01-01 — 1999-12-31).

--sql
SELECT first_name, last_name, birth_date FROM customers
WHERE birth_date BETWEEN '1990-01-01' AND '1999-12-31';

--Блок 2. Агрегация, GROUP BY, HAVING (11–20)

--11. Посчитать общее количество клиентов.

--sql
SELECT COUNT(*) AS total_customers FROM customers;

--12. Посчитать среднюю цену товаров.

--sql
SELECT ROUND(AVG(price), 2) AS avg_price FROM products;

--13. Найти минимальную и максимальную цену товара.

--sql
SELECT MIN(price) AS min_price, MAX(price) AS max_price FROM products;

--14. Посчитать количество клиентов по городам.

--sql
SELECT city, COUNT(*) AS cnt FROM customers GROUP BY city ORDER BY cnt DESC;

--15. Посчитать количество товаров в каждой категории.

--sql
SELECT category_id, COUNT(*) AS cnt FROM products GROUP BY category_id ORDER BY category_id;

--16. Вывести города, в которых больше 1 клиента.

--sql
SELECT city, COUNT(*) FROM customers GROUP BY city HAVING COUNT(*) > 1;

--17. Посчитать суммарную стоимость всех товаров на складе (price * stock).

--sql
SELECT SUM(price * stock) AS total_stock_value FROM products;

--18. Вывести среднюю цену товара по каждой категории, где средняя цена > 20 000.

--sql
SELECT category_id, ROUND(AVG(price), 2) AS avg_price
FROM products
GROUP BY category_id
HAVING AVG(price) > 20000;

--19. Посчитать количество заказов по статусам.

--sql
SELECT status, COUNT(*) FROM orders GROUP BY status;

--20. Найти клиентов, сделавших более 1 заказа.

--sql
SELECT customer_id, COUNT(*) AS orders_cnt
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;

--Блок 3. JOIN (21–32)

--21. Вывести все заказы с именем клиента.

--sql
SELECT o.order_id, o.order_date, c.first_name, c.last_name
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id;

--22. Вывести все позиции заказов с названием товара.

--sql
SELECT oi.order_item_id, oi.order_id, p.name, oi.quantity, oi.unit_price
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id;

--23. Вывести товары с названием категории.

--sql
SELECT p.name AS product, c.name AS category
FROM products p
JOIN categories c ON c.category_id = p.category_id;

--24. Вывести полную информацию по заказу №1 (клиент, товары, количество).

--sql
SELECT o.order_id, c.first_name, c.last_name, p.name, oi.quantity, oi.unit_price
FROM orders o
JOIN customers c   ON c.customer_id = o.customer_id
JOIN order_items oi ON oi.order_id  = o.order_id
JOIN products p    ON p.product_id  = oi.product_id
WHERE o.order_id = 1;

--25. Посчитать сумму заказов по каждому клиенту.

--sql
SELECT c.customer_id, c.first_name, c.last_name, SUM(o.total_amount) AS total
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total DESC NULLS LAST;

--26. Найти клиентов, не сделавших ни одного заказа.

--sql
SELECT c.*
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL;

--27. Вывести все категории и количество товаров в них (включая пустые).

--sql
SELECT c.name, COUNT(p.product_id) AS products_cnt
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id
GROUP BY c.name
ORDER BY products_cnt DESC;

--28. Найти топ-3 клиента по сумме заказов.

--sql
SELECT c.first_name, c.last_name, SUM(o.total_amount) AS total
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total DESC
LIMIT 3;

--29. Вывести товары, которые никогда не заказывали.

--sql
SELECT p.*
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.product_id
WHERE oi.order_item_id IS NULL;

--30. Найти самый продаваемый товар по количеству.

--sql
SELECT p.name, SUM(oi.quantity) AS total_qty
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id
GROUP BY p.name
ORDER BY total_qty DESC
LIMIT 1;

--31. Вывести для каждого заказа количество позиций.

--sql
SELECT o.order_id, COUNT(oi.order_item_id) AS items_cnt
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY o.order_id
ORDER BY o.order_id;

--32. Найти клиентов, купивших товары из категории "Книги".

--sql
SELECT DISTINCT c.first_name, c.last_name
FROM customers c
JOIN orders o       ON o.customer_id = c.customer_id
JOIN order_items oi ON oi.order_id   = o.order_id
JOIN products p     ON p.product_id  = oi.product_id
JOIN categories cat ON cat.category_id = p.category_id
WHERE cat.name = 'Книги';

--Блок 4. Подзапросы (33–40)

--33. Найти товары дороже средней цены.

--sql
SELECT name, price FROM products
WHERE price > (SELECT AVG(price) FROM products);

--34. Найти клиента с максимальной суммой заказов.

--sql
SELECT c.first_name, c.last_name
FROM customers c
WHERE c.customer_id = (
    SELECT customer_id FROM orders
    GROUP BY customer_id
    ORDER BY SUM(total_amount) DESC
    LIMIT 1
);

--35. Вывести клиентов, сделавших заказы в 2024 году.

--sql
SELECT * FROM customers
WHERE customer_id IN (
    SELECT customer_id FROM orders
    WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
);

--36. Вывести товары, цена которых выше средней цены их категории.

--sql
SELECT p.name, p.price, p.category_id
FROM products p
WHERE p.price > (
    SELECT AVG(p2.price) FROM products p2
    WHERE p2.category_id = p.category_id
);

--37. Найти категории, в которых больше 2 товаров.

--sql
SELECT * FROM categories c
WHERE (SELECT COUNT(*) FROM products p WHERE p.category_id = c.category_id) > 2;

--38. Вывести клиентов, потративших больше, чем клиент с id=1.

--sql
SELECT c.first_name, c.last_name,
       (SELECT COALESCE(SUM(total_amount),0) FROM orders o WHERE o.customer_id = c.customer_id) AS total
FROM customers c
WHERE (SELECT COALESCE(SUM(total_amount),0) FROM orders o WHERE o.customer_id = c.customer_id)
    > (SELECT COALESCE(SUM(total_amount),0) FROM orders o WHERE o.customer_id = 1);

--39. Использовать EXISTS: клиенты, у которых есть хотя бы один completed-заказ.

--sql
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id AND o.status = 'completed'
);

--40. Вывести товары, которые дороже любого товара категории 4.

--sql
SELECT * FROM products
WHERE price > ALL (SELECT price FROM products WHERE category_id = 4);

--Блок 5. Оконные функции, CTE, продвинутое (41–50)

--41. Пронумеровать клиентов по дате регистрации.

--sql
SELECT customer_id, first_name, signup_date,
       ROW_NUMBER() OVER (ORDER BY signup_date) AS rn
FROM customers;

--42. Для каждого города пронумеровать клиентов по дате регистрации.

--sql
SELECT city, first_name, signup_date,
       ROW_NUMBER() OVER (PARTITION BY city ORDER BY signup_date) AS rn
FROM customers;

--43. Вывести топ-1 самого дорогого товара в каждой категории.

--sql
SELECT * FROM (
    SELECT p.*, ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rn
    FROM products p
) t WHERE rn = 1;

--44. Посчитать накопительную сумму заказов по дате.

--sql
SELECT order_date, total_amount,
       SUM(total_amount) OVER (ORDER BY order_date) AS running_total
FROM orders
ORDER BY order_date;

--45. Сравнить цену товара со средней ценой его категории.

--sql
SELECT name, price, category_id,
       ROUND(AVG(price) OVER (PARTITION BY category_id), 2) AS avg_cat_price,
       price - AVG(price) OVER (PARTITION BY category_id) AS diff
FROM products;

--46. Через CTE посчитать суммарные продажи каждого товара и вывести топ-5.

--sql
WITH sales AS (
    SELECT product_id, SUM(quantity * unit_price) AS revenue
    FROM order_items
    GROUP BY product_id
)
SELECT p.name, s.revenue
FROM sales s
JOIN products p ON p.product_id = s.product_id
ORDER BY s.revenue DESC
LIMIT 5;

--47. Для каждого клиента вывести дату последнего заказа.

--sql
SELECT c.customer_id, c.first_name,
       MAX(o.order_date) AS last_order
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name;
-- или через оконную функцию:
SELECT DISTINCT customer_id,
       FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS last_order
FROM orders;

--48. Иерархия сотрудников: вывести сотрудника и его руководителя.

--sql
SELECT e.first_name || ' ' || e.last_name AS employee,
       m.first_name || ' ' || m.last_name AS manager
FROM employees e
LEFT JOIN employees m ON m.employee_id = e.manager_id;

--49. Найти клиентов, у которых средний чек выше среднего по всем заказам.

--sql
WITH avg_orders AS (
    SELECT customer_id, AVG(total_amount) AS avg_check
    FROM orders
    GROUP BY customer_id
)
SELECT c.first_name, c.last_name, a.avg_check
FROM avg_orders a
JOIN customers c ON c.customer_id = a.customer_id
WHERE a.avg_check > (SELECT AVG(total_amount) FROM orders);

--50. Категории и суммарная выручка по ним, отсортированная по убыванию; только те, где выручка > 10 000.

--sql
SELECT cat.name AS category,
       SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN products p     ON p.product_id  = oi.product_id
JOIN categories cat ON cat.category_id = p.category_id
GROUP BY cat.name
HAVING SUM(oi.quantity * oi.unit_price) > 10000
ORDER BY revenue DESC;



-- =============================================

Развернуть postgres в docker

docker run -d --name pg_sql_learning -e POSTGRES_USER=student -e POSTGRES_PASSWORD=XXXXXXX -e POSTGRES_DB=sqlcourse -p 5432:5432 -v C:\sql_learning\pgdata:/var/lib/postgresql/data postgres:16

-- =============================================

Создать и заполнить таблицы

CREATE TABLE customers (
    customer_id   SERIAL PRIMARY KEY,
    first_name    VARCHAR(50)  NOT NULL,
    last_name     VARCHAR(50)  NOT NULL,
    email         VARCHAR(100) UNIQUE,
    city          VARCHAR(50),
    country       VARCHAR(50),
    signup_date   DATE         NOT NULL,
    birth_date    DATE
);

CREATE TABLE categories (
    category_id   SERIAL PRIMARY KEY,
    name          VARCHAR(50) NOT NULL,
    parent_id     INT REFERENCES categories(category_id)
);

CREATE TABLE products (
    product_id    SERIAL PRIMARY KEY,
    name          VARCHAR(100) NOT NULL,
    category_id   INT REFERENCES categories(category_id),
    price         NUMERIC(10,2) NOT NULL CHECK (price >= 0),
    stock         INT NOT NULL DEFAULT 0,
    created_at    TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    order_id      SERIAL PRIMARY KEY,
    customer_id   INT REFERENCES customers(customer_id),
    order_date    DATE NOT NULL,
    status        VARCHAR(20) NOT NULL DEFAULT 'new',
    total_amount  NUMERIC(10,2) NOT NULL DEFAULT 0
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id      INT REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id    INT REFERENCES products(product_id),
    quantity      INT NOT NULL CHECK (quantity > 0),
    unit_price    NUMERIC(10,2) NOT NULL
);

CREATE TABLE employees (
    employee_id   SERIAL PRIMARY KEY,
    first_name    VARCHAR(50) NOT NULL,
    last_name     VARCHAR(50) NOT NULL,
    manager_id    INT REFERENCES employees(employee_id),
    position      VARCHAR(50),
    salary        NUMERIC(10,2),
    hire_date     DATE
);

-- =============================================
-- НАПОЛНЕНИЕ ДАННЫМИ
-- =============================================

-- Клиенты
INSERT INTO customers (first_name, last_name, email, city, country, signup_date, birth_date) VALUES
('Иван',    'Петров',    'ivan.petrov@mail.com',    'Москва',         'Россия',  '2022-01-15', '1990-05-12'),
('Мария',   'Сидорова',  'maria.sidorova@mail.com', 'Санкт-Петербург','Россия',  '2022-03-22', '1988-11-30'),
('Алексей', 'Кузнецов',  'alex.kuznetsov@mail.com', 'Москва',         'Россия',  '2021-12-01', '1995-07-19'),
('Ольга',   'Смирнова',  'olga.smirnova@mail.com',  'Казань',         'Россия',  '2023-02-10', '1992-02-25'),
('Дмитрий', 'Волков',    'dmitry.volkov@mail.com',  'Москва',         'Россия',  '2023-05-05', '1985-09-08'),
('Анна',    'Морозова',  'anna.morozova@mail.com',  'Новосибирск',    'Россия',  '2022-08-14', '1998-12-01'),
('Сергей',  'Новиков',   'sergey.novikov@mail.com', 'Екатеринбург',   'Россия',  '2021-06-30', '1980-03-17'),
('Елена',   'Фёдорова',  'elena.fedorova@mail.com', 'Москва',         'Россия',  '2023-09-09', '1993-10-22'),
('John',    'Smith',     'john.smith@mail.com',     'London',         'UK',      '2022-04-18', '1987-06-14'),
('Emily',   'Johnson',   'emily.johnson@mail.com',  'New York',       'USA',     '2023-01-25', '1991-08-09'),
('Michael', 'Brown',     'michael.brown@mail.com',  'Berlin',         'Germany', '2022-11-11', '1984-04-03'),
('Sophia',  'Davis',     'sophia.davis@mail.com',   'Paris',          'France',  '2023-07-19', '1996-01-27');

-- Категории (с иерархией)
INSERT INTO categories (name, parent_id) VALUES
('Электроника',                NULL),  -- 1
('Компьютеры',                 1),     -- 2
('Смартфоны',                  1),     -- 3
('Книги',                      NULL),  -- 4
('Художественная литература',  4),     -- 5
('Учебники',                   4),     -- 6
('Одежда',                     NULL),  -- 7
('Мужская одежда',             7),     -- 8
('Женская одежда',             7);     -- 9

-- Товары
INSERT INTO products (name, category_id, price, stock) VALUES
('Ноутбук Lenovo IdeaPad',              2, 55000.00,  15),
('Ноутбук Apple MacBook Air',           2, 120000.00,  5),
('Смартфон Samsung Galaxy S23',         3, 75000.00,  20),
('Смартфон iPhone 15',                  3, 110000.00, 10),
('Планшет Xiaomi Pad 6',                2, 35000.00,  25),
('Книга "Война и мир"',                 5, 1500.00,  100),
('Книга "Преступление и наказание"',    5, 1200.00,   80),
('Учебник SQL для начинающих',          6, 2500.00,   50),
('Учебник Python с нуля',               6, 2200.00,   60),
('Футболка мужская',                    8, 1500.00,  200),
('Джинсы мужские',                      8, 4500.00,   80),
('Платье женское',                      9, 5500.00,   40),
('Куртка зимняя',                       9, 12000.00,  30),
('Наушники Sony WH-1000XM5',            1, 30000.00,  12),
('Монитор Dell 27"',                    2, 25000.00,  18);

-- Заказы
INSERT INTO orders (customer_id, order_date, status, total_amount) VALUES
(1,  '2024-01-05', 'completed', 56500.00),
(1,  '2024-02-10', 'completed', 30000.00),
(2,  '2024-01-15', 'completed', 120000.00),
(2,  '2024-03-01', 'cancelled', 0.00),
(3,  '2024-02-20', 'completed', 75000.00),
(4,  '2024-03-10', 'shipped',   6000.00),
(5,  '2024-03-15', 'new',       35000.00),
(5,  '2024-04-01', 'completed', 15000.00),
(6,  '2024-04-05', 'completed', 25000.00),
(7,  '2024-04-10', 'new',       12000.00),
(8,  '2024-04-12', 'shipped',   110000.00),
(9,  '2024-01-20', 'completed', 30000.00),
(10, '2024-02-25', 'completed', 5500.00),
(11, '2024-03-20', 'completed', 3000.00),
(1,  '2024-04-15', 'new',       2500.00);

-- Позиции заказов
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(1,  1,  1,  55000.00),
(1,  8,  1,  1500.00),
(2,  14, 1,  30000.00),
(3,  2,  1,  120000.00),
(4,  3,  1,  75000.00),
(5,  3,  1,  75000.00),
(6,  6,  4,  1500.00),
(7,  5,  1,  35000.00),
(8,  10, 10, 1500.00),
(9,  15, 1,  25000.00),
(10, 13, 1,  12000.00),
(11, 4,  1,  110000.00),
(12, 14, 1,  30000.00),
(13, 12, 1,  5500.00),
(14, 7,  1,  1200.00),
(14, 8,  1,  2500.00),
(15, 8,  1,  2500.00);

-- Сотрудники (иерархия: manager_id -> employee_id)
INSERT INTO employees (first_name, last_name, manager_id, position, salary, hire_date) VALUES
('Пётр',    'Иванов',   NULL, 'CEO',                250000.00, '2015-01-10'),
('Наталья', 'Смирнова', 1,    'CTO',                180000.00, '2016-03-15'),
('Артём',   'Орлов',    1,    'CFO',                170000.00, '2016-06-01'),
('Виктор',  'Лебедев',  2,    'Team Lead Backend',  150000.00, '2018-02-20'),
('Ирина',   'Соколова', 2,    'Team Lead Frontend', 150000.00, '2018-05-11'),
('Максим',  'Попов',    4,    'Senior Developer',   130000.00, '2019-09-01'),
('Ксения',  'Козлова',  4,    'Developer',          100000.00, '2020-11-15'),
('Роман',   'Никитин',  5,    'Senior Developer',   130000.00, '2019-07-19'),
('Юлия',    'Егорова',  5,    'Developer',           95000.00, '2021-03-22'),
('Андрей',  'Беляев',   3,    'Accountant',          90000.00, '2017-08-08');

-- =============================================
-- СБРОС ПОСЛЕДОВАТЕЛЬНОСТЕЙ (не обязательно, но полезно)
-- =============================================
SELECT setval('customers_customer_id_seq',   (SELECT MAX(customer_id)   FROM customers));
SELECT setval('categories_category_id_seq',  (SELECT MAX(category_id)   FROM categories));
SELECT setval('products_product_id_seq',     (SELECT MAX(product_id)    FROM products));
SELECT setval('orders_order_id_seq',         (SELECT MAX(order_id)      FROM orders));
SELECT setval('order_items_order_item_id_seq',(SELECT MAX(order_item_id) FROM order_items));
SELECT setval('employees_employee_id_seq',   (SELECT MAX(employee_id)   FROM employees));


