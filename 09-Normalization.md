# Chapter 9: Normalization (1NF, 2NF, 3NF, BCNF)

[[README|← Back to Index]]

---

##  Overview

**Normalization** is the process of organizing database schemas to:
- **Minimize redundancy**
- **Prevent update/insert/delete anomalies**
- **Ensure data integrity**

> [!important] The Staircase Analogy
> Each normal form is a "step" that adds more restrictions:
> ```
> 1NF ⊆ 2NF ⊆ 3NF ⊆ BCNF
> ```
> - Every BCNF table is in 3NF
> - Every 3NF table is in 2NF
> - Every 2NF table is in 1NF

---

## Prime vs Non-Prime Attributes

Before understanding normalization, you **must** classify attributes:

| Type | Definition | Example |
|------|------------|---------|
| **Prime** | Member of **any** candidate key | In `ENROLL(SID, CID, Term)`, if key is `(SID, CID)`, then `SID` and `CID` are prime |
| **Non-Prime** | **Not** in any candidate key | `Term`, `Instructor`, `Grade` are non-prime |

> [!tip] How to Find Prime Attributes
> 1. Find all **candidate keys** using attribute closure
> 2. Any attribute appearing in **any** candidate key is **prime**
> 3. Everything else is **non-prime**

---

## 1NF (First Normal Form)

### Definition

> [!abstract] 1NF Rule
> Every cell must contain **only atomic values** — no lists, arrays, or nested structures.

### Violation Example

**NOT in 1NF**:
```
| Student | Courses                |
|---------|------------------------|
| Alice   | [CIS3530, CIS3750]     |
| Bob     | [MATH2000]             |
```

 **Fixed (1NF)**:
```
| Student | Course   |
|---------|----------|
| Alice   | CIS3530  |
| Alice   | CIS3750  |
| Bob     | MATH2000 |
```

### Key Point

> [!note]
> 1NF is about **structure**, not functional dependencies. It simply requires atomic values.

---

## 2NF (Second Normal Form)

### Definition

> [!abstract] 2NF Rule
> - Must be in **1NF**
> - **No partial dependencies**: Every non-prime attribute must be **fully functionally dependent** on the **entire** primary key

### When 2NF Matters

> [!warning]
> 2NF only matters when your key has **2+ columns** (composite key).

### Violation: Partial Dependency

**Definition**: A non-prime attribute depends on **only part** of a composite key.

 **Violation Example**:

**Table**: `ENROLL(SID, CourseID, Instructor, Room, Time)`  
**Key**: `(SID, CourseID)`  
**FDs**: 
- `{SID, CourseID} → Instructor, Room, Time` (full dependency ✓)
- `CourseID → Instructor, Room` (partial dependency **x**)

**Problem**: `Instructor` and `Room` depend only on `CourseID`, not on the full key `(SID, CourseID)`.

### How to Fix 2NF Violations

**Split the table**:

 **Solution**:
```
T1: ENROLL(SID, CourseID, Time)
    Key: (SID, CourseID)

T2: COURSE(CourseID, Instructor, Room)
    Key: CourseID
```

---

## 3NF (Third Normal Form)

### Definition

> [!abstract] 3NF Rule
> For every non-trivial FD `X → A`:
> - **Either** X is a superkey
> - **Or** A is a prime attribute

**Alternative definition**: No **transitive dependencies** for non-prime attributes.

### Transitive Dependency

**Definition**: Key → Middle → Non-prime

**Example**:
- `SSN → DNumber` (SSN determines department)
- `DNumber → DMgrSSN` (department determines manager)
- Therefore: `SSN → DMgrSSN` **through** `DNumber` (transitive)

### Violation Example

 **NOT in 3NF**:

**Table**: `EMPLOYEE(SSN, Name, DNumber, DMgrSSN)`  
**Key**: `SSN`  
**FDs**:
- `SSN → DNumber` ✓
- `DNumber → DMgrSSN` ← **Problem!**

**Why it violates 3NF**:
- `DNumber` is **not a superkey**
- `DMgrSSN` is **non-prime**
- This creates a transitive dependency: `SSN → DNumber → DMgrSSN`

### How to Fix 3NF Violations

 **Solution**:
```
T1: EMPLOYEE(SSN, Name, DNumber)
    Key: SSN

T2: DEPARTMENT(DNumber, DMgrSSN)
    Key: DNumber
```

---

## BCNF (Boyce-Codd Normal Form)

### Definition

> [!abstract] BCNF Rule
> For **every** non-trivial FD `X → Y`:
> - X **must be a superkey**

**BCNF is stricter than 3NF**: It removes the "or A is prime" exception.

### Key Difference from 3NF

| 3NF | BCNF |
|-----|------|
| Allows non-superkey → prime attribute | **No exceptions**: left side must be a superkey |

### Violation Example

 **NOT in BCNF** (but is in 3NF):

**Table**: `TEACH(Student, Course, Instructor)`  
**Keys**: `{Student, Course}` and `{Student, Instructor}`  
**FDs**:
- `{Student, Course} → Instructor` ✓
- `{Student, Instructor} → Course` ✓
- `Course → Instructor` ← **Violates BCNF!**

**Why**:
- `Course` is **not a superkey**
- Even though `Instructor` is **prime**, BCNF doesn't allow this

### How to Fix BCNF Violations

 **Solution**:
```
T1: COURSE_INSTRUCTOR(Course, Instructor)
    Key: Course

T2: STUDENT_COURSE(Student, Course)
    Key: (Student, Course)
```

---

##  The Decision Tree

```
┌─────────────────────────────────────┐
│ Is the table in 1NF?                │
│ (Atomic values only?)               │
└────────┬────────────────────────────┘
         │ Yes
         ▼
┌─────────────────────────────────────┐
│ Does the key have 2+ columns?       │
└────────┬────────────────────────────┘
         │ Yes
         ▼
┌─────────────────────────────────────┐
│ Any non-prime depend on part of key?│ ← 2NF check
└────────┬────────────────────────────┘
         │ No
         ▼
┌─────────────────────────────────────┐
│ Any non-superkey → non-prime?       │ ← 3NF check
└────────┬────────────────────────────┘
         │ No
         ▼
┌─────────────────────────────────────┐
│ Any non-superkey → anything?        │ ← BCNF check
└────────┬────────────────────────────┘
         │ No
         ▼
     [ BCNF ✓ ]
```

---

## 📝 Complete Examples

### Example 1: 2NF Violation

**Given**: `ENROLL(SID, Course, Instructor, Room, Time)`  
**Key**: `(SID, Course)`  
**FDs**:
- `{SID, Course} → Instructor, Room, Time`
- `Course → Instructor, Room`

**Analysis**:
-  1NF (atomic values)
-  2NF violated: `Course → Instructor` (partial dependency)

**Fix**:
```sql
T1: ENROLL(SID, Course, Time)
T2: COURSE(Course, Instructor, Room)
```

---

### Example 2: 3NF Violation

**Given**: `EMPLOYEE(SSN, Name, DNumber, DLocation)`  
**Key**: `SSN`  
**FDs**:
- `SSN → Name, DNumber`
- `DNumber → DLocation`

**Analysis**:
- 1NF, 2NF (key is single attribute)
-  3NF violated: `DNumber → DLocation` (transitive: SSN → DNumber → DLocation)

**Fix**:
```sql
T1: EMPLOYEE(SSN, Name, DNumber)
T2: DEPARTMENT(DNumber, DLocation)
```

---

### Example 3: BCNF Violation (but 3NF OK)

**Given**: `BOOKING(Student, Course, Instructor)`  
**Keys**: `{Student, Course}`, `{Student, Instructor}`  
**FDs**:
- `Course → Instructor`

**Analysis**:
- All attributes are **prime** (in some key)
-  3NF (Instructor is prime)
-  BCNF violated: `Course` is not a superkey

**Fix**:
```sql
T1: COURSE(Course, Instructor)
T2: ENROLLMENT(Student, Course)
```

---

##  Quick Reference Table

| Normal Form | Requirement | Violation Pattern |
|-------------|-------------|-------------------|
| **1NF** | Atomic values | Lists, arrays, nested tables |
| **2NF** | No partial dependency | Part-of-key → non-prime |
| **3NF** | No transitive dependency (exceptions for prime) | Non-superkey → non-prime |
| **BCNF** | Every determinant is a superkey | Non-superkey → anything |

---

## Practice Problems

> [!question] Problem 1
> **Given**: `R(A, B, C)`, Key `= (A, B)`, FDs: `A → C`
> 
> What normal form is this in?

<details>
<summary>Answer</summary>

- ✅ 1NF (atomic)
- ❌ **2NF violated**: `A → C` is a partial dependency (A is part of key, C is non-prime)
- Need to split into: `R1(A, C)` and `R2(A, B)`

</details>

---

> [!question] Problem 2
> **Given**: `R(A, B, C, D)`, Key `= A`, FDs: `A → B`, `B → C`, `C → D`
> 
> What normal form violations exist?

<details>
<summary>Answer</summary>

- ✅ 1NF, 2NF (single-attribute key)
- ❌ **3NF violated**: `B → C` and `C → D` (transitive: A → B → C → D)
- ❌ **BCNF violated**: B and C are not superkeys
- Split into: `R1(A, B)`, `R2(B, C)`, `R3(C, D)`

</details>

---

## Related Topics

- [[08-Functional-Dependencies|← Functional Dependencies]]
- [[09b-Attribute-Closure|Attribute Closure Algorithm]]
- [[10-ER-to-Relational-Mapping|ER-to-Relational Mapping]]

---

## Tags
#normalization #1nf #2nf #3nf #bcnf #functional-dependencies #database-design
