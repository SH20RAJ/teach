# System Design: Database Sharding
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Horizontal Partitioning for Scalable Databases

---

## 1. First Understand the Problem

Imagine you run an online retail store. You have a database table: `Users`.
- Columns: `id`, `name`, `email`, `address`, `purchase_history`.
- Total users: **50 Million**.

Your database runs on a single server machine.
As traffic grows, you face two limits:
1. **Storage Limit:** The hard drive is running out of space.
2. **IOPS/CPU Limit:** Too many parallel reads/writes are causing queries to lock and run slowly.

What do you do?

### Method 1: Vertical Scaling (Scaling Up)
You buy a bigger machine with 128 cores of CPU and 1 TB of RAM.
**The Problem:** This is extremely expensive, and eventually, you will hit a physical hardware limit (the ceiling). You cannot scale indefinitely.

### Method 2: Vertical Partitioning (Splitting Columns)
You split the table columns into two separate database servers:
- **Server 1:** Holds `id`, `name`, `email`.
- **Server 2:** Holds `id`, `address`, `purchase_history`.
**The Problem:** The number of users (50 Million) is still the same. Server 2's storage will still run out due to heavy `purchase_history` blobs.

We need a way to split the **rows** of our table across multiple separate database servers.
This is called **Horizontal Partitioning**, or **Sharding**.

---

## 2. Imagine the Cabinet Scene

Imagine you are a school principal. You have 10,000 student physical records in folders.
You cannot fit them all into one filing cabinet.

What do you do?
You buy 3 filing cabinets!
- **Cabinet 1 (A-I):** Holds records of students whose names start with A to I.
- **Cabinet 2 (J-R):** Holds records of students whose names start with J to R.
- **Cabinet 3 (S-Z):** Holds records of students whose names start with S to Z.

```text
       [ Student Record Lookup ]
                  │
                  ├─ Name starts with A-I ──► [ Cabinet 1 ]
                  ├─ Name starts with J-R ──► [ Cabinet 2 ]
                  └─ Name starts with S-Z ──► [ Cabinet 3 ]
```

### Mapping to Database Sharding:
- **Filing Cabinets** = **Physical Database Servers (Shards)**.
- **Letter Ranges (A-I, J-R, S-Z)** = **Sharding Key** (the criteria used to route queries).
- **Sorting process** = **Routing Logic** executed by the application or a database proxy.

Now, instead of one database server holding 50 Million rows, you have 5 database shards, each holding 10 Million rows. Reads and writes are now distributed, multiplying your database's total capacity!

---

## 3. How Queries Walk: Step-by-Step

Let's look at how the application queries the sharded system.

### Scenario A: Query with the Sharding Key
The app wants to load details for `user_id = 15`.
Our sharding key is `user_id`, and the routing function is `Shard = user_id % 3`.

```text
Query: "SELECT * FROM Users WHERE id = 15"

Application calculates: 15 % 3 = Shard 0.
Application makes a connection directly to Shard 0.
Shard 0 returns user 15.
```
This is extremely fast! Only one node is queried.

### Scenario B: Query WITHOUT the Sharding Key
The app wants to find a user by their email address:
`SELECT * FROM Users WHERE email = "bob@example.com"`

```text
Application does not know which Shard contains this email because sharding is based on id.

Now, what must the application do?
It must query Shard 0.
It must query Shard 1.
It must query Shard 2.

This is called a Scatter-Gather Query.
It merges results from all shards and returns to the client.
```
**Warning:** Scatter-Gather queries are very slow and resource-heavy. If you shard your database, you must choose your sharding key wisely based on your application's most frequent queries!

---

## 4. Database Sharding Architecture

```text
                     Client Request
                           │
                           ▼
                 +───────────────────+
                 |    API Router     |
                 |  (Calculates Key) |
                 +───────────────────+
                  /        |        \
                 /         |         \
                ▼          ▼          ▼
           [ Shard 0 ] [ Shard 1 ] [ Shard 2 ]
            IDs 0-10M   IDs 10-20M  IDs 20-30M
```

---

## 5. Common Confusion

### Sharding vs Replication
*   **Replication:** Copying the **entire** database to other servers. (Every replica has 100% of the data. Good for read scaling and high availability).
*   **Sharding:** Splitting the database rows into **separate subsets**. (No single server has all the data. Shard A has IDs 1-100, Shard B has IDs 101-200. Good for write scaling and storage scaling).

---

## 6. Interview-Safe Definitions

1. **Database Sharding:** A database design pattern that splits a single logical dataset horizontally across multiple physical database instances (shards).
2. **Sharding Key:** A column or combination of columns in a database table used to determine which shard a particular row of data should be stored in.
3. **Scatter-Gather:** A querying technique where the routing layer broadcasts a query to all database shards, collects their responses, and aggregates them before sending them back to the caller.

---

## 7. Active Recall Questions

1. **What are the main drawbacks of database sharding compared to a single-instance database?**
2. **If you have a table where 90% of your queries filter by `tenant_id`, what should be your sharding key?**
3. **How does consistent hashing fit into the design of a sharded database routing layer?**
