# MySQL Master Guide
**Created by: Rohan Choudhary**

---

## 👋 Welcome, Beginner!

**First time here?** You're in the right place! This guide will teach you SQL from the ground up, assuming you've never written a database query before.

### 🎯 What is SQL?

**SQL** (Structured Query Language) is how you talk to databases. Think of it as:
- 🔍 **Google search** - but for your company's data
- 📊 **Excel formulas** - but way more powerful
- 📚 **Library card catalog** - but instant and automated

### 🗺️ Navigation Hub

This interconnected guide is designed to take you from complete beginner to proficient in MySQL queries. Each topic builds on the previous ones, so follow the learning path below!

### 📚 Core Concepts (In Learning Order)

1. **[[guides/01_SELECT_Complete_Guide|01. SELECT - The Foundation]]**
   - Every possible SELECT syntax combination
   - Filtering, sorting, grouping, and aggregation
   - Subqueries and derived tables

2. **[[guides/02_JOINS_Guide|02. SQL JOINS Masterclass]]**
   - INNER JOIN - Finding matching records
   - LEFT JOIN - Keeping all left-side records
   - RIGHT JOIN - Keeping all right-side records
   - Visual diagrams and practical examples

3. **[[guides/03_CTE_Guide|03. Common Table Expressions (WITH CTE)]]**
   - What are CTEs and when to use them
   - Simple vs Recursive CTEs
   - Performance considerations

4. **[[guides/04_Window_Functions_Guide|04. Window Functions & PARTITION BY]]**
   - RANK, DENSE_RANK, ROW_NUMBER explained
   - PARTITION BY in detail
   - Real-world use cases

5. **[[guides/05_SQL_Best_Practices|05. Best Practices & Anti-Patterns]]**
   - How to write efficient queries
   - Common mistakes to avoid
   - Performance optimization tips

---

## 🎯 Quick Reference

### When to Use What?

- **Need to filter rows?** → See [[guides/01_SELECT_Complete_Guide#WHERE Clause|WHERE Clause]]
- **Need temporary result sets?** → See [[guides/03_CTE_Guide|CTEs]]
- **Combining tables?** → See [[guides/02_JOINS_Guide|JOINs]]
- **Ranking or numbering rows?** → See [[guides/04_Window_Functions_Guide|Window Functions]]

---

## 📖 Your Visual Learning Path

```
🌱 Week 1-2: Foundation
┌─────────────────────────────────────┐
│  SELECT Basics                      │ ← Start here!
│  "How do I read data?"              │
│                                     │
│  ↓                                  │
│  WHERE Filtering                    │
│  "How do I find specific data?"     │
│                                     │
│  ↓                                  │
│  ORDER BY & LIMIT                   │
│  "How do I organize results?"       │
└─────────────────────────────────────┘

🌿 Week 3-4: Combining Data
┌─────────────────────────────────────┐
│  INNER JOIN                         │ ← Most important!
│  "How do I connect tables?"         │
│                                     │
│  ↓                                  │
│  Aggregation (COUNT, SUM, AVG)      │
│  "How do I calculate totals?"       │
│                                     │
│  ↓                                  │
│  GROUP BY                           │
│  "How do I summarize by category?"  │
└─────────────────────────────────────┘

🌳 Month 2: Advanced Techniques
┌─────────────────────────────────────┐
│  LEFT JOIN                          │
│  "How do I find missing data?"      │
│                                     │
│  ↓                                  │
│  CTEs (WITH clause)                 │
│  "How do I organize complex logic?" │
│                                     │
│  ↓                                  │
│  Window Functions                   │
│  "How do I rank and analyze?"       │
└─────────────────────────────────────┘

🚀 Month 3+: Mastery
┌─────────────────────────────────────┐
│  Advanced Window Functions          │
│  Recursive CTEs                     │
│  Performance Optimization           │
│  Security Best Practices            │
└─────────────────────────────────────┘
```

### 🎯 Quick Start Paths

#### 🚀 **"I Need Results Fast!"** (2-3 hours)
Perfect if you have an urgent task:
1. [[guides/01_SELECT_Complete_Guide#Basic SELECT Syntax|Basic SELECT]] (30 min)
2. [[guides/01_SELECT_Complete_Guide#WHERE Clause|WHERE Filtering]] (30 min)
3. [[guides/02_JOINS_Guide#INNER JOIN|INNER JOIN]] (60 min)
4. [[guides/01_SELECT_Complete_Guide#Aggregation Functions|COUNT & SUM]] (30 min)

#### 📚 **"I Want to Master SQL"** (4-6 weeks)
Comprehensive learning:
1. Week 1: [[guides/01_SELECT_Complete_Guide|01. SELECT Complete Guide]]
2. Week 2: [[guides/02_JOINS_Guide|02. SQL JOINs Masterclass]]
3. Week 3-4: [[guides/03_CTE_Guide|03. CTEs]] and [[guides/04_Window_Functions_Guide|04. Window Functions]]
4. Week 5-6: [[guides/05_SQL_Best_Practices|05. Best Practices]] and practice projects

#### 💼 **"Interview Prep"** (1-2 weeks)
Focus on commonly tested topics:
1. [[guides/02_JOINS_Guide|All JOIN Types]] - Know the differences!
2. [[guides/04_Window_Functions_Guide#Comparison ROW_NUMBER vs RANK vs DENSE_RANK|RANK vs ROW_NUMBER vs DENSE_RANK]]
3. [[guides/03_CTE_Guide#CTEs vs Subqueries vs Temp Tables|CTEs vs Subqueries]]
4. [[guides/05_SQL_Best_Practices#Performance Optimization|Optimization Techniques]]

---

## 💡 About This Guide

This knowledge base is designed for:
- **Quick Reference**: Jump directly to what you need
- **Learning**: Follow the structured learning paths
- **Practice**: Each section includes practical examples
- **Graph Navigation**: Use Obsidian's graph view to explore connections

**Author**: Rohan Choudhary  
**Last Updated**: October 2025

---

### Getting Started

Click on any link above to dive into a specific topic. Each page is interconnected, allowing you to follow your curiosity and build a complete understanding of MySQL.

