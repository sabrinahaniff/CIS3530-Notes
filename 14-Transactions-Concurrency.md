# Chapter 14: Transactions & Concurrency Control

[[README|← Back to Index]]

---

##  What is a Transaction?

A **transaction** is a logical unit of work that accesses and possibly modifies database contents.

> [!abstract] Transaction Definition
> A sequence of operations that must be executed **atomically** — either all complete successfully or none do.

**Example**:
```sql
BEGIN TRANSACTION;
    UPDATE Account SET Balance = Balance - 100 WHERE ID = 1;
    UPDATE Account SET Balance = Balance + 100 WHERE ID = 2;
COMMIT;
```

---

## ACID Properties

### A - Atomicity

**"All or Nothing"**

- Either **all** operations in a transaction complete successfully, or **none** do
- If a transaction fails midway, the database **rolls back** to the state before the transaction started

**Example**: Transferring money between accounts — either both accounts are updated or neither is.

---

### C - Consistency

**"Rules Stay True"**

- A transaction takes the database from one **consistent state** to another
- All integrity constraints (PKs, FKs, CHECK constraints) must remain satisfied

**Example**: After transferring $100, the total money in the system remains the same.

---

### I - Isolation

**"Don't Step on Others"**

- Concurrent transactions should not interfere with each other
- Each transaction should execute as if it's **alone** (no concurrent modifications visible)

**Example**: Two users booking the last seat on a flight shouldn't both succeed.

---

### D - Durability

**"Once Committed, It Sticks"**

- After a transaction **commits**, its changes are **permanent** (even if the system crashes)
- Achieved through **Write-Ahead Logging (WAL)**

**Example**: After "ORDER CONFIRMED" appears, your purchase is safe even if the server crashes.

---

## Concurrency Problems

### 1. Lost Update

**Problem**: Two transactions read the same value, both update it, and the **last write wins** (first update is lost).

**Example**:
```
Initial: X = 200

T1: Read(X)    → 200
T2: Read(X)    → 200
T1: X = X - 50 → 150
T2: X = X * 1.1 → 220
T1: Write(X)   → X = 150
T2: Write(X)   → X = 220  ← T1's change is LOST!
```

**Result**: T1's change (−50) is overwritten by T2.

---

### 2. Dirty Read (Uncommitted Dependency)

**Problem**: T2 reads a value written by T1 **before T1 commits**, then T1 **rolls back**.

**Example**:
```
Initial: X = 100

T1: Write(X = 200)  (not committed yet)
T2: Read(X)         → 200
T1: ROLLBACK        → X back to 100
T2: Commits         → Used X = 200 (which never existed!)
```

**Result**: T2 used a value that was **never supposed to exist**.

---

### 3. Unrepeatable Read (Inconsistent Analysis)

**Problem**: T1 reads a value twice, but T2 modifies it in between, so T1 sees **different values**.

**Example**:
```
T1: Read(X)         → 100
T2: Write(X = 200)
T2: COMMIT
T1: Read(X)         → 200  ← Different from before!
```

**Result**: T1's two reads are inconsistent.

---

### 4. Phantom Read

**Problem**: T1 executes a query, T2 inserts/deletes rows that match the query, T1 re-executes and sees **different rows**.

**Example**:
```
T1: SELECT COUNT(*) WHERE Age > 30  → 5 rows
T2: INSERT INTO Person VALUES (35, 'Alice')
T2: COMMIT
T1: SELECT COUNT(*) WHERE Age > 30  → 6 rows  ← Phantom row!
```

---

##  Locking Protocols

### Two-Phase Locking (2PL)

> [!abstract] 2PL Rule
> A transaction must acquire **all locks** before releasing **any lock**.

**Two Phases**:
1. **Growing Phase**: Acquire locks (cannot release any)
2. **Shrinking Phase**: Release locks (cannot acquire any)

**Guarantees**: **Conflict serializability** (prevents anomalies)

---

### Strict Two-Phase Locking (Strict 2PL)

> [!important] Strict 2PL Rule
> Hold **all locks** (especially **X-locks**) until the transaction **commits or aborts**.

**Why?**: Prevents **cascading aborts** and ensures **recoverability**.

**Example**:
```
T1: Lock(X)
T1: Write(X)
T1: [... more operations ...]
T1: COMMIT       ← Unlock(X) happens here
```

---

## Lock Types

### Shared Lock (S-Lock)

**Purpose**: **Read-only** access  
**Compatible with**: Other S-locks  
**Conflicts with**: X-locks

**Example**: Multiple users can read the same data simultaneously.

---

### Exclusive Lock (X-Lock)

**Purpose**: **Read + Write** access  
**Compatible with**: Nothing (not even other X-locks)  
**Conflicts with**: Everything (S-locks and X-locks)

**Example**: Only one user can modify data at a time.

---

### Lock Compatibility Matrix

|       | S   | X   |
| ----- | --- | --- |
| **S** | Yes | No  |
| **X** | No  | No  |

---

## Multiple Granularity Locking

Allows locking at different levels: **Database → Table → Page → Row**

### Intention Locks

**Purpose**: Signal intent to lock at a finer granularity

| Lock | Meaning |
|------|---------|
| **IS** | Intention to acquire **S-locks** on child nodes |
| **IX** | Intention to acquire **X-locks** on child nodes |
| **SIX** | Hold **S-lock** on table + **IX** on some rows |

### Lock Compatibility Matrix (Extended)

|         | IS  | IX  | S   | SIX | X   |
| ------- | --- | --- | --- | --- | --- |
| **IS**  | Yes | Yes | Yes | Yes | No  |
| **IX**  | Yes | Yes | No  | No  | No  |
| **S**   | Yes | No  | Yes | No  | No  |
| **SIX** | Yes | No  | No  | No  | No  |
| **X**   | No  | No  | No  | No  | No  |

---

##  Deadlocks

### What is a Deadlock?

**Definition**: Two or more transactions are **waiting for each other** to release locks, creating a **cycle**.

**Example**:
```
T1: Lock(X)
T2: Lock(Y)
T1: Wait for Lock(Y)  ← Waiting for T2
T2: Wait for Lock(X)  ← Waiting for T1
```

**Result**: Both transactions are **frozen forever**.

---

### Deadlock Detection

**Waits-For Graph**:
- **Nodes**: Transactions
- **Edge T1 → T2**: T1 is waiting for a lock held by T2

**Cycle** in the graph → **Deadlock detected**

**Solution**: **Abort** one transaction to break the cycle.

---

### Deadlock Prevention (Timestamp-Based)

#### Wait-Die (Non-Preemptive)

> [!info] "Polite Senior"
> - **Old waits for Young**: Senior politely waits
> - **Young waits for Old**: Junior dies (aborts) immediately

**Example**:
- T1 (timestamp 5) wants lock held by T2 (timestamp 10) → T1 **waits**
- T2 (timestamp 10) wants lock held by T1 (timestamp 5) → T2 **dies**

---

#### Wound-Wait (Preemptive)

> [!info] "Aggressive Senior"
> - **Old waits for Young**: Senior **wounds** (aborts) the junior
> - **Young waits for Old**: Junior **waits** for senior

**Example**:
- T1 (timestamp 5) wants lock held by T2 (timestamp 10) → T1 **wounds T2** (T2 aborts)
- T2 (timestamp 10) wants lock held by T1 (timestamp 5) → T2 **waits**

---

##  Serializability

### Conflict Serializability

Two operations **conflict** if:
1. They access the **same item**
2. At least one is a **Write**
3. They belong to **different transactions**

**Conflict Types**:
- Read-Write (RW)
- Write-Read (WR)
- Write-Write (WW)

---

### Precedence Graph

**Nodes**: Transactions  
**Edge T1 → T2**: T1 must execute before T2 (due to a conflict)

**Test**: If the graph has a **cycle** → **NOT conflict serializable**

---

### Example

**Schedule**:
```
T1: Read(A)
T2: Read(A)
T1: Write(A)  ← Conflict with T2's Read(A) → T2 → T1
T2: Write(A)  ← Conflict with T1's Write(A) → T1 → T2
```

**Precedence Graph**:
```
T1 → T2 → T1  (Cycle!)
```

**Result**: **NOT conflict serializable**

---

## Practice Problems

### Problem 1: Identify the Problem

> [!question]
> What concurrency problem does this exhibit?
> 
> ```
> T1: Read(X = 100)
> T2: Read(X = 100)
> T1: X = X - 50; Write(X = 50)
> T2: X = X + 20; Write(X = 120)
> ```

<details>
<summary>Answer</summary>

**Lost Update**: T1's update (−50) is overwritten by T2's update.

**Correct final value should be**: 100 − 50 + 20 = **70**  
**Actual final value**: **120** (T1's change is lost)

</details>

---

### Problem 2: Lock Compatibility

> [!question]
> Can T1 hold an S-lock on X while T2 holds an X-lock on X?

<details>
<summary>Answer</summary>

**No**. S-lock and X-lock are **incompatible** (they conflict).

T2 must wait until T1 releases the S-lock.

</details>

---

### Problem 3: Deadlock Detection

> [!question]
> Given:
> - T1 holds Lock(A), wants Lock(B)
> - T2 holds Lock(B), wants Lock(A)
> 
> Is there a deadlock?

<details>
<summary>Answer</summary>

**Yes**. Waits-for graph:
```
T1 → T2 → T1  (Cycle!)
```

**Solution**: Abort either T1 or T2 to break the cycle.

</details>


---

