# Lab 10 — Write-Ahead Logging (WAL)

> Understanding how PostgreSQL protects committed data and recovers after crashes.

---

⏱ **Estimated Time:** 25–30 minutes

🎯 **Difficulty:** Intermediate

📚 **Prerequisites:**

- Lab 09 — Transactions
- Lab 08 — Locks & Concurrency
- Lab 02 — MVCC & Dead Tuples

---

## Objective

Imagine a customer updates an important record.

```sql
UPDATE accounts
SET balance = 15000
WHERE id = 5;
```

Suddenly...

💥 The database server loses power.

How does PostgreSQL know which changes were committed?

How does it recover without corrupting data?

The answer is **Write-Ahead Logging (WAL).**

By the end of this lab, you'll understand:

- What WAL is
- Why PostgreSQL writes to WAL before updating data pages
- How crash recovery works
- Why WAL is essential for replication and backups

---

# Key Concepts

## Write-Ahead Log (WAL)

WAL stands for **Write-Ahead Log**.

It is a sequential log that records every database change **before** PostgreSQL writes the actual table pages to disk.

The important word is:

**Ahead**

PostgreSQL writes the log first.

The table pages are updated later.

---

## Why WAL Exists

Without WAL:

```
UPDATE Row

↓

Server Crashes

↓

Data Lost
```

With WAL:

```
UPDATE Row

↓

Write WAL

↓

Server Crashes

↓

Replay WAL

↓

Data Recovered
```

WAL allows PostgreSQL to recover committed transactions after an unexpected shutdown.

---

## Sequential Writes

Writing directly to table pages involves random disk access.

Instead, PostgreSQL first appends changes to the WAL.

```
WAL Record

↓

Append

↓

Append

↓

Append
```

Sequential writes are much faster than random writes.

This improves overall database performance.

---

## Crash Recovery

Suppose the server crashes immediately after a transaction commits.

The table page might not yet contain the latest data.

When PostgreSQL starts again:

```
Read WAL

↓

Replay Changes

↓

Database Restored
```

This process is called **Crash Recovery**.

---

## WAL and COMMIT

A transaction is not considered safely committed until its WAL records have been written to disk.

Example:

```
BEGIN

↓

UPDATE

↓

Write WAL

↓

COMMIT

↓

Transaction Complete
```

This guarantees durability.

---

## WAL and Replication

WAL is also used to keep replica databases synchronized.

```
Primary Database

↓

Generate WAL

↓

Replica Receives WAL

↓

Replay WAL

↓

Replica Updated
```

Streaming Replication depends on WAL.

---

## WAL Growth

Sometimes customers report:

> "Our WAL directory suddenly grew."

Possible reasons include:

- Large UPDATE operations
- Large DELETE operations
- Bulk INSERTs
- Long-running transactions
- Replication lag
- Delayed checkpoints

WAL growth is not automatically a problem.

It is something to investigate.

---

# Why This Matters for Support Engineers

Imagine a customer reports:

> "The database crashed."

Questions a Support Engineer might ask include:

- Did crash recovery complete successfully?
- Is WAL being generated normally?
- Has WAL grown unexpectedly?
- Is replication consuming WAL correctly?
- Are long-running transactions delaying WAL cleanup?

Understanding WAL helps explain many production issues.

---

# Knowledge Check

## 1. What is WAL?

A Write-Ahead Log that records database changes before table pages are written.

---

## 2. Why does PostgreSQL write WAL first?

To ensure committed changes can be recovered after a crash.

---

## 3. What happens during crash recovery?

PostgreSQL replays WAL records to restore committed transactions.

---

## 4. Why are WAL writes fast?

Because PostgreSQL appends records sequentially instead of performing many random writes.

---

## 5. Besides crash recovery, what else uses WAL?

- Streaming Replication
- Backup Recovery
- Point-in-Time Recovery (PITR)

---

# Diagram

```text
SQL Query

     │

     ▼

Generate WAL Record

     │

     ▼

Write WAL to Disk

     │

     ▼

COMMIT

     │

     ▼

Later...

Write Data Page
```

---

# How Everything Connects

```text
Transaction

      │

      ▼

UPDATE Row

      │

      ▼

MVCC Creates New Tuple

      │

      ▼

Old Tuple Becomes Dead

      │

      ▼

WAL Records The Change

      │

      ▼

COMMIT

      │

      ▼

VACUUM Cleans Old Tuples Later
```

Every concept we've learned so far connects through the lifecycle of a database update.

---

# Key Takeaways

- WAL stands for Write-Ahead Log.
- PostgreSQL writes WAL before updating table pages.
- WAL enables crash recovery.
- WAL powers streaming replication.
- Sequential WAL writes improve performance.
- Unexpected WAL growth should be investigated rather than assumed to be a problem.

---

# Support Engineer Mindset

When investigating a PostgreSQL incident, don't only examine the tables.

Also investigate:

- WAL generation
- Replication health
- Crash recovery
- Checkpoints
- Long-running transactions

Support Engineers use WAL as an important source of evidence during production investigations.

---

# 🔍 What You'll See in Production

Common WAL-related support issues include:

- WAL disk usage growing unexpectedly.
- Replication falling behind.
- Long-running transactions preventing WAL cleanup.
- Recovery after an unexpected database restart.
- Backup and recovery operations using WAL.

---

# Looking Ahead

Next we'll learn how PostgreSQL keeps multiple database servers synchronized.

> **Lab 11 — Replication & High Availability**

You'll learn:

- Streaming Replication
- Primary and Replica databases
- Replication Lag
- Failover
- High Availability
- Why WAL makes replication possible

---

# 🎯 Interview Takeaway

After completing this lab, I can confidently explain:

- What Write-Ahead Logging (WAL) is.
- Why PostgreSQL writes WAL before table pages.
- How WAL enables crash recovery.
- Why WAL is the foundation of replication.
- Why Support Engineers investigate WAL during production incidents.