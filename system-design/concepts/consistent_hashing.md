# System Design: Consistent Hashing
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Consistent Hashing for Distributed Caching

---

## 1. First Understand the Problem

Imagine you have a website with millions of users. 
To serve requests fast, you cache user sessions.
You have 3 Cache Servers:
- **Server A**
- **Server B**
- **Server C**

When a request comes for a user (e.g., `user_id = 42`), how do you decide which cache server to look in?

### Method 1: Simple Modulo Hashing
We hash the key and take modulo of number of servers ($N = 3$):
`Server Index = Hash(Key) % N`

Let's see the math:
- `Hash("User_X") = 10` ==> `10 % 3 = 1` (Server B)
- `Hash("User_Y") = 11` ==> `11 % 3 = 2` (Server C)
- `Hash("User_Z") = 12` ==> `12 % 3 = 0` (Server A)

**Now, what is the problem?**
Imagine your website gets huge traffic. Server C crashes!
```text
[Server A]          [Server B]          [Server C]
 (Alive)             (Alive)             (CRASHED!)
```

Now, number of servers changes from $N = 3$ to $N = 2$.
Let's calculate server index again for the same users:
- `Hash("User_X") = 10` ==> `10 % 2 = 0` (Server A - Was Server B!)
- `Hash("User_Y") = 11` ==> `11 % 2 = 1` (Server B - Was Server C!)
- `Hash("User_Z") = 12` ==> `12 % 2 = 0` (Server A - Was Server A!)

**Look at this disaster!**
Almost every key now maps to a **different** server. This is called a **Cache Storm (or Cache Stampede)**. 
Because $N$ changed:
1. All your previous cache entries become useless.
2. The client asks for keys, cache is not found (Cache Miss).
3. Millions of requests hit the backend Database directly.
4. The database gets overloaded and crashes!

We need a hashing method where adding or removing a server affects **only** a tiny fraction of keys.
That method is **Consistent Hashing**.

---

## 2. Imagine the Ring Scene

Imagine a circular track (like a running track or a clock). Let's call it the **Hash Ring**.

The ring has numbers from `0` to $2^{32} - 1$.

```text
               0
           . - ~ - .
       .               .
     .                   .
    .                     .
    .                     .
     .                   .
       .               .
           . - ~ - .
```

Now, we do two things:
1. **Map Servers to the Ring:** We hash the server name (e.g., `Hash("Server_A")`) and place the server at that number on the ring.
2. **Map Keys to the Ring:** We hash the key (e.g., `Hash("User_X")`) and place the key at that number on the ring.

How does a key find its server?
**Rule:** Walk clockwise from the key's position on the ring. The first server you hit is the owner of that key!

```text
                       0 (Start)
                   . - ~ - .
               .     [Server A] .
      [Key Z] .                   .
            .                       .
            .                       . [Key X]
             .                     .
               .   [Server B]  .
                   . - ~ - .
```

- **Key X** walks clockwise ==> Hits **Server B**.
- **Key Z** walks clockwise ==> Hits **Server A**.

---

## 3. What Happens on Node Removal?

Let's watch the movie scene of Server B crashing:

```text
Server B goes offline.

A request comes for Key X.
Key X is placed on the ring.
Key X walks clockwise.
It passes the spot where Server B used to be...
It keeps walking...
It hits Server A!
```

**Now, think:**
Only the keys that belonged to Server B (like Key X) had to move to Server A.
Did Key Z move? No! Key Z still walks clockwise and hits Server A directly.
Adding or removing a server only reshuffles $1/N$ of the keys! The rest of the cache remains 100% warm.

---

## 4. Consistent Hashing System Architecture

```text
       Incoming Request (Key: "User_12")
                     │
                     ▼
         +-----------------------+
         |   Consistent Hashing  |
         |     Routing Ring      |
         +-----------------------+
          /          |          \
         /           |           \
        ▼            ▼            ▼
   [Cache A]    [Cache B]    [Cache C]
    (Hash: 120)  (Hash: 540)  (Hash: 980)
```

### The Virtual Node Concept (Scaling Fairness)
If you only place 3 servers on the ring, they might be clustered together:
```text
Server A is at position 10.
Server B is at position 12.
Server C is at position 990.
```
In this case, Server C owns 98% of the ring! Server C will handle almost all traffic and melt down.

**The Solution:** **Virtual Nodes (Vnodes)**.
Instead of placing `Server A` once, we place it 100 times using different names: `Server_A#1`, `Server_A#2`, `Server_A#3`...
This distributes the server presence uniformly across the ring, ensuring an even load distribution.

---

## 5. Common Confusion

### Confusion: Does consistent hashing mean data is replicated on all nodes?
*   **Correction:** No. Consistent hashing is simply a **routing mechanism** to locate which single node holds (or should hold) a key. Replication is a separate layer (e.g., storing a key on node $X$ and also on the next two clockwise nodes $X+1$ and $X+2$ on the ring).

---

## 6. Interview-Safe Definitions

1. **Consistent Hashing:** A special kind of hashing technique such that when a hash table is resized, only $K/N$ keys need to be remapped on average, where $K$ is the number of keys and $N$ is the number of slots (servers).
2. **Hash Ring:** A logical representation of the hash space as a circle, where both server nodes and cache keys are mapped using the same hash function.
3. **Virtual Nodes (Vnodes):** Replications of physical server nodes mapped at multiple random positions on the hash ring to prevent load distribution imbalance (hotspots).

---

## 7. Active Recall Questions

1. **Why does traditional modulo hashing (`hash(key) % N`) fail when a caching cluster auto-scales?**
2. **Explain how virtual nodes solve the "hotspot" or traffic imbalance problem in consistent hashing.**
3. **If a key is placed at coordinate 450, and servers are at 100, 300, and 600, which server receives the write request?**
