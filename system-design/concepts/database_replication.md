# System Design: Database Replication
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** High Availability, Read Scaling & Data Redundancy

---

## 1. First Understand the Problem

Imagine you run a database server: **Node A**.
Every user write and read request hits Node A.

```text
User 1 (Write) ───► [ Node A ] ◄─── User 2 (Read)
```

Now, what are the failures?
1. **The Server Down Problem:** Node A's power supply fails. Your entire database goes offline. Your app crashes.
2. **The Read Capacity Bottleneck:** 10,000 users try to read articles at the same time. Node A's CPU spikes to 100% trying to query index files, causing queries to pile up.

How do we solve this?
We buy more database machines!
- **Node A**
- **Node B**
- **Node C**

But a database is not like a stateless web server. A database holds **state (data)**.
If a user writes data to Node A, how do Node B and Node C get copies of that data so they can serve reads accurately?

This process of keeping multiple copies of data on different machines is called **Database Replication**.

---

## 2. Imagine the Office Scribe Analogy

Let's look at the three main replication topologies.

### Scene 1: Single-Leader Replication (Leader-Follower)
Imagine a corporate office. There is one Manager (Leader) and two Assistants (Followers).
- **Rule:** Only the Manager can make updates to the master file (Writes).
- When the Manager edits a line, they shout to the Assistants: "Copy this line into your notebooks!" (Replication).
- Customers can call any of the Assistants to ask what is in the notebook (Reads).

```text
User 1 (Write) ──► [ Leader Node ] ── (Replication Log) ──► [ Follower Node 1 ] ◄── User 2 (Read)
                                                          └─► [ Follower Node 2 ] ◄── User 3 (Read)
```
- **Pro:** Simple. No conflict writes because only the leader processes writes.
- **Con:** If the Leader crashes, you must hold an election to choose a new Leader, which can cause writing downtime.

---

### Scene 2: Multi-Leader Replication (Active-Active)
Imagine a global company with offices in New York and London.
- Both NY and London have Managers. Both can process updates locally.
- At the end of the day, they email each other to merge their updates.
- **Problem:** NY Manager changes a client's status to "VIP". London Manager changes the *same* client's status to "Inactive" at the exact same minute.
- When they merge, who wins? This is called a **Write Conflict**. Resolving conflicts is highly complex.

---

### Scene 3: Leaderless Replication (Dynamo-Style)
Imagine a committee of 3 members. There is no leader.
- When a client wants to write data, they send the write request to **all 3 members** directly.
- The client waits until at least **2 out of 3 members** say: "Got it!" (Write Quorum, $W=2$).
- When a client reads, they ask all 3 members. If 2 members say $X=5$ and 1 member says $X=4$ (due to replication delay), the client believes the majority vote ($X=5$) and updates the lagging member. (Read Quorum, $R=2$).

```text
                    [ Client Write ]
                     /     |     \
                    ▼      ▼      ▼
                 [Node A] [Node B] [Node C]
```
- **Pro:** High tolerance to failures. Even if 1 node crashes, writes and reads work perfectly without node elections.
- **Con:** App must handle conflicting read version stamps (e.g. vector clocks).

---

## 3. Replication Lag: The Timeline Problem

What happens if you use Single-Leader Replication and the connection between the Leader and Followers is slow?

Let's watch this timeline scene:
```text
Time 0: User updates their profile photo from "Photo A" to "Photo B".
Time 1: The update is written to the Leader.
Time 2: The replication network is congested. Follower 1 has not received the update yet.
Time 3: User refreshes their page. The page reads their photo from Follower 1.
Time 4: User sees "Photo A" again! They think: "What? My update was lost!"
Time 5: Follower 1 finally receives the copy.
Time 6: User refreshes again. They see "Photo B".
```
This is called **Read-Your-Own-Writes inconsistency**.
To fix this: If a user modifies data, the application must read *that specific user's data* from the Leader directly for the next 10 seconds, while routing other users' reads to the Followers.

---

## 4. Common Failure: Split-Brain

In a Single-Leader system, if the Follower nodes think the Leader is dead (due to a network partition) but the Leader is actually alive and isolated:
1. The Followers elect a new Leader.
2. Now you have **two Leaders** accepting writes!
3. When the network heals, the data is corrupted beyond repair.
This disaster is called **Split-Brain**.

```text
      (Network Cut)
[ Leader A ]  X  [ Leader B ]
(Old Leader)      (Newly Elected)
```
**Solution:** Nodes must verify consensus before accepting writes. They must obtain a quorum (majority vote) from the cluster to prove they are the legitimate leader.

---

## 5. Common Confusion

### Confusion: Is replication a backup?
*   **Correction:** No! Replication is for **High Availability**. If a developer accidentally runs a query: `DELETE FROM Users;` on the Leader, the replication system will faithfully delete that data from all followers within milliseconds. A backup is a point-in-time snapshot stored safely offline.

---

## 6. Interview-Safe Definitions

1. **Replication Lag:** The delay in time between a write operation succeeding on a leader database node and it being successfully copied to a follower node.
2. **Quorum:** The minimum number of database nodes that must participate in a database write ($W$) or read ($R$) operation to guarantee consistency in leaderless replication.
3. **Split-Brain:** A state in clustered database systems where network partitioning causes different nodes to elect multiple leaders, resulting in conflicting writes and data corruption.

---

## 7. Active Recall Questions

1. **If a system has $N=3$ replica nodes, and you require $W=2$ and $R=2$, why are you guaranteed to always read the latest write?**
2. **What is the difference between synchronous replication and asynchronous replication? What is the latency trade-off?**
3. **Explain how "Split-Brain" occurs in a distributed cluster and how consensus algorithms like Raft prevent it.**
