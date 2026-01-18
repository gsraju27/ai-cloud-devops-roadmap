# Complete PostgreSQL Interview Guide

Master PostgreSQL with these real-world interview questions covering core concepts, query optimization, and database administration. Practice scenarios that mirror actual backend engineering challenges.

**Companies that ask these questions:** Apple | Instagram | Spotify | Reddit | Twitch

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | What is the difference between `INNER JOIN`, `LEFT JOIN`, an... | Conceptual | SQL |
| 2 | Your database query is slow. How do you optimize it? | Debugging | Query Optimization |
| 3 | What are indexes and how do they work in PostgreSQL? | Conceptual | Indexing |
| 4 | What are transactions and why are they useful in PostgreSQL? | Conceptual | Transactions |
| 5 | How do you set up replication in PostgreSQL? | Practical | Replication |
| 6 | How do you do a backup and restore of a PostgreSQL database? | Practical | Backup and Recovery |
| 7 | What is the `JSONB` data type and how do you use it in Postg... | Conceptual | JSONB |
| 8 | How do you do full-text search in PostgreSQL? | Practical | Full-text Search |
| 9 | What are window functions and how do you use them in Postgre... | Conceptual | SQL |
| 10 | What are common table expressions (CTEs) and how do you use ... | Conceptual | SQL |
| 11 | What is the difference between `UNION` and `UNION ALL` in Po... | Conceptual | SQL |
| 12 | What is database normalization and why is it important? | Conceptual | Database Design |

---

## What You'll Learn

- Understand the core concepts of PostgreSQL.
- Master SQL queries.
- Optimize database performance.
- Administer a PostgreSQL database.
- Work with advanced features like JSONB and full-text search.

---

## Interview Questions

### Question 1: What is the difference between `INNER JOIN`, `LEFT JOIN`, and `RIGHT JOIN`?

**Type:** Conceptual | **Category:** SQL

## The Scenario

You are a backend engineer at an e-commerce company. You are writing a new service that needs to query the database to get a list of all the customers and their orders.

You have two tables: `customers` and `orders`. The `customers` table has a `customer_id` column, and the `orders` table has a `customer_id` column that is a foreign key to the `customers` table.

You need to decide which type of `JOIN` to use to get the data you need.

## The Challenge

Explain the difference between an `INNER JOIN`, a `LEFT JOIN`, and a `RIGHT JOIN`. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might not be aware of the different types of `JOIN`s. They might just use an `INNER JOIN` for everything, which would not always be the correct choice.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between the different types of `JOIN`s. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| `JOIN` Type      | Description                                                                                             |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| **`INNER JOIN`** | Returns only the rows that have a matching value in both tables.                                        |
| **`LEFT JOIN`**  | Returns all the rows from the left table, and the matching rows from the right table. If there is no match, the result is `NULL` on the right side. |
| **`RIGHT JOIN`** | Returns all the rows from the right table, and the matching rows from the left table. If there is no match, the result is `NULL` on the left side. |

### Step 2: Choose the Right Tool for the Job

For our use case, we want to get a list of all the customers and their orders.

-   If we want to get a list of only the customers who have placed an order, we should use an **`INNER JOIN`**.
-   If we want to get a list of all the customers, regardless of whether they have placed an order, we should use a **`LEFT JOIN`**.
-   If we want to get a list of all the orders, regardless of whether they are associated with a customer, we should use a **`RIGHT JOIN`**.

### Step 3: Code Examples

Here are some code examples that show the difference between the three approaches:

**`INNER JOIN`:**

```sql
SELECT customers.customer_name, orders.order_id
FROM customers
INNER JOIN orders ON customers.customer_id = orders.customer_id;
```

**`LEFT JOIN`:**

```sql
SELECT customers.customer_name, orders.order_id
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id;
```

**`RIGHT JOIN`:**

```sql
SELECT customers.customer_name, orders.order_id
FROM customers
RIGHT JOIN orders ON customers.customer_id = orders.customer_id;
```


---

### Quick Check

**You want to get a list of all the products and the number of times each product has been ordered, including products that have never been ordered. Which of the following would you use?**

   A. `INNER JOIN`
-> B. **`LEFT JOIN`**
   C. `RIGHT JOIN`
   D. `FULL OUTER JOIN`

<details>
<summary>See Answer</summary>

A `LEFT JOIN` is the correct choice for this task. It will return all the products, and it will show a count of 0 for products that have never been ordered.

</details>

---

### Question 2: Your database query is slow. How do you optimize it?

**Type:** Debugging | **Category:** Query Optimization

## The Scenario

You are a backend engineer at an e-commerce company. You are responsible for a service that is experiencing performance issues. The service is slow to respond to requests, and you have identified that the bottleneck is a slow database query.

The query is a complex `JOIN` between several tables, and it is taking several seconds to execute.

## The Challenge

Explain your strategy for optimizing the slow database query. What are the key steps you would take, and what tools would you use to help you?


> **Common Mistake:** A junior engineer might try to solve the problem by just adding more indexes. This might help, but it might not be the most effective solution. They might not be aware of the `EXPLAIN` command or the other tools that can be used to analyze the query plan.

> **Senior Engineer Approach:** A senior engineer would have a clear strategy for optimizing a slow database query. They would be able to explain how to use the `EXPLAIN` command to analyze the query plan, and they would have a clear plan for how to use this information to optimize the query.

### Step 1: Analyze the Query Plan

The first step is to analyze the query plan. The query plan is a description of how the database will execute the query. You can get the query plan by using the `EXPLAIN` command.

```sql
EXPLAIN ANALYZE SELECT * FROM my_table WHERE my_column = 'my_value';
```

The `EXPLAIN ANALYZE` command will execute the query and then show you the query plan, along with the actual execution time of each step.

### Step 2: Identify the Bottleneck

Once you have the query plan, you can use it to identify the bottleneck. Look for the parts of the query plan that are taking the most time to execute.

Common bottlenecks include:

-   **Full table scans:** The database is scanning the entire table to find the rows that match the `WHERE` clause.
-   **Nested loops:** The database is using a nested loop to join two tables.
-   **Slow sorting:** The database is taking a long time to sort the results.

### Step 3: Optimize the Query

Once you have identified the bottleneck, you can optimize the query by:

-   **Adding an index:** An index can be used to speed up the process of finding the rows that match the `WHERE` clause.
-   **Rewriting the query:** You might be able to rewrite the query in a way that is more efficient.
-   **Denormalizing the data:** You might be able to denormalize the data to avoid a complex `JOIN`.

### Step 4: Measure the Performance Improvement

After you have optimized the query, you should re-run the `EXPLAIN ANALYZE` command to measure the performance improvement.


---

### Quick Check

**You are looking at a query plan and you see that the database is doing a full table scan. What is the most likely cause of this?**

-> A. **There is no index on the column in the `WHERE` clause.**
   B. The table is very small.
   C. The query is very complex.
   D. The database is not configured correctly.

<details>
<summary>See Answer</summary>

A full table scan occurs when the database has to scan the entire table to find the rows that match the `WHERE` clause. This usually happens when there is no index on the column in the `WHERE` clause.

</details>

---

### Question 3: What are indexes and how do they work in PostgreSQL?

**Type:** Conceptual | **Category:** Indexing

## The Scenario

You are a backend engineer at an e-commerce company. You are designing a new database schema for a new service. You need to decide which columns to index to optimize the performance of your queries.

## The Challenge

Explain what indexes are in PostgreSQL and how they work. What are the different types of indexes, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might not be aware of indexes. They might just create a table without any indexes, which would lead to poor performance for large tables.

> **Senior Engineer Approach:** A senior engineer would know that indexes are a critical part of database performance. They would be able to explain what indexes are and how they work. They would also be able to explain the different types of indexes and would have a clear plan for how to use them to optimize the performance of their queries.

### Step 1: Understand What Indexes Are

An index is a data structure that is used to speed up the process of finding rows in a table. It works by creating a copy of the indexed column(s) and storing them in a sorted order. This allows the database to quickly find the rows that match a `WHERE` clause without having to scan the entire table.

### Step 2: The Different Types of Indexes

| Index Type | Description                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| **B-Tree** | The default index type in PostgreSQL. It is a self-balancing tree that is good for a wide variety of workloads. |
| **Hash**   | Can only handle simple equality comparisons. Not recommended for most use cases.                        |
| **GIN**    | Designed for indexing composite values, such as arrays and JSONB.                                     |
| **GiST**   | Designed for indexing geometric data and full-text search.                                            |

### Step 3: Choose the Right Tool for the Job

| Use Case                               | Recommended Index Type |
| -------------------------------------- | ---------------------- |
| Most use cases                         | B-Tree                 |
| Indexing an array or JSONB column      | GIN                    |
| Indexing geometric data or full-text search | GiST                   |

### Step 4: Code Examples

Here's how we can create an index in PostgreSQL:

```sql
CREATE INDEX my_index ON my_table (my_column);
```

### The Trade-offs of Using Indexes

| Pros                               | Cons                                                              |
| ---------------------------------- | ----------------------------------------------------------------- |
| Speeds up `SELECT` queries.        | Slows down `INSERT`, `UPDATE`, and `DELETE` queries.              |
|                                    | Takes up disk space.                                              |

Because of these trade-offs, you should only create indexes on the columns that you frequently use in `WHERE` clauses.


---

### Quick Check

**You want to create an index on a JSONB column. Which of the following would be the most appropriate?**

   A. B-Tree
   B. Hash
-> C. **GIN**
   D. GiST

<details>
<summary>See Answer</summary>

A GIN index is the correct choice for this task. It is designed for indexing composite values, such as arrays and JSONB.

</details>

---

### Question 4: What are transactions and why are they useful in PostgreSQL?

**Type:** Conceptual | **Category:** Transactions

## The Scenario

You are a backend engineer at a fintech company. You are writing a new service that needs to transfer money from one account to another.

The transfer consists of two separate operations: a `UPDATE` to the sender's account to debit the money, and a `UPDATE` to the receiver's account to credit the money.

You need to make sure that both operations are executed successfully, or that neither of them are.

## The Challenge

Explain what transactions are in PostgreSQL and how you would use them to solve this problem. What are the key properties of a transaction (ACID)?


### Step 1: Understand What Transactions Are

A transaction is a sequence of one or more operations that are executed as a single logical unit.

### Step 2: The ACID Properties

A transaction must have the following four properties:

| Property       | Description                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------- |
| **Atomicity**  | All the operations in a transaction are executed successfully, or none of them are.                   |
| **Consistency**| A transaction brings the database from one valid state to another.                                      |
| **Isolation**  | The intermediate state of a transaction is not visible to other transactions.                            |
| **Durability** | Once a transaction has been committed, it will remain committed even in the event of a power failure.      |

### Step 3: Use a Transaction

Here's how we can use a transaction to transfer money from one account to another:

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
```

In this example, the `BEGIN` command starts a new transaction. The two `UPDATE` statements are then executed. If both `UPDATE` statements are successful, the `COMMIT` command will commit the transaction and make the changes permanent. If either `UPDATE` statement fails, the transaction will be rolled back and the changes will be undone.


---

### Quick Check

**You are in the middle of a transaction and you want to undo all the changes that you have made so far. Which of the following would you use?**

   A. `COMMIT`
-> B. **`ROLLBACK`**
   C. `SAVEPOINT`
   D. None of the above

<details>
<summary>See Answer</summary>

`ROLLBACK` is the correct choice for this task. It will undo all the changes that you have made in the current transaction.

</details>

---

### Question 5: How do you set up replication in PostgreSQL?

**Type:** Practical | **Category:** Replication

## The Scenario

You are a database administrator at a social media company. You are responsible for a PostgreSQL database that is critical to the business.

You need to set up replication to a secondary database to provide high availability and disaster recovery.

## The Challenge

Explain how you would set up replication in PostgreSQL. What are the different types of replication, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might not be aware of replication. They might try to solve this problem by just taking periodic backups of the database, which would not provide high availability.

> **Senior Engineer Approach:** A senior engineer would know that replication is a critical part of database administration. They would be able to explain the different types of replication and would have a clear plan for how to set up replication for a production database.

### Step 1: Understand the Different Types of Replication

| Replication Type | Description                                                                                             |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| **Streaming Replication** | The primary server sends a stream of WAL (Write-Ahead Logging) records to the secondary server. The secondary server then replays the WAL records to keep itself up-to-date. |
| **Logical Replication** | The primary server sends a stream of logical changes to the secondary server. The secondary server then applies the logical changes to its own data. |

For our use case, we will use **streaming replication**. It is the most common type of replication, and it is well-suited for providing high availability and disaster recovery.

### Step 2: Configure the Primary Server

The first step is to configure the primary server. We need to edit the `postgresql.conf` file and set the following parameters:

```
wal_level = replica
max_wal_senders = 10
wal_keep_segments = 64
```

We also need to edit the `pg_hba.conf` file to allow the secondary server to connect to the primary server.

```
host  replication  all  <secondary_server_ip>/32  md5
```

### Step 3: Create a Base Backup

The next step is to create a base backup of the primary server. We can use the `pg_basebackup` utility to do this.

```bash
pg_basebackup -h <primary_server_ip> -D /var/lib/postgresql/data -U replication -P -v
```

### Step 4: Configure the Secondary Server

The final step is to configure the secondary server. We need to create a `recovery.conf` file in the data directory with the following content:

```
standby_mode = 'on'
primary_conninfo = 'host=<primary_server_ip> port=5432 user=replication password=...'
```

### Step 5: Start the Secondary Server

Now we can start the secondary server. It will connect to the primary server and start replaying the WAL records.


---

### Quick Check

**You want to replicate only a subset of the tables in your database. Which type of replication would you use?**

   A. Streaming Replication
-> B. **Logical Replication**
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

Logical replication is the correct choice for this task. It allows you to replicate only a subset of the tables in your database, and it also allows you to replicate between different versions of PostgreSQL.

</details>

---

### Question 6: How do you do a backup and restore of a PostgreSQL database?

**Type:** Practical | **Category:** Backup and Recovery

## The Scenario

You are a database administrator at a fintech company. You are responsible for a PostgreSQL database that is critical to the business.

You need to make sure that you have a backup of the database so that you can restore it in the event of a disaster.

## The Challenge

Explain how you would do a backup and restore of a PostgreSQL database. What are the different types of backups, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might not be aware of the different types of backups. They might just use `pg_dump` to create a logical backup, which would not be a very robust solution for a large database.

> **Senior Engineer Approach:** A senior engineer would know that there are two main types of backups: logical and physical. They would be able to explain the trade-offs between the two and would have a clear plan for how to do a backup and restore of a production database.

### Step 1: Understand the Different Types of Backups

| Backup Type | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **Logical** | A logical backup is a human-readable file that contains the SQL commands to recreate the database.        |
| **Physical**| A physical backup is a copy of the files that make up the database.                                     |

### Step 2: Choose the Right Tool for the Job

| Use Case                               | Recommended Tool    |
| -------------------------------------- | ------------------- |
| Backing up a small database            | `pg_dump`             |
| Backing up a large database            | `pg_basebackup`       |
| Point-in-time recovery                 | Continuous archiving |

### Step 3: Create a Backup

Here's how we can create a backup using `pg_dump`:

```bash
pg_dump -U myuser -d mydb > mydb.sql
```

Here's how we can create a backup using `pg_basebackup`:

```bash
pg_basebackup -h myhost -D /path/to/backup -U myuser -P
```

### Step 4: Restore a Backup

Here's how we can restore a backup from a `pg_dump` file:

```bash
psql -U myuser -d mydb < mydb.sql
```

To restore a backup from a `pg_basebackup`, you just need to start a new PostgreSQL server using the backup directory as the data directory.

### Continuous Archiving and Point-in-Time Recovery (PITR)

For a production database, you would want to use continuous archiving and point-in-time recovery (PITR). This involves continuously archiving the WAL (Write-Ahead Logging) files to a separate location. This allows you to restore the database to any point in time.


---

### Quick Check

**You need to be able to restore your database to any point in time. Which of the following would you use?**

   A. `pg_dump`
   B. `pg_basebackup`
-> C. **Continuous archiving and PITR**
   D. None of the above

<details>
<summary>See Answer</summary>

Continuous archiving and PITR is the correct choice for this task. It allows you to restore the database to any point in time by replaying the WAL files.

</details>

---

### Question 7: What is the `JSONB` data type and how do you use it in PostgreSQL?

**Type:** Conceptual | **Category:** JSONB

## The Scenario

You are a backend engineer at a social media company. You are building a new service that needs to store and query large amounts of JSON data.

You are considering using either a `TEXT` column or a `JSONB` column to store the JSON data.

## The Challenge

Explain what the `JSONB` data type is in PostgreSQL and why it is a better choice than a `TEXT` column for storing JSON data. What are the key operators that you would use to query a `JSONB` column?


> **Common Mistake:** A junior engineer might try to store the JSON data in a `TEXT` column. This would work, but it would be very inefficient to query the data. They might not be aware of the `JSONB` data type or the special operators that can be used to query it.

> **Senior Engineer Approach:** A senior engineer would know that `JSONB` is the perfect tool for this job. They would be able to explain what `JSONB` is and how to use it to store and query JSON data efficiently.

### Step 1: `JSONB` vs. `TEXT`

| Feature      | `TEXT`                                                              | `JSONB`                                                              |
| ------------ | ------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Storage**  | Stores the JSON data as a string.                                   | Stores the JSON data in a decomposed binary format.                 |
| **Performance**| Slow to query, because the entire string has to be parsed each time. | Fast to query, because the data is already in a binary format.      |
| **Indexing** | Can be indexed with a full-text search index.                         | Can be indexed with a GIN index, which is much more efficient.      |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use a **`JSONB` column**. This is because we need to be able to query the JSON data efficiently.

### Step 3: Code Examples

Here's how we can create a table with a `JSONB` column:

```sql
CREATE TABLE my_table (
    id SERIAL PRIMARY KEY,
    my_data JSONB
);
```

Here's how we can insert some JSON data into the table:

```sql
INSERT INTO my_table (my_data) VALUES ('{"name": "John", "age": 30}');
```

Here's how we can query the JSON data:

```sql
-- Get the value of the "name" key
SELECT my_data->>'name' FROM my_table;

-- Check if the "age" key is greater than 25
SELECT * FROM my_table WHERE (my_data->>'age')::int > 25;

-- Check if the data contains a certain key
SELECT * FROM my_table WHERE my_data ? 'name';
```


---

### Quick Check

**You want to create an index on a `JSONB` column to speed up queries that use the `?` operator. Which type of index would you use?**

   A. B-Tree
   B. Hash
-> C. **GIN**
   D. GiST

<details>
<summary>See Answer</summary>

A GIN index is the correct choice for this task. It is designed for indexing composite values, such as arrays and JSONB, and it can be used to speed up queries that use the `?`, `?|`, and `?&` operators.

</details>

---

### Question 8: How do you do full-text search in PostgreSQL?

**Type:** Practical | **Category:** Full-text Search

## The Scenario

You are a backend engineer at a social media company. You are building a new service that needs to be able to search for posts that contain a certain keyword.

You could use the `LIKE` operator to do this, but you know that this would be very inefficient for a large table.

## The Challenge

Explain how you would do full-text search in PostgreSQL. What are the key features of PostgreSQL's full-text search, and how would you use them to solve this problem?


> **Common Mistake:** A junior engineer might try to solve this problem by using the `LIKE` operator. This would be very inefficient for a large table, and it would not provide a very good search experience.

> **Senior Engineer Approach:** A senior engineer would know that PostgreSQL has a powerful full-text search engine built-in. They would be able to explain how to use the `tsvector` and `tsquery` data types to do full-text search, and they would have a clear plan for how to use them to solve this problem.

### Step 1: Understand the Key Concepts

| Concept      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **`tsvector`** | A data type that represents a document in a format that is optimized for full-text search.             |
| **`tsquery`**  | A data type that represents a search query.                                                             |
| **`to_tsvector()`** | A function that converts a string to a `tsvector`.                                                    |
| **`to_tsquery()`**  | A function that converts a string to a `tsquery`.                                                     |
| **`@@`**       | A match operator that returns `true` if a `tsvector` matches a `tsquery`.                               |

### Step 2: Create a `tsvector` Column

The first step is to create a `tsvector` column on your table.

```sql
ALTER TABLE posts ADD COLUMN tsv tsvector;
```

### Step 3: Populate the `tsvector` Column

The next step is to populate the `tsvector` column with the content of the posts.

```sql
UPDATE posts SET tsv = to_tsvector('english', title || ' ' || body);
```

You can also use a trigger to automatically update the `tsvector` column whenever a post is inserted or updated.

### Step 4: Create a GIN Index

To speed up full-text search queries, you should create a GIN index on the `tsvector` column.

```sql
CREATE INDEX posts_tsv_idx ON posts USING gin(tsv);
```

### Step 5: Perform a Full-Text Search

Now you can perform a full-text search using the `@@` operator.

```sql
SELECT title, body FROM posts WHERE tsv @@ to_tsquery('english', 'keyword');
```

---

### Question 9: What are window functions and how do you use them in PostgreSQL?

**Type:** Conceptual | **Category:** SQL

## The Scenario

You are a data analyst at an e-commerce company. You are trying to write a query to find the top 10 products by sales in each category.

You could do this with a series of subqueries, but you know that this would be inefficient and difficult to read.

## The Challenge

Explain what window functions are in PostgreSQL and how you would use them to solve this problem. What are the key benefits of using window functions?


> **Common Mistake:** A junior engineer might not be aware of window functions. They might try to solve this problem with a series of subqueries, which would be inefficient and difficult to read.

> **Senior Engineer Approach:** A senior engineer would know that window functions are the perfect tool for this job. They would be able to explain what window functions are and how to use them to write concise and efficient queries for a variety of different use cases.

### Step 1: Understand What Window Functions Are

A window function is a type of function that performs a calculation across a set of table rows that are somehow related to the current row.

### Step 2: The Key Window Functions

| Function      | Description                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| **`ROW_NUMBER()`**| Assigns a unique number to each row in the partition.                                                 |
| **`RANK()`**   | Assigns a rank to each row in the partition, with gaps for ties.                                        |
| **`DENSE_RANK()`**| Assigns a rank to each row in the partition, without gaps for ties.                                     |
| **`LEAD()`**   | Returns the value of a column from the next row in the partition.                                       |
| **`LAG()`**    | Returns the value of a column from the previous row in the partition.                                   |

### Step 3: Solve the Problem

Here's how we can use a window function to find the top 10 products by sales in each category:

```sql
SELECT product_name, category_name, sales, rank
FROM (
    SELECT
        product_name,
        category_name,
        sales,
        RANK() OVER (PARTITION BY category_name ORDER BY sales DESC) as rank
    FROM products
) as ranked_products
WHERE rank <= 10;
```

In this example, we use the `RANK()` window function to assign a rank to each product based on its sales, within each category. We then filter the results to only include the products with a rank of 10 or less.


---

### Quick Check

**You want to find the second highest salary in each department. Which of the following would you use?**

   A. `ROW_NUMBER()`
   B. `RANK()`
-> C. **`DENSE_RANK()`**
   D. None of the above

<details>
<summary>See Answer</summary>

`DENSE_RANK()` is the correct choice for this task. It will assign a rank to each employee based on their salary, without any gaps for ties. You can then filter the results to only include the employees with a rank of 2.

</details>

---

### Question 10: What are common table expressions (CTEs) and how do you use them in PostgreSQL?

**Type:** Conceptual | **Category:** SQL

## The Scenario

You are a data analyst at an e-commerce company. You are trying to write a complex query that involves multiple levels of subqueries. The query is difficult to read and maintain.

## The Challenge

Explain what common table expressions (CTEs) are in PostgreSQL and how you would use them to solve this problem. What are the key benefits of using CTEs?


> **Common Mistake:** A junior engineer might not be aware of CTEs. They might try to solve this problem by using a series of subqueries, which would be difficult to read and maintain.

> **Senior Engineer Approach:** A senior engineer would know that CTEs are the perfect tool for this job. They would be able to explain what CTEs are and how to use them to write concise and readable queries for a variety of different use cases.

### Step 1: Understand What CTEs Are

A common table expression (CTE) is a temporary named result set that you can reference within another SQL statement.

### Step 2: The `WITH` Clause

A CTE is defined using the `WITH` clause.

```sql
WITH my_cte AS (
    SELECT * FROM my_table
)
SELECT * FROM my_cte;
```

### Step 3: Solve the Problem

Here's how we can use a CTE to simplify a complex query:

**Without CTEs:**

```sql
SELECT *
FROM (
    SELECT *
    FROM my_table
    WHERE my_column = 'my_value'
) as my_subquery
WHERE my_subquery.my_other_column = 'my_other_value';
```

**With CTEs:**

```sql
WITH my_cte AS (
    SELECT *
    FROM my_table
    WHERE my_column = 'my_value'
)
SELECT *
FROM my_cte
WHERE my_other_column = 'my_other_value';
```

As you can see, the query with the CTE is much more readable and easier to understand.

### Recursive CTEs

You can also use recursive CTEs to perform recursive queries, such as traversing a hierarchical data structure.


---

### Quick Check

**You want to write a query to find all the employees who report to a specific manager, and all the employees who report to those employees, and so on. Which of the following would you use?**

   A. A subquery
   B. A `JOIN`
-> C. **A recursive CTE**
   D. A window function

<details>
<summary>See Answer</summary>

A recursive CTE is the correct choice for this task. It allows you to traverse a hierarchical data structure, such as an employee-manager relationship.

</details>

---

### Question 11: What is the difference between `UNION` and `UNION ALL` in PostgreSQL?

**Type:** Conceptual | **Category:** SQL

## The Scenario

You are a data analyst at an e-commerce company. You are trying to write a query to get a list of all the customers from two different tables: `customers_a` and `customers_b`.

You are not sure whether to use `UNION` or `UNION ALL` to combine the results of the two queries.

## The Challenge

Explain the difference between `UNION` and `UNION ALL` in PostgreSQL. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in performance or the fact that `UNION` removes duplicate rows.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `UNION` and `UNION ALL`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | `UNION`                                                              | `UNION ALL`                                                              |
| ------------ | ------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Duplicates** | Removes duplicate rows from the result set.                         | Includes all duplicate rows in the result set.                          |
| **Performance**| Slower than `UNION ALL`, because it has to sort the result set to remove the duplicates. | Faster than `UNION`, because it does not have to remove duplicates. |
| **Use Cases**  | When you want to combine the results of two queries and remove any duplicate rows. | When you want to combine the results of two queries and keep all the duplicate rows. |

### Step 2: Choose the Right Tool for the Job

For our use case, we want to get a list of all the customers from two different tables. If there is a possibility that the same customer could be in both tables, and we want to see the customer only once in the result set, we should use **`UNION`**.

If we want to see all the customers from both tables, even if there are duplicates, we should use **`UNION ALL`**.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`UNION`:**

```sql
SELECT customer_name FROM customers_a
UNION
SELECT customer_name FROM customers_b;
```

**`UNION ALL`:**

```sql
SELECT customer_name FROM customers_a
UNION ALL
SELECT customer_name FROM customers_b;
```


---

### Quick Check

**You want to get a list of all the unique cities from two different tables: `customers` and `suppliers`. Which of the following would you use?**

-> A. **`UNION`**
   B. `UNION ALL`
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

`UNION` is the correct choice for this task. It will combine the results of the two queries and remove any duplicate cities.

</details>

---

### Question 12: What is database normalization and why is it important?

**Type:** Conceptual | **Category:** Database Design

## The Scenario

You are a database administrator at a fintech company. You are designing a new database schema for a new service. You need to make sure that the database schema is well-designed and that it does not have any data redundancy.

## The Challenge

Explain what database normalization is and why it is important. What are the different normal forms, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might not be aware of database normalization. They might just create a single table with all the data, which would lead to data redundancy and would be difficult to maintain.

> **Senior Engineer Approach:** A senior engineer would know that database normalization is a critical part of database design. They would be able to explain the different normal forms and would have a clear plan for how to design a well-normalized database schema.

### Step 1: Understand What Database Normalization Is

Database normalization is the process of organizing the columns and tables of a relational database to minimize data redundancy.

### Step 2: The Different Normal Forms

| Normal Form | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **1NF**     | Each column must have a unique name, and all values in a column must be atomic.                           |
| **2NF**     | Must be in 1NF, and all non-key columns must be fully dependent on the primary key.                       |
| **3NF**     | Must be in 2NF, and all non-key columns must not be dependent on any other non-key columns.               |
| **BCNF**    | A stricter version of 3NF.                                                                              |

### Step 3: The Trade-offs of Normalization

| Pros                               | Cons                                                              |
| ---------------------------------- | ----------------------------------------------------------------- |
| **Reduces data redundancy**        | Can lead to more complex queries with more `JOIN`s.               |
| **Improves data integrity**        | Can be more difficult to design and implement.                    |
| **Makes the database more flexible**|                                                                   |

### When to Denormalize

In some cases, you might want to denormalize your database to improve performance. Denormalization is the process of intentionally adding redundant data to your database to avoid complex `JOIN`s.

You should only denormalize your database if you have a good reason to do so, and you should be aware of the trade-offs.


---

### Quick Check

**You have a table with a column that contains a list of comma-separated values. Which normal form is this table not in?**

-> A. **1NF**
   B. 2NF
   C. 3NF
   D. BCNF

<details>
<summary>See Answer</summary>

This table is not in 1NF, because all values in a column must be atomic. A list of comma-separated values is not atomic.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
