# CIS3530 - Database Systems Notes

> Comprehensive study notes for Database Systems course

## Table of Contents

### Core Concepts
- [[01-Introduction|Chapter 1: Introduction to Databases]]
- [[02-ER-Modeling|Chapter 2: ER Modeling & Diagrams]]
- [[03-Relational-Model|Chapter 3: Relational Model & Constraints]]
- [[04-SQL-Fundamentals|Chapter 4: SQL Fundamentals]]

### Theoretical Foundations
- [[05-Relational-Algebra|Chapter 5: Relational Algebra]]
- [[06-Relational-Calculus|Chapter 6: Relational Calculus (TRC)]]
- [[07-Mathematical-Concepts|Chapter 7: Mathematical Concepts]]

### Database Design
- [[08-Functional-Dependencies|Chapter 8: Functional Dependencies]]
- [[09-Normalization|Chapter 9: Normalization (1NF, 2NF, 3NF, BCNF)]]
- [[10-ER-to-Relational-Mapping|Chapter 10: ER-to-Relational Mapping]]

### Query Processing
- [[11-Query-Processing|Chapter 11: Query Processing & Optimization]]
- [[12-Access-Methods|Chapter 12: Access Methods & Indexes]]
- [[13-Join-Algorithms|Chapter 13: Join Algorithms]]

### Transaction Management
- [[14-Transactions-Concurrency|Chapter 14: Transactions & Concurrency Control]]
- [[15-Locking-Protocols|Chapter 15: Locking Protocols (2PL, Deadlocks)]]
- [[16-Recovery|Chapter 16: Recovery & ACID]]

### Security
- [[17-Database-Security|Chapter 17: Database Security (DAC, MAC, RBAC)]]

### Quick References
- [[SQL-Quick-Reference|SQL Quick Reference]]
- [[Normalization-Quick-Guide|Normalization Quick Guide]]
- [[Formulas-Cheatsheet|Formulas & Algorithms Cheatsheet]]
- [[Common-Patterns|Common Exam Patterns]]

---

## Study Tips

> [!tip] Exam Preparation
> - Focus on **normalization** and **FD reasoning**
> - Practice **SQL queries** and **relational algebra** conversions
> - Understand **2PL locking** and **deadlock scenarios**
> - Master **ER-to-relational mapping** rules

## Key Topics 

1. **ER Diagrams**: Cardinality, participation, weak entities
2. **Normalization**: 1NF → 2NF → 3NF → BCNF progression
3. **SQL**: Joins, subqueries, aggregation, HAVING
4. **Relational Algebra**: σ, π, ⨝, ÷ operators
5. **Query Optimization**: Push σ/π down, access path selection
6. **Concurrency**: Strict 2PL, conflict serializability
7. **Recovery**: UNDO/REDO, WAL, checkpoints

---

## Notation Guide

| Symbol | Meaning | Example |
|--------|---------|---------|
| σ | Selection (filter rows) | σ<sub>salary>50000</sub>(EMPLOYEE) |
| π | Projection (select columns) | π<sub>name,dept</sub>(EMPLOYEE) |
| ⨝ | Join | R ⨝<sub>R.id=S.id</sub> S |
| × | Cartesian Product | R × S |
| ∪ | Union | R ∪ S |
| ∩ | Intersection | R ∩ S |
| − | Difference | R − S |
| ÷ | Division | R ÷ S |
| ρ | Rename | ρ<sub>T(A,B)</sub>(R) |

---

*Last Updated: April 2026*
