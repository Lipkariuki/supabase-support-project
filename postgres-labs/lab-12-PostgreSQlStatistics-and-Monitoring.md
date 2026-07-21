# Lab 12 — PostgreSQL Statistics & Monitoring

> Learning how PostgreSQL tells you what is happening inside the database.

---

⏱ **Estimated Time:** 35–40 minutes

🎯 **Difficulty:** Intermediate

📚 **Prerequisites:**

- Lab 11 — Replication & High Availability
- Lab 10 — Write-Ahead Logging (WAL)
- Lab 08 — Locks & Concurrency
- Lab 06 — EXPLAIN & EXPLAIN ANALYZE

---

## Objective

Imagine a customer reports:

> "Our database became slow this morning."

How do you investigate?

Do you restart PostgreSQL?

Do you guess?

No.

You ask PostgreSQL what it knows.

PostgreSQL continuously collects statistics about:

- Active sessions
- Running queries
- Database activity
- Table usage
- Index usage
- Replication
- Vacuum activity

These statistics help Support Engineers diagnose production issues.

By the end of this lab, you'll understand:

- What PostgreSQL statistics are
- The most important `pg_stat_*` views
- When to use each statistics view
- How Support Engineers investigate production incidents

---

# Key Concepts

## PostgreSQL Statistics

PostgreSQL continuously records information about database activity.

These statistics are exposed through system views that usually begin with:

```text
pg_stat_
```

Think of them as PostgreSQL's dashboard.

---

# pg_stat_activity

This is one of the first places Support Engineers look.

```sql
SELECT *
FROM pg_stat_activity;
```

It shows:

- Connected sessions
- Running queries
- Waiting queries
- Session state
- Transaction start time
- Application name

Example:

```text
PID      State     Query

4211     active    SELECT ...

4224     idle      NULL

4310     active    UPDATE ...
```

---

## When do you use it?

Questions it answers:

- Who is connected?
- Which query is running?
- Which session is blocking?
- Are there long-running transactions?

---

# pg_stat_database

Provides statistics for each database.

```sql
SELECT *
FROM pg_stat_database;
```

Useful metrics include:

- Number of commits
- Number of rollbacks
- Blocks read
- Blocks hit
- Deadlocks
- Temporary files

Example:

```text
Database

↓

Commits

↓

Rollbacks

↓

Cache Hit Ratio
```

---

## When do you use it?

Questions it answers:

- Is the database busy?
- Are many transactions rolling back?
- Is the cache performing well?

---

# pg_stat_all_tables

Shows activity for every table.

```sql
SELECT *
FROM pg_stat_all_tables;
```

Useful information includes:

- Sequential scans
- Index scans
- Inserts
- Updates
- Deletes
- Dead tuples
- Vacuum history

Example:

```text
Table

↓

Seq Scan

↓

Index Scan

↓

Dead Tuples
```

---

## When do you use it?

Questions it answers:

- Which tables are heavily used?
- Which tables contain many dead tuples?
- Which tables need vacuuming?

---

# pg_stat_user_indexes

Shows index usage.

```sql
SELECT *
FROM pg_stat_user_indexes;
```

Useful metrics include:

- Number of index scans
- Index size
- Index usage frequency

---

## When do you use it?

Questions it answers:

- Is an index being used?
- Is an index potentially unnecessary?
- Which indexes receive the most traffic?

---

# pg_stat_replication

Used on the Primary database.

```sql
SELECT *
FROM pg_stat_replication;
```

Shows:

- Connected replicas
- Replication lag
- WAL positions
- Replication state

Example:

```text
Replica

↓

Streaming

↓

Lag

↓

Replay Position
```

---

## When do you use it?

Questions it answers:

- Is replication working?
- Is a replica behind?
- Has WAL stopped streaming?

---

# Why These Views Matter

Imagine these support tickets.

---

Customer:

> "The database is slow."

Start with:

```text
pg_stat_activity
```

---

Customer:

> "Storage keeps growing."

Start with:

```text
pg_stat_all_tables
```

---

Customer:

> "Replication is behind."

Start with:

```text
pg_stat_replication
```

---

Customer:

> "Indexes don't seem to help."

Start with:

```text
pg_stat_user_indexes
```

---

Customer:

> "The database feels busy."

Start with:

```text
pg_stat_database
```

Notice something.

You don't randomly choose a statistics view.

Each one answers a specific type of question.

---

# Why This Matters for Support Engineers

Support Engineers rarely solve problems by guessing.

Instead, they gather evidence.

Think of PostgreSQL statistics as your investigation tools.

Rather than saying:

> "I think replication is broken."

You can verify it.

Rather than saying:

> "Maybe a query is stuck."

You can verify it.

---

# Knowledge Check

## 1. What are PostgreSQL statistics?

Information PostgreSQL collects about database activity.

---

## 2. Which view shows active sessions?

```text
pg_stat_activity
```

---

## 3. Which view helps investigate table activity?

```text
pg_stat_all_tables
```

---

## 4. Which view shows replication status?

```text
pg_stat_replication
```

---

## 5. Why are statistics important?

Because they provide evidence for diagnosing database problems.

---

# Diagram

```text
              PostgreSQL

                    │

        ┌───────────┼───────────┐

        ▼           ▼           ▼

pg_stat_activity  pg_stat_database

        │

        ▼

pg_stat_all_tables

        │

        ▼

pg_stat_user_indexes

        │

        ▼

pg_stat_replication
```

---

# How Everything Connects

```text
Slow Query

      │

      ▼

pg_stat_activity

      │

      ▼

Blocking?

      │

      ▼

Locks

      │

      ▼

Transactions

      │

      ▼

MVCC

      │

      ▼

VACUUM

      │

      ▼

Storage
```

Statistics are the starting point for investigating almost every PostgreSQL issue.

---

# Key Takeaways

- PostgreSQL collects statistics automatically.
- `pg_stat_activity` shows active sessions.
- `pg_stat_database` summarizes database activity.
- `pg_stat_all_tables` shows table usage.
- `pg_stat_user_indexes` tracks index usage.
- `pg_stat_replication` monitors replication health.
- Support Engineers use statistics to gather evidence before making changes.

---

# Support Engineer Mindset

When a customer reports a problem:

Don't ask:

> "What do I think is wrong?"

Ask:

> "Which statistics view can help me verify it?"

Good troubleshooting starts with evidence, not assumptions.

---

# 🔍 What You'll See in Production

Common investigations include:

- Finding long-running queries with `pg_stat_activity`.
- Checking cache efficiency with `pg_stat_database`.
- Identifying tables with many dead tuples.
- Finding unused indexes.
- Monitoring replication lag.
- Confirming whether autovacuum is keeping up.

These statistics views are often the first stop in a production investigation.

---

# Looking Ahead

Next we'll complete the PostgreSQL write lifecycle.

> **Lab 13 — Checkpoints & Background Writer**

You'll learn:

- Dirty pages
- Checkpoints
- Background Writer
- Checkpointer process
- Why PostgreSQL doesn't immediately write every page to disk
- How checkpoints relate to WAL

---

# 🎯 Interview Takeaway

After completing this lab, I can confidently explain:

- What PostgreSQL statistics are.
- The purpose of the main `pg_stat_*` views.
- How to investigate active sessions, table activity, index usage, and replication.
- Why Support Engineers rely on statistics before making changes.