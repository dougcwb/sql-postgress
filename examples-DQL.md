## Summary
1. [WHERE](#1-where-clause)
2. [Aggregate Functions](#2-aggregate-functions)
3. [JOINS](#3-joins)
    - [INNER JOIN](#31-inner-join-or-just-join)
    - [LEFT JOIN](#32-left-join)
    - [RIGHT JOIN](#33-right-join)
    - [FULL JOIN](#34-full-join)
    - [CROSS JOIN](#35-cross-join)

4. [Windows Functions](#4-windows-functions)
    - [Comparison GROUP BY vs Windows Functions](#41-comparison-group-by-vs-windows-functions)
    - [RANK(), DENSE_RANK() and ROW_NUMBER()](#42-rank-dense_rank-and-row_number)
    - [PERCENT_RANK() and CUME_DIST()](#43-percent_rank-and-cume_dist)
    - [NTILE](#44-ntile)
    - [LAG(), LEAD()](#45-lag-lead)


### 1. WHERE
```sql
SELECT * FROM customers WHERE country='Mexico';


-- ILIKE: Ignores Upper or Lower cases (Postgres).
SELECT * FROM customers WHERE contact_name ILIKE 'c%';


-- Similar to: Search for a list of characters
SELECT * FROM customers WHERE city SIMILAR TO '(B|S|P)%';


-- IN: List of values
SELECT * FROM customers WHERE country IN ('Germany', 'France', 'UK');


-- BETWEEN: Get the rows that are inside two values
SELECT * FROM products WHERE unit_price BETWEEN 10 AND 20;

SELECT * FROM orders 
WHERE order_date BETWEEN '07/04/1996' AND '07/09/1996';
```

### 2. Aggregate Functions:
```sql
-- MIN()
SELECT MIN(unit_price) AS min_price
FROM products;

-- MAX()
SELECT MAX(unit_price) AS max_price
FROM products;

-- TOP()
SELECT product_name, MAX(unit_price) AS max_price
FROM products
GROUP BY product_name
ORDER BY max_price DESC
LIMIT 1;

-- COUNT()
SELECT COUNT(*) AS total
FROM products;

-- AVG()
SELECT AVG(unit_price) AS average_price
FROM products;

-- SUM()
SELECT SUM(quantity) AS order_details_sum_total
FROM order_details;

-- Using GROUP BY
SELECT category_id, MIN(unit_price) AS min_price
FROM products
GROUP BY category_id;

-- HAVING
SELECT o.product_id, p.product_name, SUM(o.quantity) AS total
FROM order_details o
JOIN products p ON p.product_id = o.product_id
GROUP BY o.product_id, p.product_name
HAVING SUM(o.quantity) < 200
ORDER BY total DESC;
```

### 3. JOINS:

![Joins](/images/joins.jpg)

#### 3.1 INNER JOIN (or just JOIN)
Retrieves only the related rows in both tables
```sql
-- Orders from year 1996
SELECT *
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
WHERE EXTRACT(YEAR FROM o.order_date) = 1996;
```

#### 3.2 LEFT JOIN
Retrieves the first table and the related rows from second table
```sql
SELECT e.city AS cidade, 
       COUNT(DISTINCT e.employee_id) AS emp_number, 
       COUNT(DISTINCT c.customer_id) AS cust_number
FROM employees e 
LEFT JOIN customers c ON e.city = c.city
GROUP BY e.city
ORDER BY cidade;
```

#### 3.3 RIGHT JOIN
Retrieves the second table and the related rows from first table
```sql
SELECT c.city AS cidade, 
       COUNT(DISTINCT c.customer_id) AS cust_number, 
       COUNT(DISTINCT e.employee_id) AS emp_number
FROM employees e 
RIGHT JOIN customers c ON e.city = c.city
GROUP BY c.city
ORDER BY cidade;
```

#### 3.4 FULL JOIN
Retrieves all rows from all tables
```sql
SELECT
	COALESCE(e.city, c.city) AS cidade,
	COUNT(DISTINCT e.employee_id) AS emp_number,
	COUNT(DISTINCT c.customer_id) AS cust_number
FROM employees e 
FULL JOIN customers c ON e.city = c.city
GROUP BY e.city, c.city
ORDER BY cidade;
```

#### 3.5 CROSS JOIN
Retrieves all possible combinations between two or more tables without any specific condition for the join
```sql
SELECT c.customer_id, c.company_name, o.*
FROM Customers c 
CROSS JOIN Orders o
```

> [!CAUTION]  
> Be careful! this will get a lot of rows and may crash the database or the client.

### 4. Windows Functions

#### 4.1 Comparison GROUP BY vs Windows Functions
```sql
SELECT customer_id,
   MIN(freight) AS min_freight,
   MAX(freight) AS max_freight,
   AVG(freight) AS avg_freight
FROM orders
GROUP BY customer_id
ORDER BY customer_id;

SELECT DISTINCT customer_id,
   MIN(freight) OVER (PARTITION BY customer_id) AS min_freight,
   MAX(freight) OVER (PARTITION BY customer_id) AS max_freight,
   AVG(freight) OVER (PARTITION BY customer_id) AS avg_freight
FROM orders
ORDER BY customer_id;

SELECT 
    customer_id,
    order_id,  -- for each order
    freight,
    MIN(freight) OVER (PARTITION BY customer_id) AS min_freight,
    MAX(freight) OVER (PARTITION BY customer_id) AS max_freight,
    AVG(freight) OVER (PARTITION BY customer_id) AS avg_freight
FROM orders
ORDER BY customer_id, order_id;
```

#### 4.2 RANK(), DENSE_RANK() and ROW_NUMBER()
```sql
SELECT  
  o.order_id, 
  p.product_name, 
  (o.unit_price * o.quantity) AS total_sale,
  ROW_NUMBER() OVER (ORDER BY (o.unit_price * o.quantity) DESC) AS order_rn, 
  RANK() OVER (ORDER BY (o.unit_price * o.quantity) DESC) AS order_rank, 
  DENSE_RANK() OVER (ORDER BY (o.unit_price * o.quantity) DESC) AS order_dense
FROM  
  order_details o
JOIN 
  products p ON p.product_id = o.product_id;
```

#### 4.3 PERCENT_RANK() and CUME_DIST()
```sql
SELECT  
  order_id, 
  unit_price * quantity AS total_sale,
  ROUND(CAST(PERCENT_RANK() OVER (PARTITION BY order_id 
    ORDER BY (unit_price * quantity) DESC) AS numeric), 2) AS order_percent_rank,
  ROUND(CAST(CUME_DIST() OVER (PARTITION BY order_id 
    ORDER BY (unit_price * quantity) DESC) AS numeric), 2) AS order_cume_dist
FROM  
  order_details;
```

#### 4.4 NTILE
```sql
SELECT first_name, last_name, title,
   NTILE(3) OVER (ORDER BY first_name) AS group_number
FROM employees;
```

#### 4.5 LAG(), LEAD()
```sql
SELECT 
  customer_id, 
  TO_CHAR(order_date, 'YYYY-MM-DD') AS order_date, 
  shippers.company_name AS shipper_name, 
  LAG(freight) OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS previous_order_freight, 
  freight AS order_freight, 
  LEAD(freight) OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS next_order_freight
FROM 
  orders
JOIN 
  shippers ON shippers.shipper_id = orders.ship_via;
```