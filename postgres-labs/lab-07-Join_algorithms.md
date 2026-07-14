# Lab 07 — Join Algorithms

⏱ Estimated Time: 20–25 minutes

> Understanding how PostgreSQL combines data from multiple tables.

---

## Objective

Most production queries involve joining multiple tables.

For example:

```sql
SELECT *
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

PostgreSQL has several ways of performing joins.

Rather than always using the same approach, the Query Planner evaluates multiple join algorithms and chooses the one with the lowest estimated cost.

By the end of this lab, you'll understand:

- What a Nested Loop is
- What a Hash Join is
- What a Merge Join is
- When PostgreSQL chooses each algorithm

---

# Key Concepts

## Join Algorithm

A join algorithm is the method PostgreSQL uses to combine rows from two or more tables.

Different algorithms work better depending on:

- Table size
- Number of matching rows
- Available indexes
- Whether the data is already sorted

The Query Planner selects the algorithm with the lowest estimated cost.

---

## Nested Loop

Nested Loop is the simplest join algorithm.

For every row in the first table, PostgreSQL searches for matching rows in the second table.

Imagine two tables.

Employees

```
Philip

Jane

Alice
```

Departments

```
HR

Finance

IT
```

PostgreSQL works like this:

```
Philip

↓

Search Departments

↓

Jane

↓

Search Departments

↓

Alice

↓

Search Departments
```

---

## When is Nested Loop Good?

Nested Loop performs well when:

- One table is relatively small.
- Indexes exist on the joined columns.
- Only a few matching rows are expected.

Nested Loop is commonly seen in OLTP databases such as Supabase applications.

---

## Hash Join

Hash Join works differently.

Instead of repeatedly searching one table, PostgreSQL first builds a hash table from one dataset.

```
Departments

↓

Hash Table

↓

Fast Lookup
```

It then searches the hash table for matching rows.

This greatly improves performance when joining larger datasets.

---

## When is Hash Join Good?

Hash Join performs well when:

- Many rows are being joined.
- The join condition uses equality (`=`).
- Building the hash table is cheaper than repeated index lookups.

---

## Merge Join

Merge Join is used when both datasets are already sorted.

Imagine:

Employees

```
1

2

3

4
```

Departments

```
1

2

3

4
```

Instead of searching repeatedly, PostgreSQL walks through both datasets together.

```
Employees →

← Departments
```

This makes Merge Join very efficient for sorted data.

---

## When is Merge Join Good?

Merge Join performs well when:

- Both datasets are already sorted.
- Large datasets are involved.
- Sorting is unnecessary or inexpensive.

---

# How PostgreSQL Chooses a Join Algorithm

The Query Planner evaluates multiple execution plans.

For example:

```
Nested Loop

Cost = 120

---------------------

Hash Join

Cost = 75

---------------------

Merge Join

Cost = 140
```

The planner selects:

```
Hash Join
```

Not because it is always faster,

but because it has the lowest estimated cost for that specific query.

---

# Multiple Join Algorithms

One SQL query can use multiple join algorithms.

Example:

```
Table A

↓

Hash Join

↓

Table B

↓

Nested Loop

↓

Table C

↓

Merge Join

↓

Table D
```

This is completely normal.

Each join is evaluated independently by the Query Planner.

---

# Why This Matters for Support Engineers

Imagine a customer reports:

> "Our search query became slower."

The execution plan shows:

```
Hash Join
```

A Support Engineer should not immediately assume something is wrong.

Instead, investigate:

- How many rows are being joined?
- Are indexes available?
- Why did PostgreSQL choose this join?
- Would another join algorithm actually be cheaper?

Execution plans should always be interpreted in context.

---

# Knowledge Check

## 1. What is a join algorithm?

A join algorithm is the method PostgreSQL uses to combine rows from multiple tables.

---

## 2. What is a Nested Loop?

Nested Loop joins tables by repeatedly searching one table for matching rows in another table.

---

## 3. What is a Hash Join?

Hash Join builds a hash table from one dataset to quickly locate matching rows.

---

## 4. What is a Merge Join?

Merge Join combines two sorted datasets by walking through both at the same time.

---

## 5. Does PostgreSQL use only one join algorithm per query?

No.

A single query can contain multiple join algorithms.

The planner chooses the best algorithm for each individual join.

---

# Diagram

```text
                 SQL Query

                     │

                     ▼

              Query Planner

         ┌────────┼────────┐

         ▼        ▼        ▼

   Nested Loop  Hash Join  Merge Join

         │

         ▼

 Lowest Estimated Cost

         │

         ▼

      Query Executes
```

---

# Key Takeaways

- PostgreSQL has multiple join algorithms.
- Nested Loop works well for small result sets and indexed lookups.
- Hash Join works well for larger equality joins.
- Merge Join works well for sorted datasets.
- PostgreSQL can use multiple join algorithms within a single query.
- The Query Planner chooses the algorithm with the lowest estimated cost.

---

# Support Engineer Mindset

When reading an execution plan, avoid judging a join algorithm in isolation.

Instead, ask yourself:

- Why did PostgreSQL choose this join?
- How many rows are being joined?
- Are indexes available?
- Is the planner making a reasonable decision?

Support Engineers investigate evidence before recommending changes.

---

# 🔍 What You'll See in Production

- Nested Loop is common in OLTP applications where indexes are heavily used.
- Hash Join often appears when joining larger datasets.
- Merge Join is less common but very efficient when both inputs are sorted.
- Large production queries frequently contain multiple join algorithms.

---

# Looking Ahead

Next we'll learn one of the most important PostgreSQL concepts for production troubleshooting:

> **Lab 08 — Locks & Concurrency**

You'll learn:

- Why queries sometimes "hang"
- What database locks are
- Why one transaction can block another
- How PostgreSQL protects data consistency
- How Support Engineers investigate blocking sessions

---

# 🎯 Interview Takeaway

After completing this lab, I can confidently explain:

- The three main PostgreSQL join algorithms.
- When PostgreSQL uses Nested Loop, Hash Join, and Merge Join.
- Why one query can contain multiple join algorithms.
- How the Query Planner selects join strategies.
- Why join algorithms should always be evaluated in the context of the execution plan.