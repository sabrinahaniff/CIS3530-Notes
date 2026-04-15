# Chapter 2b: Relationship Types & Structural Constraints

[[README|← Back to Index]] | [[02-ER-Modeling|← ER Modeling Basics]]

---

## Binary Relationships & Roles

A **binary relationship** connects **two entity types**.

### Cardinality Ratios

Written on the diamond in ER diagrams:

| Notation | Meaning | Example |
|----------|---------|---------|
| **1:1** | One-to-One | `MANAGES`: One employee manages one department |
| **1:N** | One-to-Many | `WORKS_FOR`: One department has many employees |
| **M:N** | Many-to-Many | `WORKS_ON`: Many employees work on many projects |

---

## Participation Constraints

### Total vs Partial Participation

| Type | Symbol | Meaning | Example |
|------|--------|---------|---------|
| **Total** | Double line (==) | Every entity **must** participate | Every `EMPLOYEE` works for some `DEPARTMENT` |
| **Partial** | Single line (−) | Some entities may **not** participate | Not every `EMPLOYEE` is a `MANAGER` |

**Diagram**:
```
EMPLOYEE ==WORKS_FOR== DEPARTMENT
   (total)          (partial)
```

> [!important] Total Participation
> **Total participation** = **existence dependency**
> - Every employee MUST work for a department
> - An employee cannot exist without being assigned to a department

---

## Relationship Attributes

Relationships can have their own attributes:

### Examples

**WORKS_ON** (EMPLOYEE, PROJECT):
- Attributes: `Hours`, `Start_Date`

**MANAGES** (EMPLOYEE, DEPARTMENT):
- Attributes: `Start_Date`

> [!tip] When to Use Relationship Attributes
> If the attribute describes the **connection** between entities (not the entities themselves), it belongs on the relationship.

---

## Roles

**Roles** name how each entity participates in the relationship.

### Recursive Relationships

When an entity relates to **itself**, roles become essential:

**Example: SUPERVISION**
```
EMPLOYEE ─(Supervisor)─→ SUPERVISION ←─(Supervisee)─ EMPLOYEE
```

- One `EMPLOYEE` plays the **Supervisor** role
- Another `EMPLOYEE` plays the **Supervisee** role

### Role Naming Example

> "A mother has a maximum of N daughters"

```
Mother ─(1)─ Has_Daughter ─(N)─ Daughter
```

- Every daughter participates (total) in "having a mother"
- A daughter has a maximum of 1 mother

---

## Structural Constraints Examples

### Example 1: Employee-Department

**Scenario**:
- Every employee works for **exactly one** department (total, 1)
- A department can have **zero or many** employees (partial, N)

**ER Diagram**:
```
EMPLOYEE ==1── WORKS_FOR ──N== DEPARTMENT
```

**Translation**:
- `EMPLOYEE` has total participation (double line)
- `DEPARTMENT` has partial participation (single line)
- Cardinality: **1:N**

---

### Example 2: Employee-Project (Many-to-Many)

**Scenario**:
- An employee can work on **many projects**
- A project can have **many employees**
- Track **hours** each employee works on each project

**ER Diagram**:
```
EMPLOYEE ──M── WORKS_ON ──N── PROJECT
              [Hours]
```

**Translation**:
- M:N relationship
- Relationship attribute: `Hours`

---

##  Practice Problems

### Problem 1: Interpreting Cardinality

> [!question]
> Given: `STUDENT ──1── ASSIGNED_TO ──N── ADVISOR`
> 
> What does this mean?

<details>
<summary>Answer</summary>

- Each **student** is assigned to **exactly one** advisor
- Each **advisor** can have **many** students (0 to N)
- This is a **1:N** relationship from ADVISOR to STUDENT

</details>

---

### Problem 2: Participation

> [!question]
> Scenario: "Every course must have an instructor, but not every instructor teaches a course this semester."
> 
> How do you model participation?

<details>
<summary>Answer</summary>

```
INSTRUCTOR ──1── TEACHES ──N== COURSE
  (partial)              (total)
```

- `COURSE` has **total participation** (every course must have an instructor)
- `INSTRUCTOR` has **partial participation** (some instructors don't teach)

</details>

---

## Common Patterns

### Pattern 1: Manager-Department (1:1)

- One department has at most one manager
- One employee manages at most one department

```
EMPLOYEE ──1── MANAGES ──1── DEPARTMENT
         [Start_Date]
```

---

### Pattern 2: Parent-Child Hierarchy (Recursive)

```
EMPLOYEE ───(Manager)──→ SUPERVISION ←──(Subordinate)─── EMPLOYEE
```

- Self-referencing relationship
- Roles are **mandatory** to distinguish the two sides

---

## Mapping to Relations (Preview)

> [!tip] Quick Rule
> - **1:N**: Foreign key goes on the **N-side**
> - **M:N**: Create a **new relationship table** with two foreign keys
> - **1:1**: Foreign key goes on the **total participation** side

See [[10-ER-to-Relational-Mapping|ER-to-Relational Mapping]] for full details.

---

## Related Topics

- [[02-ER-Modeling|← Back to ER Basics]]
- [[02c-Weak-Entities|Next: Weak Entities]]
- [[10-ER-to-Relational-Mapping|ER-to-Relational Mapping]]

---

## Tags
#relationships #cardinality #participation #roles #er-diagram
