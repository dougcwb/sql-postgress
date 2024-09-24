## Summary
1. [Where clause](#1-where-clause)

### 1. Where clause
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

### Aggregate Functions:
```sql
-- MIN()
SELECT MIN(unit_price) AS min_price
FROM products;

-- MAX()
SELECT MAX(unit_price) AS max_price
FROM products;

-- TOP product price
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
```
![Joins](<joins.jpg>)

### JOINS:
```sql
-- Inner Join: Retrieves only the related rows in both tables
-- Orders from year 1996
SELECT *
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
WHERE EXTRACT(YEAR FROM o.order_date) = 1996;

-- RIGHT JOIN: Retrieves the second table and the related rows from first table
SELECT c.city AS cidade, 
       COUNT(DISTINCT c.customer_id) AS numero_de_clientes, 
       COUNT(DISTINCT e.employee_id) AS numero_de_funcionarios
FROM employees e 
RIGHT JOIN customers c ON e.city = c.city
GROUP BY c.city
ORDER BY cidade;

-- FULL JOIN: Retrieves all rows from all tables
SELECT
	COALESCE(e.city, c.city) AS cidade,
	COUNT(DISTINCT e.employee_id) AS numero_de_funcionarios,
	COUNT(DISTINCT c.customer_id) AS numero_de_clientes
FROM employees e 
FULL JOIN customers c ON e.city = c.city
GROUP BY e.city, c.city
ORDER BY cidade;

-- CROSS JOIN: Retrieves all possible combinations between two or more tables without any specific condition for the join.
SELECT c.customer_id, c.company_name, o.*
FROM Customers c 
CROSS JOIN Orders o
-- Be careful! this will get a lot of rows and can crash the database or the client
```
### HAVING
```sql
-- HAVING
SELECT o.product_id, p.product_name, SUM(o.quantity) AS quantidade_total
FROM order_details o
JOIN products p ON p.product_id = o.product_id
GROUP BY o.product_id, p.product_name
HAVING SUM(o.quantity) < 200
ORDER BY quantidade_total DESC;
```

### Windows Functions

#### Comparison GROUP BY vs Windows Functions
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

#### RANK(), DENSE_RANK() and ROW_NUMBER()
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

#### PERCENT_RANK() and CUME_DIST()
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

#### NTILE
```sql
SELECT first_name, last_name, title,
   NTILE(3) OVER (ORDER BY first_name) AS group_number
FROM employees;
```

#### LAG(), LEAD()
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