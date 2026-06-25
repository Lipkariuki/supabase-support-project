# Lab 00 – PostgreSQL Architecture

## Objectives

By the end of this lab, I should understand:

* How PostgreSQL processes a SQL query
* The role of the Parser, Planner, and Executor
* Why PostgreSQL optimizes queries before execution
* Why understanding the query lifecycle is essential when troubleshooting slow queries

---

# 1. What is the role of the Parser?

The **Parser** is the first PostgreSQL component that handles an incoming SQL statement.

Its primary responsibilities are:

* Verify that the SQL syntax is valid.
* Ensure SQL keywords and commands are correctly structured.
* Identify referenced database objects such as tables, columns, functions, and operators.
* Convert the SQL statement into an internal structure known as a **parse tree**.

### Example

Input SQL:

```sql
SELECT name
FROM users
WHERE id = 1;
```

The Parser checks that:

* `SELECT` is valid SQL.
* The query follows PostgreSQL grammar.
* `users` and `name` are valid identifiers (during parsing and analysis).
* The statement can be represented internally for further processing.

If the SQL is invalid:

```sql
SELEC * FROM users;
```

PostgreSQL immediately returns a syntax error because the Parser cannot understand the statement.

---

# 2. What is the difference between the Planner and the Executor?

Although they work closely together, they perform completely different tasks.

## Planner (Query Optimizer)

The Planner's job is to determine **the most efficient way** to execute the query.

It considers information such as:

* Table statistics
* Available indexes
* Estimated number of rows
* Join strategies
* Estimated execution costs

The Planner may decide to:

* Perform a Sequential Scan
* Use an Index Scan
* Use a Bitmap Heap Scan
* Choose Nested Loop, Hash Join, or Merge Join

The Planner does **not** retrieve any data.

Its only job is to create an **execution plan**.

---

## Executor

The Executor takes the plan produced by the Planner and actually performs the work.

It:

* Reads table pages
* Uses indexes if instructed
* Filters rows
* Performs joins
* Sorts data
* Aggregates results
* Returns the final rows to the client

Unlike the Planner, the Executor is responsible for interacting with PostgreSQL storage and producing results.

---

### Simple analogy

Imagine driving to another city.

The **Planner** decides:

* Which road to take
* Whether to use the highway
* Which route is fastest

The **Executor** is the person actually driving the car.

---

# 3. Why doesn't PostgreSQL execute SQL immediately after receiving it?

Executing SQL immediately could lead to extremely inefficient queries.

Instead, PostgreSQL first evaluates multiple possible execution strategies and estimates their costs.

For example, if a table contains **100 million rows**, PostgreSQL must decide whether it is faster to:

* Read the entire table (Sequential Scan), or
* Use an existing index (Index Scan)

Without planning, PostgreSQL might always read every row, making many queries unnecessarily slow.

The optimization phase allows PostgreSQL to choose the lowest-cost execution plan before any data is read.

This is one of the main reasons PostgreSQL performs well on very large datasets.

---

# 4. If a query is slow, why is understanding the query's journey useful for troubleshooting?

Understanding the query lifecycle helps identify **where** the problem occurs instead of guessing.

For example:

### Parser

Problems here include:

* Syntax errors
* Invalid table names
* Invalid SQL

---

### Planner

Problems here often involve:

* Missing indexes
* Outdated statistics
* Poor join strategies
* Incorrect cost estimates

This is often where Support Engineers investigate using:

```sql
EXPLAIN
```

or

```sql
EXPLAIN ANALYZE
```

---

### Executor

Problems here include:

* Reading too many rows
* Sequential scans on large tables
* Disk I/O bottlenecks
* Lock contention
* Long-running queries

---

Knowing which stage is responsible makes troubleshooting much faster and prevents unnecessary changes.

---

# 5. Query Flow

The complete lifecycle of a PostgreSQL query can be visualized as follows.

```text
Application
     │
     ▼
Client Driver
(psql, Supabase, API, etc.)
     │
     ▼
 PostgreSQL Server
     │
     ▼
+----------------+
|    Parser      |
+----------------+
     │
     ▼
+----------------+
| Analyzer       |
| & Rewriter     |
+----------------+
     │
     ▼
+----------------+
| Planner        |
| (Optimizer)    |
+----------------+
     │
     ▼
+----------------+
| Executor       |
+----------------+
     │
     ▼
Shared Buffers
     │
     ▼
Disk (Tables & Indexes)
     │
     ▼
Rows Returned
     │
     ▼
Application
```

---

# Key Takeaways

* Every SQL query passes through multiple stages before execution.
* The Parser validates SQL and creates an internal representation.
* The Planner selects the lowest-cost execution strategy.
* The Executor performs the actual work and retrieves data.
* Understanding this lifecycle is essential for diagnosing slow queries, interpreting `EXPLAIN ANALYZE`, and solving real production issues as a PostgreSQL or Support Engineer.
