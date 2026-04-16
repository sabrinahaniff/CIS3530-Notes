# Chapter 5: Relational Algebra

[[README|← Back to Index]]

---

##  Overview

**Relational Algebra** is a **procedural** query language — you specify **how** to get the answer using a sequence of operations.

> [!abstract] Key Concept
> Each operation takes one or more **relations** (tables) as input and produces a **relation** as output.
> 
> Think of them as **LEGO blocks** you can stack together!

---

## Core Operations

### 1. Selection (σ) - Filter Rows

**Symbol**: σ  
**Purpose**: Keep rows that satisfy a condition  
**Type**: **Unary** (operates on one table)

**Notation**: σ<sub>condition</sub>(R)

**Example**:
```
σ_{DNO=5}(EMPLOYEE)
```
**Translation**: "Employees in department 5"

**SQL Equivalent**:
```sql
SELECT *
FROM EMPLOYEE
WHERE DNO = 5;
```

---

### 2. Projection (π) - Select Columns

**Symbol**: π  
**Purpose**: Keep specified columns, **remove duplicates**  
**Type**: **Unary**

**Notation**: π<sub>A,B,C</sub>(R)

**Example**:
```
π_{FNAME, LNAME}(EMPLOYEE)
```
**Translation**: "First and last names of all employees"

**SQL Equivalent**:
```sql
SELECT DISTINCT FNAME, LNAME
FROM EMPLOYEE;
```

> [!important] Duplicate Removal
> Projection **automatically removes duplicates** (unlike `SELECT` in SQL, which keeps them by default).

---

### 3. Rename (ρ) - Rename Tables/Columns

**Symbol**: ρ  
**Purpose**: Rename a relation or its attributes  
**Type**: **Unary**

**Notation**: ρ<sub>S(A,B,C)</sub>(R)

**Example**:
```
ρ_{EMPLOYEE2(SSN, NAME)}(EMPLOYEE)
```
**Translation**: "Rename EMPLOYEE to EMPLOYEE2, and rename columns to SSN and NAME"

**Why it matters**: Essential for **self-joins** and when column names clash.

---

### 4. Union (∪) - Combine Rows

**Symbol**: ∪  
**Purpose**: Return all rows from R **or** S (remove duplicates)  
**Type**: **Binary**

**Requirement**: R and S must be **union-compatible**:
- Same number of columns
- Corresponding columns have the same data types

**Example**:
```
STUDENTS ∪ TEACHERS
```

**SQL Equivalent**:
```sql
SELECT * FROM STUDENTS
UNION
SELECT * FROM TEACHERS;
```

---

### 5. Difference (−) -  Rows in R but not in S

**Symbol**: −  
**Purpose**: Rows in R that are **not** in S  
**Type**: **Binary**

**Example**:
```
ENROLLED − GRADUATED
```
**Translation**: "Students who enrolled but haven't graduated"

**SQL Equivalent**:
```sql
SELECT * FROM ENROLLED
EXCEPT
SELECT * FROM GRADUATED;
```

> [!warning] Not Commutative
> R − S ≠ S − R

---

### 6. Cartesian Product (×) - All Pairs

**Symbol**: ×  
**Purpose**: Combine **every** row of R with **every** row of S  
**Type**: **Binary**

**Example**:
```
EMPLOYEE × DEPARTMENT
```
**Result**: If EMPLOYEE has 100 rows and DEPARTMENT has 5 rows → 500 rows

**SQL Equivalent**:
```sql
SELECT *
FROM EMPLOYEE, DEPARTMENT;
```

> [!tip] Product + Selection = Join
> You almost **always** follow a product with a selection to create a meaningful join.

---

### 7. Join (⋈) - Combine Related Rows

**Symbol**: ⋈  
**Purpose**: Combine rows from R and S where a condition is satisfied  
**Type**: **Binary**

#### Theta Join (⋈<sub>θ</sub>)

Combines product and selection:

**Definition**:
```
R ⋈_{A=B} S  =  σ_{A=B}(R × S)
```

**Example**:
```
EMPLOYEE ⋈_{DNO=DNUMBER} DEPARTMENT
```

**SQL Equivalent**:
```sql
SELECT *
FROM EMPLOYEE
JOIN DEPARTMENT ON EMPLOYEE.DNO = DEPARTMENT.DNUMBER;
```

#### Equijoin

A theta join where the condition is **equality** (`=`).

#### Natural Join (⋈)

- Automatically joins on **same-named columns**
- Removes one copy of the duplicate column

**Example**:
```
EMPLOYEE ⋈ DEPARTMENT
```
(Joins on any columns with the same name, e.g., `DNO`)

> [!note] Name Mismatch
> If column names don't match, use **ρ** (rename) first to align them.

---

### 8. Division (÷) - "For All" Queries

**Symbol**: ÷  
**Purpose**: Find rows in R that are related to **all** rows in S  
**Type**: **Binary**

**Use Case**: "Find employees who worked on **all** projects in Houston"

**Example**:
```
π_{ESSN}(WORKS_ON) ÷ π_{PNO}(σ_{PLOCATION='Houston'}(PROJECT))
```

**SQL Equivalent** (using NOT EXISTS):
```sql
SELECT ESSN
FROM EMPLOYEE E
WHERE NOT EXISTS (
    SELECT PNO FROM PROJECT WHERE PLOCATION = 'Houston'
    EXCEPT
    SELECT PNO FROM WORKS_ON WHERE ESSN = E.SSN
);
```

---

##  Aggregation & Grouping

### Grouping (𝓖)

**Notation**: <sub>grouping_attrs</sub>𝓖<sub>agg_func</sub>(R)

**Example**: Average salary per department
```
_{DNO} 𝓖_{AVG(SALARY) → AvgSal}(EMPLOYEE)
```

**SQL Equivalent**:
```sql
SELECT DNO, AVG(SALARY) AS AvgSal
FROM EMPLOYEE
GROUP BY DNO;
```

### Aggregate Functions

| Function | Meaning |
|----------|---------|
| `SUM` | Total |
| `AVG` | Average |
| `COUNT` | Count |
| `MIN` | Minimum |
| `MAX` | Maximum |

---

## RA ↔ SQL Mapping

### Pattern 1: Selection + Projection

**RA**:
```
π_{A,B}(σ_{condition}(R))
```

**SQL**:
```sql
SELECT A, B
FROM R
WHERE condition;
```

---

### Pattern 2: Join

**RA**:
```
R ⋈_{R.a = S.b} S
```

**SQL**:
```sql
SELECT *
FROM R
JOIN S ON R.a = S.b;
```

---

### Pattern 3: Nested Operations

**RA**:
```
π_{FNAME,LNAME}(σ_{DNO=5 AND SALARY>50000}(EMPLOYEE))
```

**SQL**:
```sql
SELECT FNAME, LNAME
FROM EMPLOYEE
WHERE DNO = 5 AND SALARY > 50000;
```

---

## Practice Problems

### Problem 1: Basic Selection & Projection

> [!question]
> Find first and last names of employees earning over $90,000.

<details>
<summary>RA Answer</summary>

```
π_{FNAME,LNAME}(σ_{SALARY>90000}(EMPLOYEE))
```

</details>

<details>
<summary>SQL Answer</summary>

```sql
SELECT FNAME, LNAME
FROM EMPLOYEE
WHERE SALARY > 90000;
```

</details>

---

### Problem 2: Join

> [!question]
> List employee names with their department names.

<details>
<summary>RA Answer</summary>

```
π_{FNAME,LNAME,DNAME}(EMPLOYEE ⋈_{DNO=DNUMBER} DEPARTMENT)
```

</details>

<details>
<summary>SQL Answer</summary>

```sql
SELECT E.FNAME, E.LNAME, D.DNAME
FROM EMPLOYEE E
JOIN DEPARTMENT D ON E.DNO = D.DNUMBER;
```

</details>

---

### Problem 3: "For All" with Division

> [!question]
> Find employees who worked on all projects controlled by department 5.

<details>
<summary>RA Answer</summary>

```
π_{ESSN}(WORKS_ON) ÷ π_{PNO}(σ_{DNUM=5}(PROJECT))
```

Then join back to EMPLOYEE to get names:
```
π_{FNAME,LNAME}(
    EMPLOYEE ⋈_{SSN=ESSN} (π_{ESSN}(WORKS_ON) ÷ π_{PNO}(σ_{DNUM=5}(PROJECT)))
)
```

</details>

---

## Key Properties

### Commutative Operations

- σ<sub>c1</sub>(σ<sub>c2</sub>(R)) = σ<sub>c2</sub>(σ<sub>c1</sub>(R))
- R ∪ S = S ∪ R
- R ⋈ S = S ⋈ R

### Non-Commutative

- R − S ≠ S − R
- (Usually) R ÷ S ≠ S ÷ R

### Associative

- (R ∪ S) ∪ T = R ∪ (S ∪ T)
- (R ⋈ S) ⋈ T = R ⋈ (S ⋈ T)

---

## Query Optimization Rules

> [!tip] Push Operations Down
> 1. **Push σ (selection) down** → Filter early to reduce data size
> 2. **Push π (projection) down** → Keep only needed columns
> 3. **Replace × followed by σ with ⋈** → More efficient

**Example**:

**Inefficient**:
```
π_{FNAME}(σ_{DNO=5}(EMPLOYEE × DEPARTMENT))
```

 **Optimized**:
```
π_{FNAME}(σ_{DNO=5}(EMPLOYEE) ⋈_{DNO=DNUMBER} DEPARTMENT)
```

---

## Related Topics

- [[04-SQL-Fundamentals|SQL Fundamentals]]
- [[06-Relational-Calculus|Relational Calculus (TRC)]]
- [[11-Query-Processing|Query Optimization]]

---

