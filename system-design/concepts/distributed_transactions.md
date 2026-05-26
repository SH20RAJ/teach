# System Design: Distributed Transactions (2PC vs Saga)
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Maintaining Data Consistency across Microservices

---

## 1. First Understand the Problem

Imagine you run an airline booking microservices system.
When a user books a trip, the app must do two operations:
1. **Payment Service:** Charge $500 from the user's bank account (Database A).
2. **Seat Service:** Reserve Seat 12A on Flight 302 (Database B).

If both Databases are in a single MySQL instance, we can run a local SQL transaction:
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE seats SET status = 'reserved' WHERE flight_id = 302 AND seat = '12A';
COMMIT; -- If either fails, database rolls back both!
```

**Now, what is the distributed crisis?**
Your system is a microservices architecture.
- **Payment Database** is PostgreSQL.
- **Seat Database** is MongoDB.
They are on separate physical servers!

```text
[ Payment Service ] ──► PostgreSQL (Charges balance)
[ Seat Service ]    ──► MongoDB (Reserves seat)
```

What happens if:
1. Payment Service successfully charges $500.
2. The network wire snaps, or the Seat Service MongoDB crashes.
3. The seat is **not** reserved, but the user's money is gone!

This is a **Distributed Transaction** problem.
We need a way to guarantee **All-or-Nothing consistency** across completely separate database systems.

Let's study the two main patterns: **Two-Phase Commit (2PC)** and the **Saga Pattern**.

---

## 2. 2-Phase Commit (2PC): The Wedding Analogy

Imagine a traditional wedding ceremony with a Coordinator (the Priest).
The Priest must make sure both the Bride and Groom agree to marry before making it official.

### Phase 1: The Prepare Phase (Voting)
The Coordinator asks:
- To the Bride: "Do you agree?" -> Bride checks her heart and says: "Yes." (Voted: Ready).
- To the Groom: "Do you agree?" -> Groom checks his heart and says: "Yes." (Voted: Ready).

```text
               [ Coordinator ]
               /             \
      Prepare? /               \ Prepare?
              ▼                 ▼
          [ Bride ]         [ Groom ]
```

---

### Phase 2: The Commit Phase (Action)
Since both voted "Yes", the Coordinator declares:
- "I now pronounce you husband and wife." (Commit command is sent to both!).
- Both update their social status permanently.

**What if someone votes "No"?**
If the Groom says: "No, I run away!" (Voted: Abort).
The Coordinator cancels the whole wedding and shouts: "Rollback!"
The Bride walks home. No transaction occurred!

```text
                 [ Coordinator ]
                 /             \
        Rollback! /               \ Rollback!
                ▼                 ▼
            [ Bride ]         [ Groom ]
```

### Technical Trade-offs of 2PC:
- **Pro:** Strong consistency. Databases are guaranteed to be in sync.
- **Con:** **Blocking.** If the Coordinator crashes *after* the Bride votes "Yes" but *before* receiving the final Commit command, the Bride is stuck waiting forever. Her database records remain locked, preventing other queries from executing. This hurts scaling throughput.

---

## 3. The Saga Pattern: The Undo-Chain Analogy

Because 2PC is slow and locks databases, modern high-scale systems use **Eventual Consistency** via the **Saga Pattern**.

Imagine you order a custom dress online.
1. The shop charges your card ($100).
2. The tailor tries to cut the dress, but runs out of silk.
3. The shop does not hold a 2-day locking session.
4. Instead, the shop calls your bank and processes a **Refund** ($100).

In a Saga, each step executes a local transaction. If a step fails, the system executes **Compensating Transactions** (Undo operations) in reverse order to clean up the data.

```text
Normal Flow:       Step 1 (Charge Card)   ──►   Step 2 (Reserve Seat) [FAIL]
                                                        │
Rollback Flow:     Undo Step 1 (Refund Card) ◄──────────┘
```

---

### Saga Architectures: Orchestration vs Choreography

How do Sagas coordinate their steps?

#### Method A: Orchestration (The Conductor)
You build a dedicated service called the **Saga Orchestrator**.
- The Orchestrator acts like a conductor.
- "Payment Service, charge card." -> Payment says: "Done."
- "Seat Service, reserve seat." -> Seat says: "Failed!"
- "Payment Service, run refund." -> Payment says: "Undone."

```text
                 [ Orchestrator ]
                  /     ▲    \
     1. Charge?  /      │     \ 3. Refund? (Undo)
                ▼       │ 2.   ▼
          [ Payment ]   │ Failed! [ Seat ]
                        │
```

#### Method B: Choreography (The Domino Effect)
There is no central manager. Services listen to Event Queues.
- Order Service publishes event: `OrderCreated`.
- Payment Service hears `OrderCreated` -> Charges card -> Publishes event: `PaymentSuccessful`.
- Seat Service hears `PaymentSuccessful` -> Tries to reserve -> Fails -> Publishes event: `SeatReservationFailed`.
- Payment Service hears `SeatReservationFailed` -> Refunds card.

---

## 4. 2PC vs Saga Comparison

Let's write the summary on the whiteboard:

| Feature | Two-Phase Commit (2PC) | Saga Pattern |
| :--- | :--- | :--- |
| **Consistency** | **Strong Consistency** (All nodes updated immediately). | **Eventual Consistency** (Data is temporarily out of sync). |
| **Locks** | Locks database rows until transaction ends (Slow). | No database locks. Steps commit instantly (Fast). |
| **Rollback** | Automatic database engine rollback. | Must write manual "Undo" actions for every step. |
| **Scale** | Hard to scale past a few database nodes. | Highly scalable across hundreds of microservices. |

---

## 5. Common Confusion

### Confusion: Does a Saga rollback restore database logs to their exact prior state?
*   **Correction:** No! In a database rollback (like in 2PC), the row state is physically reverted as if the change never occurred. In a Saga, the original write is *committed* permanently. The compensating transaction is a **new write** that cancels the effect of the previous write (e.g., adding $100 back to an account is a new transaction log entry, not an erase of the original -$100 entry).

---

## 6. Interview-Safe Definitions

1. **Two-Phase Commit (2PC):** A consensus protocol that coordinates all database nodes in a distributed transaction to either commit or abort the transaction collectively.
2. **Saga Pattern:** A design pattern that coordinates a sequence of local transactions across multiple microservices, executing compensating transactions in reverse order if a failure occurs.
3. **Compensating Transaction:** An application-level action executed to revert the effects of a previously committed step in a distributed workflow.

---

## 7. Active Recall Questions

1. **Why is the Saga pattern preferred over 2PC in high-throughput microservices architectures?**
2. **What is the difference between Orchestration and Choreography in Saga pattern coordinate systems?**
3. **What is a "pivot transaction" in a Saga workflow, and how does it determine if a transaction can roll back?**
