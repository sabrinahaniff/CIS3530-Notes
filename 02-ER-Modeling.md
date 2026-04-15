# Chapter 2: ER Modeling & Diagrams

[[README|← Back to Index]]

## Overview

Entity-Relationship (ER) diagrams are used to model the **logical structure** of a database by representing entities, their attributes, and relationships between entities.

---

## Core ER Concepts

### Entity Type vs Entity Set

| Concept | Definition | Analogy |
|---------|------------|---------|
| **Entity Type (Schema)** | The blueprint/structure that defines how data is organized | The class `Car` in programming |
| **Entity Set** | The current collection of entity instances | All actual `Car` objects in memory |

> [!note] Key Distinction
> - **Schema**: Fixed structure (columns, tables, relationships, rules)
> - **State**: Current data (varies with INSERT/UPDATE/DELETE)

**Example**: 
- Schema defines: `STUDENT(ID, Name, Major)`
- State contains: `{(1, "Alice", "CS"), (2, "Bob", "Math")}`

---

## ER Diagram Notation

### Basic Symbols

| Symbol | Meaning | Example |
|--------|---------|---------|
| Rectangle | Entity Type | `EMPLOYEE`, `DEPARTMENT` |
| Oval | Attribute | `Name`, `SSN`, `Salary` |
| Diamond | Relationship | `WORKS_FOR`, `MANAGES` |
| Double Rectangle | Weak Entity | `DEPENDENT` |
| Double Oval | Multivalued Attribute | `Phone_Numbers` |
| Dashed Oval | Derived Attribute | `Age` (from `Birth_Date`) |
| Underlined Oval | Key Attribute | <u>SSN</u> |
| Double Diamond | Identifying Relationship | `DEPENDENTS_OF` |

### Composite Attributes

```
       Name
      /  |  \
   First Middle Last
```

**Example**: `Address` → `(Street, City, Province, Postal_Code)`

---

##  Keys in ER Diagrams

### Key Attribute Rules

> [!important] Key Attributes
> A **key attribute** uniquely identifies an entity within its entity set.

**Properties**:
- Shown by **underlining** the attribute name
- Composite keys are allowed (but must be **minimal**)
- ER diagrams don't choose a primary key — that happens during **ER-to-relational mapping**

**Examples**:
- `EMPLOYEE`: <u>SSN</u> is a key
- `COURSE`: <u>Course_Code</u> is a key
- `ENROLLMENT`: <u>(Student_ID, Course_Code)</u> is a composite key

### Multiple Keys vs No Keys

- Some entity types may have **multiple candidate keys**
- Some entities have **no key at all** → **Weak Entity** (see Section 3.5)

---

## Domains / Value Sets

Every **simple attribute** has a **value set** (domain):
- `INTEGER`, `FLOAT`, `STRING`, `DATE`
- `ENUM('Male', 'Female', 'Other')`
- Custom ranges: `Salary: INTEGER(0..999999)`

> [!note]
> Domains are typically **not drawn** on ER diagrams but are documented separately.

---

## Practice Questions

> [!question] Q1: Identify Components
> In an ER diagram for a university:
> - What entity types would you create?
> - What would be the key attributes?
> - What relationships exist?

<details>
<summary>Answer</summary>

**Entities**: `STUDENT`, `COURSE`, `PROFESSOR`, `DEPARTMENT`

**Keys**: 
- `STUDENT`: <u>Student_ID</u>
- `COURSE`: <u>Course_Code</u>
- `PROFESSOR`: <u>Prof_ID</u>

**Relationships**: 
- `ENROLLS_IN` (STUDENT, COURSE)
- `TEACHES` (PROFESSOR, COURSE)
- `BELONGS_TO` (PROFESSOR, DEPARTMENT)

</details>

---

## Related Topics

- [[03-Relational-Model|Next: Relational Model & Constraints]]
- [[10-ER-to-Relational-Mapping|ER-to-Relational Mapping Rules]]
- [[02b-Relationships|Detailed: Relationship Types & Constraints]]

---

## Tags
#er-diagram #entity #attribute #key #schema #database-design
