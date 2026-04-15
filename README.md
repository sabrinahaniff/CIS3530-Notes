# CIS3530 Database Systems - Professional Study Notes

## About These Notes

These notes cover a full Database Systems course (CIS3530) with topics including:
- ER modeling and database design
- SQL and relational algebra
- Normalization and functional dependencies
- Query processing and optimization
- Transaction management and concurrency control
- Recovery and security

**Format**: Professional markdown notes optimized for [Obsidian](https://obsidian.md/) with internal linking, callouts, and organization.

---

## Structure

### Core Topics
- **Chapters 1-4**: Database fundamentals, ER modeling, SQL basics
- **Chapters 5-7**: Relational algebra, calculus, mathematical foundations
- **Chapters 8-10**: Functional dependencies, normalization, ER-to-relational mapping
- **Chapters 11-13**: Query processing, optimization, access methods
- **Chapters 14-17**: Transactions, concurrency, recovery, security

### Quick Reference
- **Formulas Cheatsheet**: Quick lookup for all major formulas and algorithms
- **SQL Quick Reference**: Common SQL patterns and syntax
- **Normalization Quick Guide**: Step-by-step normalization rules

---

## How to Use

### Option 1: Obsidian (Recommended)

1. **Install Obsidian**: Download from [obsidian.md](https://obsidian.md/)
2. **Open as Vault**: File → Open folder as vault → Select `database-notes` folder
3. **Start with README.md**: This is your main index with links to all chapters
4. **Navigate**: Click internal links like `[[Chapter-Name]]` to jump between topics

**Benefits**:
- Graph view to see connections between topics
- Quick switching between related concepts
- Search across all notes instantly
- Theme customization

### Option 2: Any Markdown Viewer

These notes are plain markdown and work in:
- **VS Code** (with markdown preview)
- **GitHub** (renders automatically)
- **Typora**, **Mark Text**, or any markdown editor

### Option 3: Convert to Other Formats

Use [Pandoc](https://pandoc.org/) to convert to PDF, HTML, or DOCX:

```bash
# Convert a chapter to PDF
pandoc 09-Normalization.md -o Normalization.pdf

# Convert all to HTML
for file in *.md; do pandoc "$file" -o "${file%.md}.html"; done
```

---

## Study Paths

### For Deep Understanding (Full Course)
1. Follow the chapter order in [[README]]
2. Do practice problems at the end of each chapter
3. Draw your own examples
4. Teach concepts to someone else (Feynman technique)

### For Quick Lookup
- Jump to [[Formulas-Cheatsheet]] for specific formulas
- Use search (`Ctrl/Cmd + O` in Obsidian) for specific terms
- Check the tags at the bottom of each chapter

---

## Key Features

### Interactive Elements
- **Collapsible answers** for practice problems
- **Internal links** between related topics
- **Tags** for easy searching
- **Examples** with both correct and incorrect approaches

### Exam-Focused
- **Common mistake warnings**
- **Decision trees** for algorithm selection
- **Quick reference tables**
- **Practice problems with solutions**

---

##  Contributing

Found an error or want to add examples?

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

**Contribution ideas**:
- Add more practice problems
- Create visual diagrams (Mermaid or ASCII art)
- Add real-world examples
- Expand on specific topics
- Fix typos or improve explanations

---

## Notes on Notation

### Relational Algebra Symbols

Due to markdown limitations, we use standard notation:
- `σ` = Selection (sigma)
- `π` = Projection (pi)
- `ρ` = Rename (rho)
- `⋈` = Join
- `×` = Product
- `∪` = Union
- `∩` = Intersection
- `−` = Difference
- `÷` = Division

In Obsidian, these render properly. In plain text editors, they may appear as Unicode symbols.

---

##  Maintenance

These notes are maintained as part of course study materials. Updates include:
- New practice problems
- Clarifications based on common questions
- Additional examples
- Links to external resources

**Last Updated**: April 2026

---

##  Recommended Resources

### Books
- *Database System Concepts* by Silberschatz, Korth, Sudarshan
- *Fundamentals of Database Systems* by Elmasri, Navathe

### Online Resources
- [SQL Tutorial - W3Schools](https://www.w3schools.com/sql/)
- [Stanford DB Course](https://cs.stanford.edu/people/widom/DB-mooc.html)
- [CMU Database Systems Course](https://15445.courses.cs.cmu.edu/)

### Practice
- [SQLZoo](https://sqlzoo.net/) - Interactive SQL practice
- [LeetCode Database](https://leetcode.com/problemset/database/) - SQL problems
- [HackerRank SQL](https://www.hackerrank.com/domains/sql) - Structured practice

---

##  License

These notes are provided for educational purposes. Feel free to use, share, and modify for your own learning.

**Attribution**: If you share or build upon these notes, a link back to this repository is appreciated!

---

##  Star This Repo

If these notes helped you, consider starring the repository to help others find it!
