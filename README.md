# Postgres SQL 


## Install Postgres and PGAdmin
In VsCode, Open terminal and run:
```bash
docker compose up
```

### Postgres Access
- URL: http://localhost:5050  
- password: postgres

### Config Connection
- host: db
- database: northwind
- user: postgres  
- PW: postgres  

This is based on [Northwind Github](https://github.com/pthom/northwind_psql)

![northwind](/images/northwind.png)

## SQL Commands

![sql_mindmap](/images/sql_mindmap.jpg)

## 1. DDL – Data Definition Language
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

## 2. DQL – Data Query Language
Allows getting the data out of the database to perform operations with it. When a SELECT is fired against a table or tables the result is compiled into a further temporary table.

Used By: Aplications, Front-End Web App, Developers, PO, PM, General use

|Command|Description|Syntax|
|---|---|---|
|SELECT|It is used to retrieve data from the database|SELECT column1, column2, ...FROM table_name WHERE condition;|

It looks simple, but its not. Check this file:
[Examples of DQL](examples-DQL.md)

## 3. DML – Data Manipulation Language
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

## 4. DCL – Data Control Language
Mainly deal with the rights, permissions, and other controls of the database system.

Used By: DBA

|Command|Description|Syntax|
|---|---|---|
|GRANT|Assigns new privileges to a user account, allowing access to specific database objects, actions, or functions.|GRANT privilege_type [(column_list)] ON [object_type] object_name TO user [WITH GRANT OPTION];|
|REVOKE|Removes previously granted privileges from a user account, taking away their access to certain database objects or actions.|REVOKE [GRANT OPTION FOR] privilege_type [(column_list)] ON [object_type] object_name FROM user [CASCADE];|

## 5. TCL – Transaction Control Language
Transactions group a set of tasks into a single execution unit. Each transaction begins with a specific task and ends when all the tasks in the group are successfully completed. If any of the tasks fail, the transaction fails.

Therefore, a transaction has only two results: success or failure.

Used By: Developers, Aplications, Data Engineers

|Command|Description|Syntax|
|---|---|---|
|BEGIN TRANSACTION|Starts a new transaction|BEGIN TRANSACTION [transaction_name];|
|COMMIT|Saves all changes made during the transaction|COMMIT;|
|ROLLBACK|Undoes all changes made during the transaction|ROLLBACK;|
|SAVEPOINT|Creates a savepoint within the current transaction|SAVEPOINT savepoint_name;|