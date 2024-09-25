![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

# Postgres SQL Data Analyst

## Summary
1. [Install Postgres and PGAdmin with Docker](#1-install-postgres-and-pgadmin-with-docker)
    - [Postgres Access](#11-postgres-access)
    - [Config Connection](#12-config-connection)
2. [SQL Commands](#2-sql-commands)
    - [DDL – Data Definition Language](#21-ddl--data-definition-language)
    - [DQL – Data Query Language](#22-dql--data-query-language)
    - [DML – Data Manipulation Language](#23-dml--data-manipulation-language)
    - [DCL – Data Control Language](#24-dcl--data-control-language)
    - [TCL – Transaction Control Language](#25-tcl--transaction-control-language)
3. [CTE (Common Table Expressions)](#3-cte-common-table-expressions)
4. [Subqueries](#4-subqueries)
    - [Turning into CTE](#41-turning-into-cte)
5. [Views](#5-views)
6. [Temporary Tables / Staging / Testes ETL](#6-temporary-tables--staging--testes-etl)
7. [Materialized Views / Snapshot](#7-materialized-views--snapshot)

## 1. Install Postgres and PGAdmin with Docker
In VsCode, Open terminal and run:
```bash
docker compose up
```

### 1.1 Postgres Access
- URL: http://localhost:5050  
- password: postgres

### 1.2 Config Connection
- host: db
- database: northwind
- user: postgres  
- PW: postgres  

This is based on [Northwind Github](https://github.com/pthom/northwind_psql)

![northwind](/images/northwind.png)

## 2. SQL Commands

![sql_mindmap](/images/sql_mindmap.jpg)

## 2.1 DDL – Data Definition Language
DDL is a set of SQL commands used to create, modify, and delete database structures but not data. 

Used By: DBA (Database Admin)

|Command|Description|Syntax|
|---|---|---|
CREATE|Create database or its objects (table, index, function, views, store procedure, and triggers)|CREATE TABLE table_name (column1 data_type, column2 data_type, ...);|
|DROP|Delete objects from the database|DROP TABLE table_name;
|ALTER|Alter the structure of the database|ALTER TABLE table_name ADD COLUMN column_name data_type;|
|TRUNCATE|Remove all records from a table, including all spaces allocated for the records are removed|TRUNCATE TABLE table_name;
|COMMENT|Add comments to the data dictionary|COMMENT 'comment_text' ON TABLE table_name;|
|RENAME|Rename an object existing in the database|RENAME TABLE old_table_name TO new_table_name;|

## 2.2 DQL – Data Query Language
Allows getting the data out of the database to perform operations with it. When a SELECT is fired against a table or tables the result is compiled into a further temporary table.

Used By: Aplications, Front-End Web App, Developers, PO, PM, General use

|Command|Description|Syntax|
|---|---|---|
|SELECT|It is used to retrieve data from the database|SELECT column1, column2, ...FROM table_name WHERE condition;|

It looks simple, but its not. Check this file:
[Examples of DQL](examples-DQL.md)

## 2.3 DML – Data Manipulation Language
Manipulation of data present in the database.

Used By: Developers, Data Engineer, Aplications (CRUD)

|Command|Description|Syntax|
|---|---|---|
|INSERT|Insert data into a table|INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...);|
|UPDATE|Update existing data within a table|UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;|
|DELETE|Delete records from a database table|DELETE FROM table_name WHERE condition;|
|LOCK|Table control concurrency|LOCK TABLE table_name IN lock_mode;|
|CALL|Call a PL/SQL or JAVA subprogram|CALL procedure_name(arguments);|
|EXPLAIN PLAN|Describe the access path to data|EXPLAIN PLAN FOR SELECT * FROM table_name;|

## 2.4 DCL – Data Control Language
Mainly deal with the rights, permissions, and other controls of the database system.

Used By: DBA

|Command|Description|Syntax|
|---|---|---|
|GRANT|Assigns new privileges to a user account, allowing access to specific database objects, actions, or functions.|GRANT privilege_type [(column_list)] ON [object_type] object_name TO user [WITH GRANT OPTION];|
|REVOKE|Removes previously granted privileges from a user account, taking away their access to certain database objects or actions.|REVOKE [GRANT OPTION FOR] privilege_type [(column_list)] ON [object_type] object_name FROM user [CASCADE];|

## 2.5 TCL – Transaction Control Language
Transactions group a set of tasks into a single execution unit. Each transaction begins with a specific task and ends when all the tasks in the group are successfully completed. If any of the tasks fail, the transaction fails.

Therefore, a transaction has only two results: success or failure.

Used By: Developers, Aplications, Data Engineers

|Command|Description|Syntax|
|---|---|---|
|BEGIN TRANSACTION|Starts a new transaction|BEGIN TRANSACTION [transaction_name];|
|COMMIT|Saves all changes made during the transaction|COMMIT;|
|ROLLBACK|Undoes all changes made during the transaction|ROLLBACK;|
|SAVEPOINT|Creates a savepoint within the current transaction|SAVEPOINT savepoint_name;|

## 3. CTE (Common Table Expressions)
```sql
WITH TotalRevenues AS (
    SELECT 
        customers.company_name, 
        SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total
    FROM customers
    INNER JOIN orders ON customers.customer_id = orders.customer_id
    INNER JOIN order_details ON order_details.order_id = orders.order_id
    GROUP BY customers.company_name
)
SELECT * FROM TotalRevenues;
```


## 4. Subqueries
```sql
SELECT product_id FROM (
SELECT product_id 
FROM (
	SELECT product_id, rank
	FROM (SELECT 
			product_id,
			SUM( det.quantity * det.unit_price * ( 1 - det.discount )) sold_value,
			RANK() OVER (ORDER BY SUM( det.quantity * det.unit_price * ( 1 - det.discount )) DESC) rank -- WINDOW FUNCTION
		FROM order_details det
		GROUP BY det.product_id
		ORDER BY rank)
	WHERE rank <= 5 )
WHERE product_id BETWEEN 35 and 65 )
ORDER BY product_id DESC
```

### 4.1 Turning into CTE
```sql
WITH CalculatedValues AS (
    SELECT 
        product_id,
        SUM(det.quantity * det.unit_price * (1 - det.discount)) AS sold_value,
        RANK() OVER (ORDER BY SUM(det.quantity * det.unit_price * (1 - det.discount)) DESC) AS rank
    FROM order_details det
    GROUP BY product_id
), 
TopRankedProducts AS ( 
    SELECT product_id FROM CalculatedValues WHERE rank <= 5 
), 
FilteredProducts AS ( 
    SELECT product_id FROM TopRankedProducts WHERE product_id BETWEEN 35 AND 65 
)
 
SELECT product_id FROM FilteredProducts ORDER BY product_id DESC;
```

## 5. Views
```sql
CREATE VIEW TotalRevenues AS
SELECT 
    customers.company_name, 
    SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total
FROM customers
INNER JOIN orders ON customers.customer_id = orders.customer_id
INNER JOIN order_details ON order_details.order_id = orders.order_id
GROUP BY customers.company_name;

SELECT * FROM TotalRevenues;
```

```sql
GRANT SELECT ON TotalRevenues TO user1;
```

## 6. Temporary Tables / Staging / Testes ETL
```sql
CREATE TEMP TABLE TempTotalRevenues AS
SELECT 
    customers.company_name, 
    SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total
FROM customers
INNER JOIN orders ON customers.customer_id = orders.customer_id
INNER JOIN order_details ON order_details.order_id = orders.order_id
GROUP BY customers.company_name;

SELECT * FROM TempTotalRevenues;
```
## 7. Materialized Views / Snapshot

```sql
CREATE MATERIALIZED VIEW MaterializedTotalRevenues AS
SELECT 
    customers.company_name, 
    SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total
FROM customers
INNER JOIN orders ON customers.customer_id = orders.customer_id
INNER JOIN order_details ON order_details.order_id = orders.order_id
GROUP BY customers.company_name;

SELECT * FROM MaterializedTotalRevenues;
```
