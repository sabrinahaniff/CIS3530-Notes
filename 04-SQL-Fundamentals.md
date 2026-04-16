# Chapter 4: SQL Fundamentals

[[README|← Back to Index]]

---

## Query Structure

### The SQL Reading Pattern

> [!tip] Read SQL Bottom-to-Top
> ```sql
> SELECT [what to show]        ← 5. Finally, show these columns
> FROM [tables]                ← 1. Start with these tables
> WHERE [row filter]           ← 2. Keep only rows that match
> GROUP BY [grouping]          ← 3. Group rows by this column
> HAVING [group filter]        ← 4. Keep only groups that match
> ORDER BY [sort]              ← 6. Sort the final result
> ```

---

## Basic SELECT

### Simple Query

```sql
SELECT FName, LName
FROM EMPLOYEE
WHERE SALARY > 50000;
```

**Translation**: "Show first and last names of employees earning over $50,000"

### Select All Columns

```sql
SELECT *
FROM EMPLOYEE;
```

---

## WHERE Clause Operators

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal | `WHERE Age = 25` |
| `<>` or `!=` | Not equal | `WHERE Dept <> 'Sales'` |
| `>`, `<` | Greater/Less than | `WHERE Salary > 60000` |
| `>=`, `<=` | Greater/Less or equal | `WHERE Age >= 18` |
| `BETWEEN` | Range (inclusive) | `WHERE Salary BETWEEN 50000 AND 80000` |
| `IN` | Match any in list | `WHERE Dept IN ('Sales', 'HR')` |
| `LIKE` | Pattern match | `WHERE Name LIKE 'A%'` |
| `IS NULL` | Check for NULL | `WHERE Phone IS NULL` |

### Pattern Matching with LIKE

| Pattern | Meaning | Example |
|---------|---------|---------|
| `%` | Any string (0+ chars) | `'A%'` → Starts with A |
| `_` | Single character | `'_at'` → `cat`, `bat`, `hat` |

**Examples**:
```sql
-- Names starting with 'John'
WHERE Name LIKE 'John%'

-- 4-letter names ending in 'y'
WHERE Name LIKE '___y'

-- Contains 'son'
WHERE Name LIKE '%son%'
```

> [!note] Case-Insensitive Matching
> Use `ILIKE` instead of `LIKE` for case-insensitive pattern matching (PostgreSQL)

---

## NULL Handling

### The NULL Rules

> [!warning] NULL is UNKNOWN
> - `NULL = NULL` → **NULL** (not TRUE!)
> - `5 > NULL` → **NULL**
> - `NULL OR TRUE` → **TRUE**
> - `NULL AND TRUE` → **NULL**

### Correct NULL Checks

 **WRONG**:
```sql
WHERE Phone = NULL  -- This never works!
```

 **CORRECT**:
```sql
WHERE Phone IS NULL

WHERE Phone IS NOT NULL
```

---

##  JOINS

### INNER JOIN (Default)

Returns rows where the join condition is TRUE.

```sql
SELECT D.DName, E.FName, E.LName
FROM DEPARTMENT D
JOIN EMPLOYEE E ON D.MgrSSN = E.SSN;
```

**Translation**: "Show department names with their manager names"

### LEFT OUTER JOIN

Returns all rows from the left table, NULL for unmatched right.

```sql
SELECT E.FName, D.DName
FROM EMPLOYEE E
LEFT JOIN DEPARTMENT D ON E.DNo = D.DNumber;
```

**Result**: All employees, even those without a department (DName = NULL)

### RIGHT OUTER JOIN

Returns all rows from the right table, NULL for unmatched left.

```sql
SELECT E.FName, D.DName
FROM EMPLOYEE E
RIGHT JOIN DEPARTMENT D ON E.DNo = D.DNumber;
```

**Result**: All departments, even those without employees

### FULL OUTER JOIN

Returns all rows from both tables.

```sql
SELECT E.FName, D.DName
FROM EMPLOYEE E
FULL OUTER JOIN DEPARTMENT D ON E.DNo = D.DNumber;
```

---

## Aggregation Functions

### Basic Aggregates

| Function | Purpose | Example |
|----------|---------|---------|
| `COUNT(*)` | Count all rows | `SELECT COUNT(*) FROM EMPLOYEE` |
| `COUNT(col)` | Count non-NULL values | `SELECT COUNT(Phone) FROM EMPLOYEE` |
| `SUM(col)` | Sum of values | `SELECT SUM(Salary) FROM EMPLOYEE` |
| `AVG(col)` | Average | `SELECT AVG(Salary) FROM EMPLOYEE` |
| `MIN(col)` | Minimum | `SELECT MIN(Salary) FROM EMPLOYEE` |
| `MAX(col)` | Maximum | `SELECT MAX(Salary) FROM EMPLOYEE` |

> [!important] COUNT(*) vs COUNT(col)
> - `COUNT(*)` counts **all rows** (including NULLs)
> - `COUNT(col)` counts only **non-NULL** values

---

## GROUP BY

Group rows by column values, then apply aggregates to each group.

### Basic Example

```sql
SELECT DNo, AVG(Salary) AS AvgSalary
FROM EMPLOYEE
GROUP BY DNo;
```

**Result**: Average salary per department

### Multiple Columns

```sql
SELECT DNo, JobTitle, COUNT(*) AS EmpCount
FROM EMPLOYEE
GROUP BY DNo, JobTitle;
```

**Result**: Employee count for each department-job combination

---

## HAVING (Group Filter)

Filter groups **after** aggregation (unlike WHERE which filters rows **before** aggregation).

```sql
SELECT DNo, AVG(Salary) AS AvgSalary
FROM EMPLOYEE
GROUP BY DNo
HAVING AVG(Salary) > 60000;
```

**Translation**: "Show departments where average salary > $60,000"

### WHERE vs HAVING

| Clause   | Filters    | When            | Can Use Aggregates? |
| -------- | ---------- | --------------- | ------------------- |
| `WHERE`  | **Rows**   | Before grouping | No                  |
| `HAVING` | **Groups** | After grouping  | Yes                 |

**Example**:
```sql
SELECT DNo, AVG(Salary)
FROM EMPLOYEE
WHERE Salary > 40000      -- Filter rows first
GROUP BY DNo
HAVING COUNT(*) > 5;      -- Then filter groups
```

---

## 🎯 Set Operations

### UNION (Remove Duplicates)

```sql
SELECT Name FROM Students
UNION
SELECT Name FROM Teachers;
```

### UNION ALL (Keep Duplicates)

```sql
SELECT Name FROM Students
UNION ALL
SELECT Name FROM Teachers;
```

### INTERSECT (Common Rows)

```sql
SELECT CourseID FROM Fall2024
INTERSECT
SELECT CourseID FROM Winter2025;
```

### EXCEPT/MINUS (Difference)

```sql
SELECT StudentID FROM Enrolled
EXCEPT
SELECT StudentID FROM Graduated;
```

> [!note] Union Compatibility
> All set operations require tables to have:
> - Same **number of columns**
> - **Compatible data types** in corresponding positions

---

##  Subqueries

### IN (Membership)

```sql
SELECT FName, LName
FROM EMPLOYEE
WHERE DNo IN (
    SELECT DNumber
    FROM DEPARTMENT
    WHERE DName = 'Research'
);
```

**Translation**: "Employees in the Research department"

### EXISTS (Check for Existence)

```sql
SELECT FName
FROM EMPLOYEE E
WHERE EXISTS (
    SELECT *
    FROM DEPENDENT D
    WHERE D.ESSN = E.SSN
);
```

**Translation**: "Employees who have at least one dependent"

### NOT EXISTS ("None" Queries)

```sql
SELECT FName
FROM EMPLOYEE E
WHERE NOT EXISTS (
    SELECT *
    FROM DEPENDENT D
    WHERE D.ESSN = E.SSN
);
```

**Translation**: "Employees with no dependents"

### Correlated Subquery

```sql
SELECT E1.FName, E1.Salary
FROM EMPLOYEE E1
WHERE E1.Salary > (
    SELECT AVG(E2.Salary)
    FROM EMPLOYEE E2
    WHERE E2.DNo = E1.DNo
);
```

**Translation**: "Employees earning above their department's average"

---

## DISTINCT (Remove Duplicates)

```sql
SELECT DISTINCT DNo
FROM EMPLOYEE;
```

**Result**: List of unique department numbers

---

## ORDER BY (Sorting)

```sql
SELECT FName, LName, Salary
FROM EMPLOYEE
ORDER BY Salary DESC, LName ASC;
```

- `DESC` = Descending (high to low)
- `ASC` = Ascending (low to high, default)

---

##  Practice Problems

### Problem 1: Basic Query

> [!question]
> Write a query to find employees earning between $50,000 and $80,000, ordered by salary.

<details>
<summary>Answer</summary>

```sql
SELECT FName, LName, Salary
FROM EMPLOYEE
WHERE Salary BETWEEN 50000 AND 80000
ORDER BY Salary DESC;
```

</details>

---

### Problem 2: JOIN

> [!question]
> List all employees with their department names.

<details>
<summary>Answer</summary>

```sql
SELECT E.FName, E.LName, D.DName
FROM EMPLOYEE E
JOIN DEPARTMENT D ON E.DNo = D.DNumber;
```

</details>

---

### Problem 3: Aggregation with HAVING

> [!question]
> Find departments with more than 5 employees and average salary > $60,000.

<details>
<summary>Answer</summary>

```sql
SELECT DNo, COUNT(*) AS EmpCount, AVG(Salary) AS AvgSal
FROM EMPLOYEE
GROUP BY DNo
HAVING COUNT(*) > 5 AND AVG(Salary) > 60000;
```

</details>

---

## Related Topics

- [[05-Relational-Algebra|Relational Algebra ↔ SQL Mapping]]
- [[04b-Advanced-SQL|Advanced SQL (Window Functions, CTEs)]]
- [[06-Relational-Calculus|Relational Calculus]]

---
