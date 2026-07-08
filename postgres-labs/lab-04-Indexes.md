# Lab 04 — Indexes

> Understanding how PostgreSQL finds rows efficiently without scanning an entire table.

---

## Objective

As databases grow, searching every row becomes increasingly expensive.

Imagine a table containing millions of records.

How does PostgreSQL quickly find a single row without reading the entire table?

The answer is **Indexes**.

By the end of this lab, you should understand what indexes are, why they improve query performance, their trade-offs, and why PostgreSQL sometimes chooses not to use them.

---

# Key Concepts

## Sequential Scan

A Sequential Scan is PostgreSQL's simplest way of finding data.

It reads every page in a table until it finds the matching row.

For small tables, this is often the fastest approach.

Example:

```sql
SELECT *
FROM users
WHERE email = 'philip@example.com';
```

Without an index, PostgreSQL checks every page until it finds the matching row.

---

## Index

An index is a separate data structure that helps PostgreSQL locate rows quickly.

Instead of searching every page, PostgreSQL can use the index to jump directly to the page containing the requested data.

Think of an index like the table of contents in a book.

Rather than reading every page, you jump directly to the section you need.

---

## B-tree Index

The default index type in PostgreSQL is the **B-tree**.

Whenever you create an index without specifying a type, PostgreSQL creates a B-tree index.

Example:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

B-tree indexes are excellent for:

- Equality searches (`=`)
- Range queries (`>`, `<`, `BETWEEN`)
- Sorting (`ORDER BY`)

---

# How Indexes Work

Imagine the following table.

| id | name |
|----|------|
|1|Alice|
|2|Bob|
|3|Philip|
|4|Jane|

Without an index, PostgreSQL searches every page until it finds **Philip**.

```
Page 1

↓

Page 2

↓

Page 3

↓

Result
```

With an index, PostgreSQL first looks in the index.

```
Index

↓

Page 3

↓

Tuple

↓

Result
```

Instead of searching every page, PostgreSQL immediately knows where the data is located.

---

# Why Indexes Improve Performance

Remember what we learned in Lab 01.

PostgreSQL reads **pages**, not individual rows.

Without an index:

```
Read Page 1

↓

Read Page 2

↓

Read Page 3

↓

Read Page 4
```

With an index:

```
Read Index

↓

Read One Page

↓

Return Result
```

The fewer pages PostgreSQL reads, the faster the query becomes.

---

# The Cost of Indexes

Indexes are not free.

Every INSERT, UPDATE, and DELETE must also update the affected indexes.

Imagine a table with five indexes.

Every INSERT becomes:

```
Insert Row

↓

Update Index 1

↓

Update Index 2

↓

Update Index 3

↓

Update Index 4

↓

Update Index 5
```

This means:

- Reads become faster.
- Writes become slower.

Support Engineers should always consider this trade-off before recommending new indexes.

---

# Why PostgreSQL Doesn't Always Use an Index

Many beginners believe PostgreSQL should always use an index if one exists.

That is not true.

Examples where PostgreSQL may prefer a Sequential Scan include:

- The table is very small.
- The query returns most of the rows.
- Reading the entire table is cheaper than using the index.
- PostgreSQL's planner estimates that an index would be slower.

The PostgreSQL planner always chooses the option with the lowest estimated cost.

---

# Why This Matters for Support Engineers

Imagine a customer says:

> "We created an index, but our query is still slow."

Instead of assuming PostgreSQL is wrong, investigate:

- Is PostgreSQL actually using the index?
- Is the table large enough?
- Does the query return too many rows?
- Would a Sequential Scan actually be faster?

Understanding indexes helps explain many performance-related support tickets.

---

# Knowledge Check

## 1. What is an index?

An index is a separate data structure that helps PostgreSQL locate rows efficiently without scanning the entire table.

---

## 2. Why are indexes faster?

Indexes reduce the number of pages PostgreSQL must read to locate matching rows.

---

## 3. Why do indexes slow down writes?

Every INSERT, UPDATE, and DELETE must also update any indexes associated with the affected rows.

---

## 4. What is a Sequential Scan?

A Sequential Scan reads every page in a table until the required rows are found.

---

## 5. Why might PostgreSQL ignore an index?

Because the planner estimates that reading the entire table is cheaper than using the index.

---

# Diagram

```text
Without Index

Query

↓

Page 1

↓

Page 2

↓

Page 3

↓

Result


--------------------------------------


With Index

Query

↓

Index

↓

Correct Page

↓

Correct Tuple

↓

Result
```

---

# Key Takeaways

- An index helps PostgreSQL locate rows quickly.
- PostgreSQL reads pages, not individual rows.
- Without an index, PostgreSQL may perform a Sequential Scan.
- B-tree is PostgreSQL's default index type.
- Indexes improve read performance.
- Indexes increase the cost of INSERT, UPDATE, and DELETE operations.
- PostgreSQL uses indexes only when they reduce the estimated query cost.

---

# Support Engineer Mindset

When troubleshooting slow queries, don't immediately recommend adding an index.

Instead, ask yourself:

- Is PostgreSQL performing a Sequential Scan?
- Is an appropriate index already available?
- Is the table large enough for an index to help?
- Does the query return a large percentage of the table?
- Is the planner choosing the cheapest execution plan?

Great Support Engineers don't add indexes blindly—they understand when an index provides a real benefit.

---

# Looking Ahead

In the next lab, we'll study one of PostgreSQL's most intelligent components:

> **The Query Planner.**

We'll answer questions such as:

- Why does PostgreSQL sometimes ignore an index?
- How does PostgreSQL estimate query cost?
- What statistics influence the planner's decisions?
- How can Support Engineers verify the chosen execution plan?

By understanding the planner, you'll begin thinking the way PostgreSQL itself makes decisions.