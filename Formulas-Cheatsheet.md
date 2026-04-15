# Quick Reference & Cheatsheet

[[README|← Back to Index]]

---

##  ER-to-Relational Mapping Rules

| ER Pattern | Mapping Rule | Example |
|------------|--------------|---------|
| **Regular Entity** | → Table, PK = entity key | `STUDENT(SID PK, Name)` |
| **1:N Relationship** | → FK on **N-side** | `EMPLOYEE(DNO FK → DEPARTMENT)` |
| **M:N Relationship** | → **New table** with 2 FKs | `ENROLL(SID FK, CID FK, Grade)` |
| **1:1 Relationship** | → FK on **total side** | Put FK where participation is total |
| **Weak Entity** | → Table: PK = **owner PK + partial key** | `DEPENDENT(EmpID FK, Name PK)` |
| **Multivalued Attribute** | → **Separate table** (owner PK, value) | `EMP_PHONES(SSN FK, Phone)` |
| **Derived Attribute** | → **Don't store** | Calculate `Age` from `Birth_Date` |

---

##  Normalization Quick Guide

| Normal Form | Rule | Violation Pattern | Fix |
|-------------|------|-------------------|-----|
| **1NF** | Atomic values only | Lists, nested structures | Split into rows |
| **2NF** | No partial dependency | Part-of-key → non-prime | Split table |
| **3NF** | No transitive dependency | Non-superkey → non-prime | Split table |
| **BCNF** | Every determinant is superkey | Non-superkey → anything | Split table |

### Quick Check Algorithm

```
1. Find all candidate keys (use closure)
2. Mark prime vs non-prime attributes

For 2NF:
  - If key is single attr → 2NF OK
  - If composite key: check if any part-of-key → non-prime
  
For 3NF:
  - Check: any non-superkey → non-prime?
  
For BCNF:
  - Check: any non-superkey → anything?
```

---

##  Relational Algebra Operators

| Symbol | Name | Purpose | Example |
|--------|------|---------|---------|
| **σ** | Selection | Filter rows | σ<sub>salary>50000</sub>(EMP) |
| **π** | Projection | Select columns | π<sub>name,dept</sub>(EMP) |
| **ρ** | Rename | Rename table/cols | ρ<sub>E2(id,name)</sub>(EMP) |
| **∪** | Union | Combine rows | R ∪ S |
| **∩** | Intersection | Common rows | R ∩ S |
| **−** | Difference | R not in S | R − S |
| **×** | Product | All pairs | R × S |
| **⋈** | Join | Combine on condition | R ⋈<sub>a=b</sub> S |
| **÷** | Division | "For all" queries | R ÷ S |

### RA ↔ SQL Mapping

```
π_{A,B}(σ_{C>5}(R))  ⟷  SELECT A, B FROM R WHERE C > 5

R ⋈_{R.x=S.y} S      ⟷  SELECT * FROM R JOIN S ON R.x = S.y

_{D}𝓖_{AVG(S)}(R)     ⟷  SELECT D, AVG(S) FROM R GROUP BY D
```

---

## Query Optimization Rules

### Heuristics

1. **Push σ (selection) down** → Filter early
2. **Push π (projection) down** → Keep only needed columns
3. **Replace (product + selection) with join**
4. **Do most selective operations first**
5. **Join smaller tables first**

### Example Transformation

 **Before**:
```
π_{name}(σ_{dept='CS'}(EMPLOYEE × DEPARTMENT))
```

 **After**:
```
π_{name}(σ_{dept='CS'}(EMPLOYEE) ⋈_{dno=did} DEPARTMENT)
```

---

##  Access Methods (Selection Algorithms)

| Code | Method | When to Use |
|------|--------|-------------|
| **S1** | Linear Scan | No index, unsorted |
| **S2** | Binary Search | File ordered on key, equality |
| **S3** | Primary Index (equality) | Clustered index on key |
| **S4** | Primary Index (range) | Clustered index, range query |
| **S5** | Clustering Index | Non-key with clustering |
| **S6** | Secondary Index | B+ tree, equality/range |

### Selection Logic

```
1. Equality on primary/clustered key? → S3
2. Range on ordered column? → S4
3. Clustering index on predicate column? → S5
4. Secondary B+ tree (few matches)? → S6
5. Otherwise → S1 (scan)
```

---

##  Join Algorithms

| Code | Method | When to Use | Time |
|------|--------|-------------|------|
| **J1** | Nested-Loop | No index, small tables | O(n·m) |
| **J2** | Index Nested-Loop | **Inner has index** on join key | O(n·log m) |
| **J3** | Sort-Merge | Both sorted (or need sorted output) | O(n log n + m log m) |
| **J4** | Hash Join | Equality join, no indexes, enough RAM | O(n + m) |

### Join Selection Logic

```
1. Inner has index on join key? → J2
2. No indexes, equality join, enough RAM? → J4
3. Both sorted or need sorted output? → J3
4. Otherwise (or outer is tiny)? → J1
```

---

##  Locking Quick Reference

### Lock Types

| Lock | Purpose | Compatible With |
|------|---------|-----------------|
| **S** | Read | S |
| **X** | Write | Nothing |
| **IS** | Intent to read children | IS, IX, S |
| **IX** | Intent to write children | IS, IX |
| **SIX** | S + IX | IS |

### Lock Compatibility Matrix

|     | IS  | IX  | S   | SIX | X   |
| --- | --- | --- | --- | --- | --- |
| IS  | Yes | Yes | Yes | Yes | No  |
| IX  | Yes | Yes | No  | No  | No  |
| S   | Yes | No  | Yes | No  | No  |
| SIX | Yes | No  | No  | No  | No  |
| X   | No  | No  | No  | No  | No  |

---

## Concurrency Problems

| Problem | Description | Example |
|---------|-------------|---------|
| **Lost Update** | Two txns update same item from stale reads | Both read 100, write 150 and 120 → 120 wins |
| **Dirty Read** | Read uncommitted data that later rolls back | T1 writes 200, T2 reads 200, T1 aborts → T2 used invalid data |
| **Unrepeatable Read** | Same query returns different values | T1 reads X=100, T2 updates X=200, T1 reads X=200 |
| **Phantom Read** | Same query returns different rows | T1: COUNT(*) = 5, T2 inserts row, T1: COUNT(*) = 6 |

**Solution**: **Strict 2PL** (hold all locks until commit)

---

##  SQL Quick Patterns

### Basic Query Structure

```sql
SELECT [columns]
FROM [table]
WHERE [row filter]
GROUP BY [grouping]
HAVING [group filter]
ORDER BY [sort];
```

### Common Patterns

**Find employees earning above dept average**:
```sql
SELECT Name
FROM EMPLOYEE E1
WHERE Salary > (
    SELECT AVG(Salary)
    FROM EMPLOYEE E2
    WHERE E2.DNo = E1.DNo
);
```

**"For all" with NOT EXISTS**:
```sql
-- Employees who worked on ALL projects in Houston
SELECT FName
FROM EMPLOYEE E
WHERE NOT EXISTS (
    SELECT PNo FROM PROJECT WHERE PLocation = 'Houston'
    EXCEPT
    SELECT PNo FROM WORKS_ON WHERE ESSN = E.SSN
);
```

**Departments with >5 employees earning avg >60K**:
```sql
SELECT DNo, AVG(Salary) AS AvgSal
FROM EMPLOYEE
GROUP BY DNo
HAVING COUNT(*) > 5 AND AVG(Salary) > 60000;
```

---

##  Exam Patterns

### Pattern 1: ER Diagram → Tables

1. Identify entity types → tables
2. Check relationships:
   - 1:N → FK on N-side
   - M:N → new relationship table
   - 1:1 → FK on total side
3. Weak entities → PK = owner PK + partial key
4. Multivalued → separate table

---

### Pattern 2: Find Candidate Keys

```
1. Find attributes that NEVER appear on RHS → likely key
2. Compute closure: X⁺
3. If X⁺ = all attributes → X is a superkey
4. Remove attributes one at a time to find minimal key
```

---

### Pattern 3: Check Normal Forms

```
1. Find all candidate keys (closure)
2. Mark prime vs non-prime
3. Check 2NF: any part-of-composite-key → non-prime?
4. Check 3NF: any non-superkey → non-prime?
5. Check BCNF: any non-superkey → anything?
```

---

### Pattern 4: Convert RA to SQL

```
σ_{cond}       → WHERE cond
π_{A,B}        → SELECT A, B
R ⋈_{cond} S   → R JOIN S ON cond
_{G}𝓖_{agg(A)} → SELECT G, agg(A) ... GROUP BY G
```

---

## Common Mistakes to Avoid

1. **NULL checks**: Use `IS NULL`, not `= NULL`
2. **HAVING vs WHERE**: HAVING filters groups, WHERE filters rows
3. **COUNT(*) vs COUNT(col)**: COUNT(*) includes NULLs
4. **2NF**: Only matters with composite keys
5. **BCNF vs 3NF**: BCNF has NO exceptions (not even prime)
6. **Closure**: Start with X⁺ = X, then iteratively add attributes
7. **Division (÷)**: Use for "for all" queries in RA
8. **Strict 2PL**: Hold locks until **commit**, not just end of operation

---

## Tags
#cheatsheet #quick-reference #exam-prep #study-guide
