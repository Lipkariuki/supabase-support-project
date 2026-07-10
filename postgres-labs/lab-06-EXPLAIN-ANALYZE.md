# Lab 06 — EXPLAIN & EXPLAIN ANALYZE

> Understanding how PostgreSQL executes queries and how Support Engineers investigate slow queries.

---

## Objective

Imagine a customer says:

> "Our query is taking 8 seconds."

How do you investigate?

You don't guess.

You ask PostgreSQL:

> **"Show me how you executed this query."**

That is exactly what **EXPLAIN** and **EXPLAIN ANALYZE** are for.

By the end of this lab, you'll understand:

- What EXPLAIN does
- What EXPLAIN ANALYZE does
- The difference between estimated and actual execution
- How to begin reading an execution plan

---

# Key Concepts

## EXPLAIN

EXPLAIN shows the execution plan PostgreSQL **plans** to use.

It does **not** execute the query.

Example:

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'philip@example.com';
```

Think of it as asking PostgreSQL:

> "How would you execute this query?"

---

## EXPLAIN ANALYZE

EXPLAIN ANALYZE actually executes the query.

After execution, PostgreSQL reports what really happened.

Example:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'philip@example.com';
```

This is the tool Support Engineers use when investigating slow queries.

---

# EXPLAIN vs EXPLAIN ANALYZE

| EXPLAIN | EXPLAIN ANALYZE |
|----------|-----------------|
| Shows estimated execution plan | Executes the query |
| Does not run the query | Runs the query |
| Shows estimated rows | Shows actual rows |
| Shows estimated cost | Shows actual execution time |

---

# Reading an Execution Plan

Consider the following output.

```text
Index Scan using idx_users_email

cost=0.28..8.30

rows=1

width=42
```

Let's break it down.

---

## Index Scan

This tells us **how PostgreSQL found the data**.

In this example, it used an index.

---

## Cost

Example:

```text
cost=0.28..8.30
```

Cost is PostgreSQL's internal estimate of how expensive the operation is.

> 💡 Remember

Cost is **not** execution time.

It is only used by the Query Planner to compare different execution strategies.

---

## Rows

Example:

```text
rows=1
```

This is PostgreSQL's estimate of how many rows the operation will return.

When using EXPLAIN ANALYZE, PostgreSQL also reports the **actual** number of rows.

---

## Width

Example:

```text
width=42
```

Width is the estimated average size of each returned row in bytes.

It helps PostgreSQL estimate the overall cost of the query.

---

# Common Execution Plan Nodes

These are some of the most common operations you'll encounter.

## Sequential Scan

Reads every page in a table.

---

## Index Scan

Uses an index to locate rows.

---

## Nested Loop

A join algorithm used to combine rows from two tables.

---

## Hash Join

Another join algorithm that uses a hash table to match rows efficiently.

---

## Sort

Orders rows before returning them.

---

## Limit

Stops processing once enough rows have been returned.

---

# Why EXPLAIN ANALYZE Matters

Suppose PostgreSQL estimates:

```text
rows = 10
```

But after executing the query, EXPLAIN ANALYZE reports:

```text
actual rows = 10,000
```

This tells us the planner's estimate was inaccurate.

One possible cause is outdated statistics.

Understanding the difference between estimated and actual values is one of the most important PostgreSQL troubleshooting skills.

---

# Why This Matters for Support Engineers

Imagine a customer says:

> "Our search endpoint became slow after yesterday's deployment."

Rather than guessing, a Support Engineer gathers evidence.

The first question is often:

> "What does EXPLAIN ANALYZE show?"

The execution plan tells us:

- Which indexes were used
- Whether PostgreSQL performed a Sequential Scan
- Which join algorithms were chosen
- How many rows were processed
- Where most of the execution time was spent

---

# Knowledge Check

## 1. What does EXPLAIN do?

EXPLAIN shows PostgreSQL's estimated execution plan without executing the query.

---

## 2. What does EXPLAIN ANALYZE do?

It executes the query and reports what actually happened during execution.

---

## 3. What does cost represent?

It is PostgreSQL's internal estimate of the work required to execute part of a query.

It is **not** execution time.

---

## 4. What is the difference between estimated rows and actual rows?

Estimated rows come from the Query Planner.

Actual rows are measured after the query executes.

---

## 5. Why is EXPLAIN ANALYZE valuable?

Because it allows Support Engineers to compare PostgreSQL's estimates with reality.

---

# Diagram

```text
SQL Query

        │

        ▼

      EXPLAIN

        │

Estimated Execution Plan

        │

────────────────────────────

SQL Query

        │

        ▼

EXPLAIN ANALYZE

        │

Query Executes

        │

Actual Execution Statistics

        │

Execution Time

Actual Rows

Actual Loops
```

---

# Key Takeaways

- EXPLAIN shows PostgreSQL's estimated execution plan.
- EXPLAIN ANALYZE executes the query and reports actual execution details.
- Cost is an estimate, not execution time.
- Comparing estimated rows with actual rows helps identify planner issues.
- Execution plans are one of the most valuable troubleshooting tools for PostgreSQL Support Engineers.

---

# Support Engineer Mindset

When investigating slow queries, don't immediately recommend adding an index.

Instead, ask:

- Which execution plan did PostgreSQL choose?
- Is it using an index?
- Is it performing a Sequential Scan?
- Are the estimated rows close to the actual rows?
- Is PostgreSQL making a reasonable decision based on the available information?

Support Engineers investigate evidence before proposing solutions.

---

# Looking Ahead

In the next lesson, we'll learn how PostgreSQL joins tables.

We'll answer questions such as:

- What is a Nested Loop?
- What is a Hash Join?
- When does PostgreSQL choose each join algorithm?
- How do join algorithms affect query performance?

You'll also revisit a real production execution plan and understand why PostgreSQL selected different join strategies.

---

# 🎯 Interview Takeaway

After completing this lab, I can confidently explain:

- The difference between EXPLAIN and EXPLAIN ANALYZE.
- Why execution plans are essential for troubleshooting.
- What cost, rows and width represent.
- The purpose of common execution plan nodes.
- Why comparing estimated and actual results helps diagnose performance problems.