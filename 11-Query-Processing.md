# Chapter 11: Query Processing & Optimization

[[README|← Back to Index]]

---

##  Overview

**Query Processing**: Translating SQL into executable operations  
**Query Optimization**: Choosing the **cheapest** execution plan

> [!abstract] Goal
> Minimize **disk I/O** (the most expensive operation)

---

## Query Processing Pipeline

```
SQL Query
   ↓
Parser/Translator
   ↓
Relational Algebra Expression
   ↓
Query Optimizer
   ↓
Execution Plan
   ↓
Query Execution Engine
   ↓
Result
```

---

## The Optimization Problem

### Why Optimize?

The same query can be executed in **many different ways**:

**Example**: `SELECT * FROM R, S, T WHERE R.a = S.a AND S.b = T.b`

**Possible Execution Orders**:
1. (R ⋈ S) ⋈ T
2. (R ⋈ T) ⋈ S  ← Might not even work!
3. R ⋈ (S ⋈ T)

**Size Matters**: Intermediate result sizes vary dramatically!

---

## Cost Model

**What we count**: **Disk I/O operations** (reads/writes)

> [!important] Key Insight
> - **RAM** is ~50× faster than SSD
> - **SSD** is ~156× faster than HDD
> - **Disk I/O dominates query cost**

**Goal**: Keep data in memory (buffer pool) as much as possible

---

## 🔍 Access Methods (Selection Algorithms)

### S1: Linear Scan

**Description**: Read every block and test each record  
**When to use**: No index, unsorted file, or predicate has low selectivity

**Cost**: O(b) where b = number of blocks

---

### S2: Binary Search

**Description**: Binary search on ordered file, then sequential scan  
**When to use**: File ordered on search key, equality condition

**Cost**: O(log b) + scan matching blocks

---

### S3: Primary Index (Equality)

**Description**: Use clustered index to jump directly to the record  
**When to use**: Equality on primary/clustering key

**Cost**: O(log n) + 1 block read

---

### S4: Primary Index (Range)

**Description**: Use index to find first match, then scan sequentially  
**When to use**: Range query (<, >, BETWEEN) on clustering key

**Cost**: O(log n) + scan contiguous blocks

---

### S5: Clustering Index (Non-Key Equality)

**Description**: Jump to first match via clustering index, scan group  
**When to use**: Equality on clustered non-key attribute

**Cost**: O(log n) + few blocks (records are clustered)

---

### S6: Secondary B+ Tree Index

**Description**: B+ tree on non-clustering attribute  
**When to use**: Equality or range, but records are **not** clustered

**Cost**: 
- **Few matches**: O(log n) + k blocks (where k = # matches)
- **Many matches**: Linear scan is cheaper!

---

## Selection Decision Tree

```
┌─────────────────────────────────────┐
│ Equality on primary/clustering key? │
└────────┬────────────────────────────┘
         │ Yes → S3
         │ No
         ▼
┌─────────────────────────────────────┐
│ Range on ordered/clustering column? │
└────────┬────────────────────────────┘
         │ Yes → S4
         │ No
         ▼
┌─────────────────────────────────────┐
│ Clustering index on predicate col?  │
└────────┬────────────────────────────┘
         │ Yes → S5
         │ No
         ▼
┌─────────────────────────────────────┐
│ Secondary index with few matches?   │
└────────┬────────────────────────────┘
         │ Yes → S6
         │ No → S1 (scan)
```

---

##  Join Algorithms

### J1: Nested-Loop Join

**Algorithm**:
```
for each r in R:
    for each s in S:
        if r.a = s.b:
            output (r, s)
```

**Cost**: O(b<sub>R</sub> · b<sub>S</sub>)  
**When to use**: No better option, or R is very small

**Tip**: Make the **smaller** table the outer loop!

---

### J2: Index Nested-Loop Join

**Algorithm**:
```
for each r in R:
    use index on S to find matching s
    output (r, s)
```

**Cost**: O(b<sub>R</sub> + |R| · log|S|)  
**When to use**: **Inner table has index** on join key

> [!tip] Golden Rule
> If inner table has an index on the join key → **Use J2**

---

### J3: Sort-Merge Join

**Algorithm**:
```
1. Sort R on join key (if not sorted)
2. Sort S on join key (if not sorted)
3. Merge both sorted lists:
   - Advance pointers when no match
   - Output matches
```

**Cost**: O(b<sub>R</sub> log b<sub>R</sub> + b<sub>S</sub> log b<sub>S</sub> + b<sub>R</sub> + b<sub>S</sub>)

**When to use**:
- Both inputs already sorted
- Need sorted output anyway
- Good for large tables with range joins

---

### J4: Hash Join

**Algorithm**:
```
1. Hash R into buckets (partition phase)
2. Hash S into same buckets
3. For each bucket pair:
   - Load R's bucket into memory
   - Probe with S's bucket
   - Output matches
```

**Cost**: O(b<sub>R</sub> + b<sub>S</sub>) — **Linear!**

**When to use**:
- **Equality join** (= condition)
- No indexes available
- Enough memory for hash table

> [!important] Hash Join is King
> For equality joins without indexes, hash join is usually the **fastest**.

---

## Join Selection Logic

```
┌─────────────────────────────────────┐
│ Does inner table have index on key? │
└────────┬────────────────────────────┘
         │ Yes → J2 (Index Nested-Loop)
         │ No
         ▼
┌─────────────────────────────────────┐
│ Is it an equality join?             │
└────────┬────────────────────────────┘
         │ Yes → J4 (Hash Join)
         │ No
         ▼
┌─────────────────────────────────────┐
│ Are both inputs sorted / need sorted│
│ output?                             │
└────────┬────────────────────────────┘
         │ Yes → J3 (Sort-Merge)
         │ No → J1 (Nested-Loop)
```

---

##  Heuristic Optimization Rules

### Rule 1: Push Selection (σ) Down

**Why**: Filter early to reduce data size

 **Bad**:
```
σ_{dept='CS'}(EMPLOYEE ⋈ DEPARTMENT)
```

**Good**:
```
σ_{dept='CS'}(EMPLOYEE) ⋈ DEPARTMENT
```

---

### Rule 2: Push Projection (π) Down

**Why**: Keep only needed columns (but keep join keys!)

 **Bad**:
```
π_{name}(EMPLOYEE ⋈_{dno=did} DEPARTMENT)
```

 **Good**:
```
π_{name}(π_{name,dno}(EMPLOYEE) ⋈_{dno=did} π_{did}(DEPARTMENT))
```

---

### Rule 3: Replace (× + σ) with ⋈

 **Bad**:
```
σ_{R.a=S.b}(R × S)
```

 **Good**:
```
R ⋈_{R.a=S.b} S
```

---

### Rule 4: Do Selective Operations First

If `σ_{salary>100000}` filters 90% of rows, do it **before** joining!

---

### Rule 5: Join Smaller Tables First

**Example**: If |R| = 100, |S| = 10,000, |T| = 1,000

**Good order**: (R ⋈ S) ⋈ T  
**Bad order**: (S ⋈ T) ⋈ R

---

##  External Sort-Merge (Important Formula!)

**Given**:
- **b** = number of blocks in file
- **n<sub>B</sub>** = number of buffer pages

**Steps**:
1. **Initial Runs**: n<sub>R</sub> = ⌈b / n<sub>B</sub>⌉
2. **Merge Degree**: d<sub>M</sub> = min(n<sub>B</sub> − 1, n<sub>R</sub>)
3. **Number of Passes**: n<sub>P</sub> = ⌈log<sub>d<sub>M</sub></sub>(n<sub>R</sub>)⌉ + 1

**Total Cost**: 2b · n<sub>P</sub>

---

### Example

**Given**: b = 1200 blocks, n<sub>B</sub> = 101 buffers

**Solution**:
1. n<sub>R</sub> = ⌈1200 / 101⌉ = **12 runs**
2. d<sub>M</sub> = min(101 − 1, 12) = **12**
3. n<sub>P</sub> = ⌈log<sub>12</sub>(12)⌉ + 1 = **1 + 1 = 2 passes**

**Total Cost**: 2 · 1200 · 2 = **4800 block I/Os**

---

##  Practice Problems

### Problem 1: Access Method Selection

> [!question]
> Given:
> - Query: `SELECT * FROM STUDENT WHERE SID = 42`
> - Primary index on `SID`
> 
> Which access method?

<details>
<summary>Answer</summary>

**S3** (Primary Index, Equality)

Use the clustered index to jump directly to the record.

</details>

---

### Problem 2: Join Algorithm Selection

> [!question]
> Given:
> - Query: `R ⋈_{R.id=S.id} S`
> - No indexes
> - Equality join
> - Enough RAM for hash table
> 
> Which join algorithm?

<details>
<summary>Answer</summary>

**J4** (Hash Join)

Equality join + no indexes + sufficient memory → Hash join is optimal.

</details>

---

### Problem 3: Query Rewrite

> [!question]
> Optimize this query:
> ```
> SELECT name
> FROM (EMPLOYEE × DEPARTMENT)
> WHERE dept = 'CS' AND emp.dno = dept.did
> ```

<details>
<summary>Answer</summary>

**Step 1**: Replace product + selection with join:
```
SELECT name
FROM EMPLOYEE ⋈_{dno=did} DEPARTMENT
WHERE dept = 'CS'
```

**Step 2**: Push selection down:
```
SELECT name
FROM EMPLOYEE ⋈_{dno=did} (σ_{dept='CS'}(DEPARTMENT))
```

**Step 3**: Push projection down:
```
π_{name}(π_{name,dno}(EMPLOYEE) ⋈_{dno=did} π_{did}(σ_{dept='CS'}(DEPARTMENT)))
```

</details>

---

## Optimization Check

Before submitting a query plan:

- [ ] Pushed all selections (σ) as far down as possible?
- [ ] Pushed projections (π) down (keeping join keys)?
- [ ] Replaced all (× + σ) with ⋈?
- [ ] Joined smaller tables first?
- [ ] Used index nested-loop if inner has index?
- [ ] Used hash join for equality joins without indexes?
- [ ] Used sort-merge if inputs are sorted?

---

## Related Topics

- [[12-Access-Methods|Detailed Access Methods]]
- [[13-Join-Algorithms|Detailed Join Algorithms]]
- [[05-Relational-Algebra|Relational Algebra Basics]]

---
