# Lab 08 — Locks & Concurrency

> Understanding how PostgreSQL protects data when multiple users access the database at the same time.

---

⏱ **Estimated Time:** 25–30 minutes

🎯 **Difficulty:** Intermediate

📚 **Prerequisites:**

- Lab 02 — MVCC & Dead Tuples
- Lab 03 — VACUUM & Autovacuum
- Lab 05 — Query Planner
- Lab 06 — EXPLAIN & EXPLAIN ANALYZE

---

## Objective

Imagine two users trying to update the same row at the same time.

How does PostgreSQL prevent data corruption?

The answer is **locks**.

By the end of this lab, you'll understand:

- What locks are
- Why PostgreSQL needs them
- How MVCC and locks work together
- Why queries sometimes wait
- What a deadlock is

---

# Key Concepts

## Lock

A lock is PostgreSQL's way of protecting data while it is being modified.

Locks prevent multiple transactions from making conflicting changes at the same time.

---

## MVCC vs Locks

MVCC allows multiple users to read data simultaneously without blocking each other.

However, when two transactions try to modify the same row, PostgreSQL uses locks to ensure the changes happen safely.

---

## Blocking

Blocking occurs when one transaction is waiting for another transaction to release a lock.

This is normal behavior and helps maintain data consistency.

---

## Deadlock

A deadlock happens when two transactions wait for each other indefinitely.

Example:

Transaction A locks Row 1.

Transaction B locks Row 2.

Transaction A needs Row 2.

Transaction B needs Row 1.

Neither transaction can continue.

PostgreSQL automatically detects this situation and cancels one transaction.

---

# How Locks Work

Imagine two people editing the same document.

```
User A

↓

Editing Document

↓

Lock Acquired

---------------------

User B

↓

Waiting...

↓

Lock Released

↓

Editing Continues
```

Only one person can safely modify the document at a time.

PostgreSQL works in the same way.

---

# Why This Matters for Support Engineers

Imagine a customer reports:

> "Our application has frozen."

Possible causes include:

- A long-running transaction
- A blocking lock
- A deadlock
- A slow query

A Support Engineer investigates before assuming the database is slow.

---

# Knowledge Check

## 1. What is a lock?

A lock temporarily protects data while PostgreSQL performs an operation.

---

## 2. Why are locks necessary?

They prevent conflicting updates from multiple transactions.

---

## 3. Does MVCC eliminate locks?

No.

MVCC allows concurrent reads, but writes still require locks.

---

## 4. What is blocking?

Blocking occurs when one transaction waits for another transaction to release a lock.

---

## 5. What is a deadlock?

A deadlock occurs when two transactions wait for each other.

PostgreSQL detects deadlocks automatically and cancels one transaction.

---

# Diagram

```text
Transaction A

      │

UPDATE Row

      │

Lock Acquired

      │

──────────────

Transaction B

      │

UPDATE Same Row

      │

Waiting...

      │

Lock Released

      │

Transaction Continues
```

---

# Key Takeaways

- Locks protect data during modifications.
- MVCC allows concurrent reads.
- Writes still require locks.
- Blocking is normal.
- PostgreSQL automatically resolves deadlocks.

---

# Support Engineer Mindset

When a customer says:

> "The database is hanging."

Ask yourself:

- Is another transaction holding a lock?
- Is someone waiting?
- Is there a deadlock?
- Is a transaction open for too long?

Gather evidence before making assumptions.

---

# 🔍 What You'll See in Production

Common support tickets include:

- "My UPDATE never finishes."
- "The application froze."
- "Everything is waiting."
- "The query never completes."

Many of these issues are caused by locks rather than slow queries.

---

# Looking Ahead

Next we'll investigate locks using PostgreSQL system views.

We'll learn:

- `pg_locks`
- `pg_stat_activity`
- Blocking sessions
- Waiting transactions

This is where we'll begin troubleshooting real production locking issues.

---

# 🎯 Interview Takeaway

After completing this lab, I can confidently explain:

- What PostgreSQL locks are.
- Why MVCC does not replace locks.
- What blocking is.
- What a deadlock is.
- How a Support Engineer begins investigating locking issues.