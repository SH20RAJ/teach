# System Design: Distributed ID Generation (Snowflake ID)
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Generating Globally Unique, Ordered Identifiers at Scale

---

## 1. First Understand the Problem

Imagine you are designing **Twitter (X)**.
Every time a user posts a tweet, you must assign a unique `tweet_id` to that record in your database.

Your system is huge: **100,000 tweets are posted every single second**.
Your database is sharded across **10 physical servers**.

```text
Server 1 (Sharded DB)  ──► Wants to generate ID
Server 2 (Sharded DB)  ──► Wants to generate ID
```

How do you generate IDs such that:
1. **Globally Unique:** No two database nodes ever generate the exact same ID.
2. **Roughly Time-Sorted:** Tweet 2 posted *after* Tweet 1 should have a higher ID number (so sorting tweets on homepages is fast without needing heavy index scans).
3. **High Throughput:** Generating an ID must take less than 1 microsecond and require no network coordination.

### Naive Attempts:
- **Attempt A: Database Auto-Increment (`AUTO_INCREMENT`).** You have 10 servers. If all write to one central ticket server to get their next number: this ticket server becomes a single bottleneck and crashes.
- **Attempt B: UUID (Universally Unique Identifier).** UUIDs are 128-bit random strings (e.g., `f81d4fae-7dec-11d0-a765-00a0c91e6bf6`). They are unique and require no coordination.
  - **The Drawbacks:**
    1. They are **not index-friendly**. In MySQL, inserting random UUIDs into a B+ Tree index causes page splits, heavily slowing down writes.
    2. They are **not time-ordered**.
    3. They are **huge** (128-bit strings), consuming valuable storage and indexing memory.

We need a 64-bit integer ID that is unique, sequential, and generates locally on each node.
The standard system design solution is the **Twitter Snowflake Algorithm**.

---

## 2. Imagine the License Plate Analogy

Imagine a massive shipping company with 1,000 delivery trucks nationwide.
Every parcel must be labeled with a unique parcel number.
The drivers do not have internet access to ask a central office for the next parcel number.

How does the company design the label?
They divide the label code into **sections**:

```text
+-----------------------+---------------------+-------------------------+
| Section 1: Timestamp  | Section 2: Truck ID | Section 3: Parcel count |
| (Year-Month-Day-Min)  |  (Unique to Truck)  | (Reset every minute)    |
| e.g. 2026052611       |      Truck 42       |       Parcel #3         |
+-----------------------+---------------------+-------------------------+
```

### The Driver's Action:
- The driver of Truck 42 prints a label. They look at their clock: `2026052611`.
- They append their truck ID: `42`.
- They look at their local parcel counter: `3`.
- The code is: `2026052611-42-3`.

**Think about this:**
- Can Driver 12 in California ever print the same label as Driver 42 in New York? **No**, because the middle section (Truck ID) is different!
- Do they need to call each other? **No**, they work completely offline.
- Are the labels roughly sorted by time? **Yes**, because the first section is the timestamp!

---

## 3. The Snowflake ID Bitwise Layout: Step-by-Step

Let's look at the bit-layout of a 64-bit Snowflake ID on the whiteboard.

A Snowflake ID is a signed 64-bit integer (represented as bits 0 to 63):

```text
 0      1                                  42           52            63
+-+------------------------------------------+------------+------------+
|0|            Timestamp (41 bits)           | Machine ID |  Sequence  |
| |                                          |  (10 bits) |  (12 bits) |
+-+------------------------------------------+------------+------------+
```

### The Breakdown of the 64 Bits:
1. **Sign Bit (1 bit):** Always set to `0` to keep the integer positive.
2. **Timestamp (41 bits):** Stores milliseconds elapsed since a custom epoch (e.g. your company's founding date, not the default Unix 1970 epoch).
   - **Math:** 41 bits can represent $2^{41}$ milliseconds, which gives about **69 years** of unique IDs before overflowing.
3. **Machine ID / Worker ID (10 bits):** Identifies which specific server node is generating the ID.
   - **Math:** 10 bits can represent $2^{10} = 1024$ physical server nodes in your cluster.
4. **Sequence Number (12 bits):** A local counter that increments for every ID generated in the same millisecond. Resets to `0` every time the millisecond clock ticks.
   - **Math:** 12 bits can represent $2^{12} = 4096$ values. This means a single server can generate up to **4,096 unique IDs per millisecond** (over 4 Million IDs per second!).

---

## 4. The Clock Drift Failure Scenario

What happens if a server's clock drifts backward (e.g. Network Time Protocol NTP adjusts the server clock backward by 10 milliseconds)?

```text
Time was 11:20:05.100. Server generated ID with this timestamp.
Clock is adjusted backward to 11:20:05.090.
Server tries to generate a new ID.
The timestamp is 11:20:05.090.
But the server already generated IDs for 11:20:05.100!
This will generate DUPLICATE IDs!
```

### The Solution:
The generator code tracks the `last_timestamp`.
If `current_timestamp < last_timestamp`, the generator detects clock drift.
It immediately rejects ID generation (throws an error) or waits until the system clock catches up to `last_timestamp` before resuming.

---

## 5. Common Confusion

### Confusion: Is a Snowflake ID strictly ordered?
*   **Correction:** No. It is **roughly time-ordered** (monotonically increasing). If Server A and Server B generate IDs in the same millisecond, Server B's ID might have a higher Machine ID bit value, even if Server A's event happened a fraction of a millisecond earlier. For system design indexes, this rough sorting is perfectly fine.

---

## 6. Interview-Safe Definitions

1. **Snowflake ID:** A 64-bit unique identifier generation algorithm originally developed by Twitter that creates roughly time-ordered integers without centralized coordination.
2. **Epoch:** A reference starting point in time used to measure elapsed duration (milliseconds) to maximize the utility of timestamp bits in ID generators.
3. **Clock Drift:** a phenomenon where a computer system's hardware clock operates at a slightly different rate than a reference clock or shifts backward during synchronization events.

---

## 7. Active Recall Questions

1. **Why does inserting random UUIDs into a database index (like MySQL B+ Tree) cause severe write degradation at scale?**
2. **What happens in the Twitter Snowflake algorithm if a server exceeds 4,096 ID requests in a single millisecond?**
3. **How do we assign the 10-bit Machine ID to servers in an auto-scaling cloud cluster? (Hint: ZooKeeper/Consul coordinator).**
