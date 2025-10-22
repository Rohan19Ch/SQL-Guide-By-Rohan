# 📁 Repository Structure

This SQL HelpSheet is organized for **progressive learning** - files are numbered in the order you should learn them!

## 🗂️ Folder Structure

```
SQL_HelpSheet/
│
├── 📄 README.md                    ← Start here! Overview & learning paths
├── 🗺️ SQL_Master_Guide.md          ← Navigation hub with visual learning paths
├── 📋 STRUCTURE.md                 ← This file (explains organization)
│
└── 📁 guides/                      ← All learning guides (numbered in sequence)
    ├── 01_SELECT_Complete_Guide.md       ⭐ START HERE - Query basics
    ├── 02_JOINS_Guide.md                 🔗 Combine tables (most important!)
    ├── 03_CTE_Guide.md                   📝 Organize complex queries
    ├── 04_Window_Functions_Guide.md      📊 Advanced analytics
    └── 05_SQL_Best_Practices.md          ⚡ Write professional code
```

## 📚 Learning Sequence

The guides are numbered to show the **recommended learning order**:

### 01. SELECT - Complete Guide
**Start here!** Learn to read and filter data
- Basic SELECT syntax
- WHERE, ORDER BY, LIMIT
- Aggregation functions (COUNT, SUM, AVG)
- GROUP BY and HAVING
- Subqueries

**Time to learn:** 1-2 weeks for beginners

---

### 02. JOINS - Masterclass
**Most important SQL skill!** Combine data from multiple tables
- INNER JOIN (matching records)
- LEFT JOIN (keep all from left)
- RIGHT JOIN (keep all from right)
- Visual diagrams & real-world examples

**Time to learn:** 1-2 weeks

---

### 03. CTE - Common Table Expressions
**Organize complex queries** into readable steps
- Break down complex logic
- Reusable temporary results
- Recursive CTEs for hierarchies

**Time to learn:** 3-5 days

---

### 04. Window Functions & PARTITION BY
**Advanced analytics** without losing rows
- ROW_NUMBER, RANK, DENSE_RANK
- PARTITION BY for grouped calculations
- LAG, LEAD for comparisons
- Running totals and moving averages

**Time to learn:** 1 week

---

### 05. Best Practices & Anti-Patterns
**Write like a professional** from day one
- Security (SQL injection prevention)
- Performance optimization
- Common mistakes to avoid
- Production-ready code standards

**Time to learn:** Ongoing reference

---

## 🎯 Quick Navigation Paths

### Absolute Beginner Path
```
README.md → 01_SELECT → 02_JOINS → Practice!
```

### Intermediate Path
```
01_SELECT → 02_JOINS → 03_CTE → 04_Window_Functions
```

### Advanced/Interview Prep
```
All guides + focus on:
- JOIN types comparison
- Window functions comparison
- Performance optimization
- CTEs vs Subqueries
```

## 🔗 How Links Work

### Internal Links (Obsidian Wiki-style)
Within the guides, you'll see links like:
- `[[guides/01_SELECT_Complete_Guide|SELECT Guide]]` - Links to other guides
- `[[guides/02_JOINS_Guide#INNER-JOIN|INNER JOIN]]` - Links to specific sections

### Markdown Links (GitHub/Web)
In README.md, standard markdown links work:
- `[SELECT Guide](guides/01_SELECT_Complete_Guide.md)` - GitHub-compatible

## 📊 File Naming Convention

All guide files follow this pattern:
```
XX_TOPIC_Guide.md
│  │     │
│  │     └── Always ends with "_Guide.md"
│  └──────── Topic in Title_Case
└──────────── Two-digit sequence number (01-05)
```

**Benefits:**
- ✅ Sorts alphabetically in order
- ✅ Clear sequence in Git
- ✅ Easy to add new guides between existing ones
- ✅ Visual learning progression

## 🔄 Git History

Files are tracked with their numbers, so git history shows:
```
guides/01_SELECT_Complete_Guide.md
guides/02_JOINS_Guide.md
guides/03_CTE_Guide.md
guides/04_Window_Functions_Guide.md
guides/05_SQL_Best_Practices.md
```

This makes it easy to:
- See the learning sequence
- Track changes to specific topics
- Know which guide comes before/after

## 🎓 For Teachers/Trainers

If using this for teaching:

1. **Week 1-2:** Guide 01 (SELECT)
2. **Week 3-4:** Guide 02 (JOINS)
3. **Week 5:** Guide 03 (CTEs)
4. **Week 6-7:** Guide 04 (Window Functions)
5. **Ongoing:** Guide 05 (Best Practices)

Students can follow the numbers and won't get lost!

## 💡 Tips

- **Don't skip numbers!** Each builds on the previous
- **Master basics first** - Guide 01 before anything else
- **JOINs are crucial** - Spend time on Guide 02
- **Use Master Guide** for quick reference navigation
- **Bookmark frequently used sections** in your favorites

---

**Questions about the structure?** Check [README.md](README.md) or [SQL_Master_Guide.md](SQL_Master_Guide.md)

---

Made with ❤️ for SQL learners everywhere

