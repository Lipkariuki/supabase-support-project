# Lab 11 — Replication & High Availability

> Understanding how PostgreSQL keeps multiple database servers synchronized using Write-Ahead Logging (WAL).

---

⏱ **Estimated Time:** 30–35 minutes

🎯 **Difficulty:** Intermediate

📚 **Prerequisites:**

- Lab 10 — Write-Ahead Logging (WAL)
- Lab 09 — Transactions
- Lab 02 — MVCC

---

## Objective

Imagine your production database suddenly crashes.

Do users lose access?

Do you restore yesterday's backup?

Or does another database immediately take over?

This is where **Replication** and **High Availability (HA)** come in.

By the end of this lab, you'll understand:

- What replication is
- Why organizations use replicas
- How WAL makes replication possible
- What replication lag means
- What failover is
- Why Support Engineers monitor replication health

---

# Key Concepts

## Primary Database

The **Primary** database is the server that accepts read and write operations.

Every INSERT, UPDATE and DELETE happens here.

```
Application

↓

Primary Database
```

---

## Replica Database

A **Replica** is another PostgreSQL server that receives changes from the Primary.

Normally, replicas are read-only.

```
Application

↓

Primary

↓

Replica
```

The goal is for the replica to stay as close as possible to the primary.

---

## Why Use Replication?

Organizations replicate databases for several reasons:

- High Availability
- Disaster Recovery
- Read Scaling
- Backups
- Reporting

Instead of sending every query to one server:

```
Users

↓

Primary
```

Traffic can be distributed:

```
Writes

↓

Primary

↓

Replicas

↓

Read Queries
```

---

## How Replication Works

This is where WAL becomes important.

Whenever the Primary changes data:

```
UPDATE

↓

Generate WAL

↓

Write WAL

↓

Send WAL

↓

Replica Receives WAL

↓

Replay WAL

↓

Replica Updated
```

Notice something.

The Primary is **not** sending table pages.

It is sending WAL records.

---

## Streaming Replication

The most common PostgreSQL replication method is **Streaming Replication**.

Instead of waiting for large batches of changes:

```
Primary

↓

Continuous WAL Stream

↓

Replica
```

The replica continuously receives new WAL records.

---

## Replication Lag

The replica is usually a few milliseconds or seconds behind.

This delay is called **Replication Lag**.

```
Primary

10:00:01

↓

Replica

10:00:03
```

Small lag is normal.

Large lag may indicate a problem.

---

## Causes of Replication Lag

Common causes include:

- Slow network
- Heavy write workload
- Slow replica hardware
- Large WAL generation
- Long-running queries on the replica

Support Engineers often investigate replication lag before assuming replication is broken.

---

## Failover

Imagine the Primary server crashes.

```
Primary

❌ Offline
```

Instead of waiting for repairs:

```
Replica

↓

Promoted

↓

New Primary
```

This process is called **Failover**.

Applications reconnect to the promoted replica.

---

## High Availability (HA)

High Availability means minimizing downtime.

Instead of:

```
Primary Crashes

↓

Application Offline
```

You aim for:

```
Primary Crashes

↓

Replica Promoted

↓

Application Continues
```

Replication is the foundation of High Availability.

---

# Why This Matters for Support Engineers

Imagine a customer reports:

> "Our reporting dashboard shows old data."

Possible questions include:

- Is the dashboard reading from a replica?
- How much replication lag exists?
- Is WAL streaming correctly?
- Has the replica stopped replaying WAL?

Understanding replication helps explain these issues.

---

# Knowledge Check

## 1. What is a Primary database?

The server that accepts read and write operations.

---

## 2. What is a Replica?

A secondary PostgreSQL server that receives changes from the Primary.

---

## 3. What makes replication possible?

Write-Ahead Logging (WAL).

---

## 4. What is Replication Lag?

The delay between changes being committed on the Primary and applied on the Replica.

---

## 5. What is Failover?

Promoting a replica to become the new Primary after the original Primary fails.

---

# Diagram

```text
                Application
                     │
          ┌──────────┴──────────┐
          │                     │
      Read/Write            Read Only
          │                     │
          ▼                     ▼
     Primary Database -----> Replica Database
             │
             ▼
      Generate WAL
             │
             ▼
       Stream WAL
             │
             ▼
      Replay Changes
```

---

# How Everything Connects

```text
SQL Query

      │

      ▼

Transaction

      │

      ▼

MVCC

      │

      ▼

Generate WAL

      │

      ▼

COMMIT

      │

      ▼

Stream WAL

      │

      ▼

Replica Replays WAL

      │

      ▼

High Availability
```

Replication builds directly on the concepts you've already learned.

---

# Key Takeaways

- A Primary database handles writes.
- Replicas receive changes from the Primary.
- PostgreSQL uses WAL for replication.
- Streaming Replication continuously sends WAL records.
- Replication Lag measures how far behind a replica is.
- Failover promotes a replica when the Primary fails.

---

# Support Engineer Mindset

When investigating replication issues, ask:

- Is WAL being generated?
- Is WAL reaching the replica?
- Is the replica replaying WAL?
- What is the current replication lag?
- Is failover configured correctly?

Replication problems are often communication or resource issues rather than database corruption.

---

# 🔍 What You'll See in Production

Common replication-related support cases include:

- Replica falling behind the Primary.
- Reporting dashboards showing stale data.
- Replication stopping after a network interruption.
- Replica disk filling because WAL cannot be replayed.
- Planned failover during maintenance.
- Emergency failover after infrastructure failure.

---

# Looking Ahead

Next we'll learn how PostgreSQL keeps track of everything happening inside the database.

> **Lab 12 — PostgreSQL Statistics & Monitoring**

You'll learn:

- pg_stat_activity
- pg_stat_database
- pg_stat_all_tables
- pg_stat_user_indexes
- pg_stat_replication
- How Support Engineers diagnose production issues using PostgreSQL statistics.

---

# 🎯 Interview Takeaway

After completing this lab, I can confidently explain:

- What Primary and Replica databases are.
- How Streaming Replication works.
- Why WAL enables replication.
- What Replication Lag means.
- How Failover supports High Availability.
- Common replication issues Support Engineers investigate.