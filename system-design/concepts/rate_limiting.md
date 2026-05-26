# System Design: Rate Limiting (Token vs Leaky Bucket)
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** API Rate Limiter Design and Algorithms

---

## 1. First Understand the Problem

Imagine you run an API endpoint:
`POST /send-sms` (which costs you $0.01 per SMS sent).

A user writes a python script with an infinite loop:
```python
while True:
    send_sms_request()
```

If they run this overnight, your server will:
1. Crash due to resource exhaustion.
2. Generate a bill of thousands of dollars.

To prevent this, you must restrict the number of requests a user can make within a time frame.
For example: **Max 10 requests per minute**.

How do you design a system that enforces this constraint?
Let's study the two classical whiteboard algorithms: **Token Bucket** and **Leaky Bucket**.

---

## 2. Token Bucket: The Arcade Analogy

Imagine a game arcade room.
Inside is a ticket machine.

```text
       [ Token Generator ]
      Generates 1 coin/sec
               │
               ▼
     +───────────────────+
     |   Token Bucket    |  (Max capacity: 5 coins)
     |   (O) (O) (O)     |
     +───────────────────+
               │
   Request requires 1 coin
               │
               ▼
     [ User enters Arcade ]
```

### The Rules:
1. The Arcade machine drops **1 token (coin)** into a bucket every second.
2. The bucket can hold a **maximum of 5 tokens**. If the bucket is full, extra tokens overflow and are discarded.
3. Every time a user request arrives:
   - If there is a token in the bucket, the system removes 1 token and lets the request pass.
   - If the bucket is empty, the request is rejected immediately with HTTP 429 (Too Many Requests).

### The Bursty Traffic Rhythm:
```text
The system is quiet for 10 seconds.
The bucket fills up to its limit (5 tokens).

Suddenly:
5 user requests arrive at the exact same millisecond!
1st request takes 1 token -> ALLOWED
2nd request takes 1 token -> ALLOWED
3rd request takes 1 token -> ALLOWED
4th request takes 1 token -> ALLOWED
5th request takes 1 token -> ALLOWED

Bucket is now empty.
6th request arrives -> REJECTED (HTTP 429).
```
**Key Advantage:** Token Bucket allows **bursts of traffic**. It can handle a short spike up to the maximum capacity of the bucket.

---

## 3. Leaky Bucket: The Funnel Analogy

Now, imagine a water funnel.

```text
       Requests (Water Drops)
       💧  💧💧  💧      💧💧
       ▼   ▼ ▼   ▼      ▼ ▼
     +───────────────────+
     |   \           /   |
     |    \         /    |  Funnel holding water
     |     \       /     |
     +──────\     /──────+
             │   │
             ▼   ▼
       Leaking at a constant rate
       (e.g., 1 drop per second)
```

### The Rules:
1. When requests arrive, they are poured into the funnel (a FIFO queue of fixed capacity).
2. The funnel has a tiny hole at the bottom. Water leaks out at a **constant, steady rate** (e.g., 1 request processed per second).
3. If the funnel is full (queue is full) and more water is poured in, the water overflows. These excess requests are rejected.

### The Bursty Traffic Rhythm:
```text
5 user requests arrive at the exact same millisecond.
All 5 are queued into the funnel.

Funnel drains...
Second 1: 1st request is processed.
Second 2: 2nd request is processed.
Second 3: 3rd request is processed.
...
```
**Key Advantage:** Leaky Bucket guarantees a **smooth, constant flow** of output requests. It completely eliminates bursty traffic spikes.

---

## 4. Token Bucket vs Leaky Bucket Comparison

Let's summarize their differences on the whiteboard:

| Feature | Token Bucket | Leaky Bucket |
| :--- | :--- | :--- |
| **Spike handling** | Allows sudden bursts of requests (up to capacity). | Smooths out bursts; outputs at a constant rate. |
| **Mechanism** | Tokens accumulate; requests consume tokens. | Requests queue up; queue drains at a steady speed. |
| **Storage** | Stores idle tokens for future use. | Stores pending requests in a queue. |
| **Target Use Case** | Good for APIs that need fast response times for occasional spikes (e.g., searches). | Good for downstream databases that need a stable, un-burstable traffic load. |

---

## 5. Common Confusion

### Confusion: Do both algorithms store requests?
*   **Correction:** No! Token Bucket does **not** store requests. If there are no tokens, the request is immediately rejected. It only stores tokens. 
*   Leaky Bucket, however, **stores requests** in a queue. If the queue has space, the request is held there until it is slowly processed.

---

## 6. Interview-Safe Definitions

1. **Rate Limiter:** A control system used to limit the rate of requests sent by a user, client, or IP address to protect servers from overload.
2. **Token Bucket Algorithm:** A rate limiting algorithm that generates tokens at a fixed rate in a bucket. A request is allowed if a token can be drawn from the bucket.
3. **Leaky Bucket Algorithm:** A rate limiting algorithm that stores incoming requests in a queue and processes them at a constant, fixed rate.

---

## 7. Active Recall Questions

1. **Under what scenario would you choose a Token Bucket rate limiter over a Leaky Bucket rate limiter?**
2. **If a Leaky Bucket has a capacity of 10 requests, and leaks 2 requests per second, what happens if 15 requests arrive in 1 millisecond?**
3. **How does Token Bucket handle memory consumption when millions of users are being rate-limited?**
