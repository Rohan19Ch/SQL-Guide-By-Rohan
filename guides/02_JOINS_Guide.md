# SQL JOINS - Complete Masterclass
**Created by: Rohan Choudhary**

[[SQL_Master_Guide|← Back to Master Guide]]

---

## 🤝 Welcome to JOINs - The Most Important SQL Skill!

**Why are JOINs so important?**

Imagine you have two notebooks:
- 📓 Notebook 1: Customer names and their customer ID
- 📓 Notebook 2: Orders with customer ID and order details

**To answer "What did John Smith order?"** you need to:
1. Find John Smith in Notebook 1 → Get his customer ID (say, #42)
2. Look through Notebook 2 → Find all orders with customer ID #42
3. Match them together!

**That's exactly what JOIN does!** It connects information from different tables.

### 🎯 Real-World Analogy

Think of JOIN like:
- 🏪 Matching receipts with customer loyalty cards
- 📚 Matching library books with the person who borrowed them
- 🎬 Matching movies with their actors
- 📦 Matching Amazon orders with customers

**Without JOINs, databases would be useless!** Most real-world questions require combining data from multiple tables.

---

## Table of Contents
1. [[#What is a JOIN?]]
2. [[#INNER JOIN]]
3. [[#LEFT JOIN (LEFT OUTER JOIN)]]
4. [[#RIGHT JOIN (RIGHT OUTER JOIN)]]
5. [[#FULL OUTER JOIN]]
6. [[#CROSS JOIN]]
7. [[#SELF JOIN]]
8. [[#Multiple Table JOINS]]
9. [[#JOIN Conditions and Performance]]
10. [[#Common Mistakes]]
11. [[#Visual Guide]]

---

## What is a JOIN?

A **JOIN** combines rows from two or more tables based on a related column between them. It's how we bring together data that's spread across multiple tables.

### Why Do We Need JOINs?

**The Phone Book Analogy:**

Imagine if every time someone appeared in your contacts, you had to write their FULL address, phone number, email, birthday, etc. 

That would be:
- 😫 Repetitive (write same info many times)
- 💾 Wasteful (uses lots of space)
- 🐛 Error-prone (typos, outdated info)

Instead, you:
1. Keep ONE list of contacts with all their details
2. Keep another list of calls/messages with just their name
3. Match them together when needed!

**That's exactly what databases do!** Data is split into logical tables, and JOIN brings them together.

### 📊 Visual Example: Two Tables

**Table 1: employees** (Who works here?)
```
employee_id | first_name | last_name | department_id
------------|------------|-----------|---------------
1           | John       | Smith     | 10
2           | Jane       | Doe       | 20
3           | Bob        | Johnson   | 10
4           | Alice      | Brown     | 99  ← No matching department!
```

**Table 2: departments** (What are the departments?)
```
department_id | department_name | location
--------------|-----------------|----------
10            | Sales           | New York
20            | Engineering     | San Francisco
30            | HR              | Chicago  ← No employees yet!
```

**The Question:** "Who works in which department?"

**The Problem:** Employee names are in one table, department names in another!

**The Solution:** JOIN them using `department_id` (the common column)!

**🔑 Key Concept:** 
- `department_id` in employees → **Foreign Key** (points to another table)
- `department_id` in departments → **Primary Key** (unique identifier)
- JOINs match these keys together!

---

## INNER JOIN

**INNER JOIN = Only give me matches!**

### 🎯 The Dating App Analogy

Imagine a dating app:
- Table A: People who like coffee ☕
- Table B: People who like hiking 🥾
- INNER JOIN: Show me people who like BOTH coffee AND hiking

**Result?** Only people who appear in BOTH lists!

### Visual Representation
```
Table A          Table B          INNER JOIN Result
-------          -------          -----------------
  A                 B              Only A ∩ B
 ███               ███                  ███
███████           ███████                ███
 ███               ███                   ███
                                   
Left only        Right only        Both sides!
(excluded)       (excluded)        (included!)
```

### 📊 Back to Our Example

**employees table:**
```
employee_id | first_name | department_id
------------|------------|---------------
1           | John       | 10
2           | Jane       | 20
3           | Bob        | 10
4           | Alice      | 99  ← No matching dept!
```

**departments table:**
```
department_id | department_name
--------------|----------------
10            | Sales
20            | Engineering
30            | HR  ← No employees!
```

**INNER JOIN Result:**
```
employee_id | first_name | department_name
------------|------------|----------------
1           | John       | Sales
2           | Jane       | Engineering
3           | Bob        | Sales

❌ Alice excluded (no matching department)
❌ HR excluded (no matching employees)
```

**🎓 Key Point:** INNER JOIN is picky - both sides must match!

### Basic Syntax
```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

### Example 1: Basic INNER JOIN
```sql
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

**Result:**
```
employee_id | first_name | last_name | department_name
------------|------------|-----------|----------------
1           | John       | Smith     | Sales
2           | Jane       | Doe       | Engineering
3           | Bob        | Johnson   | Sales
```

**Notice:** Only employees with a matching department_id are returned. If an employee has `department_id = NULL` or a department_id that doesn't exist in the departments table, they won't appear in the result.

### Example 2: INNER JOIN with WHERE
```sql
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    e.salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 50000
  AND d.location = 'New York';
```

**This finds:** Employees in New York departments earning over $50,000.

### Example 3: INNER JOIN with Multiple Conditions
```sql
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d 
    ON e.department_id = d.department_id 
    AND e.hire_date >= d.established_date;  -- Additional join condition
```

### When to Use INNER JOIN?

✅ **Use INNER JOIN when:**
- You only want rows that have matches in both tables
- You're combining master data (e.g., employee + department)
- Missing relationships should be excluded

❌ **Don't use INNER JOIN when:**
- You need to keep all rows from one table even without matches (use LEFT JOIN)
- You want to find rows that DON'T match (use LEFT JOIN with IS NULL)

---

## LEFT JOIN (LEFT OUTER JOIN)

**LEFT JOIN = Keep everything from the left, match what you can from the right**

### 🎯 The Guest List Analogy

Imagine you're throwing a party:
- **Left table:** Your complete guest list (everyone invited)
- **Right table:** People who RSVP'd "yes"
- **LEFT JOIN:** Show me everyone invited, and mark who confirmed

**Result?** 
- ✅ Everyone on your list appears
- ✅ Those who RSVP'd → Show their details
- ❓ Those who didn't RSVP → Show them anyway, mark as "unknown"

### Visual Representation
```
Table A          Table B          LEFT JOIN Result
-------          -------          ----------------
  A                 B              All of A + matches from B
 ███               ███            ███████
███████           ███████        ███████
 ███               ███            ███████
                                   
All left side    Right side       Left side ALWAYS shows
(ALL included)   (if matches)     Right = NULL if no match
```

### 📊 Back to Our Example

**employees (LEFT table):**
```
employee_id | first_name | department_id
------------|------------|---------------
1           | John       | 10
2           | Jane       | 20
3           | Bob        | NULL      ← No department
4           | Alice      | 99        ← Invalid dept
```

**departments (RIGHT table):**
```
department_id | department_name
--------------|----------------
10            | Sales
20            | Engineering
30            | HR
```

**LEFT JOIN Result:**
```
employee_id | first_name | department_id | department_name
------------|------------|---------------|------------------
1           | John       | 10            | Sales
2           | Jane       | 20            | Engineering
3           | Bob        | NULL          | NULL  ← No match, but Bob still appears!
4           | Alice      | 99            | NULL  ← No match, but Alice still appears!

✅ ALL employees shown (left side complete)
❌ HR department not shown (no employees)
```

### 🎓 When to Use LEFT JOIN?

**Use LEFT JOIN when you want to answer:**
- "Show me ALL customers, and their orders if they have any"
- "Show me ALL products, including ones never sold"
- "Show me ALL students, including ones not enrolled in courses"
- "Who HASN'T done something?" (find the NULLs!)

**The pattern:** "Show me everything from table A, with related info from table B *if it exists*"

### Basic Syntax
```sql
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;
```

### Example 1: Basic LEFT JOIN
```sql
-- Get ALL employees, with their department (if they have one)
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;
```

**Sample Data:**
```
employees:
employee_id | first_name | department_id
------------|------------|---------------
1           | John       | 10
2           | Jane       | 20
3           | Bob        | NULL
4           | Alice      | 99  (doesn't exist in departments)

departments:
department_id | department_name
--------------|----------------
10            | Sales
20            | Engineering
```

**Result:**
```
employee_id | first_name | department_name
------------|------------|------------------
1           | John       | Sales
2           | Jane       | Engineering
3           | Bob        | NULL
4           | Alice      | NULL
```

**Key Point:** Bob and Alice appear in results even though they have no matching department. Their department_name is NULL.

### Example 2: Find Rows Without Matches
A common pattern: Find employees WITHOUT a department.

```sql
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;  -- No match found
```

**Result:**
```
employee_id | first_name | last_name
------------|------------|----------
3           | Bob        | Johnson
4           | Alice      | Brown
```

**This pattern is powerful for finding:**
- Customers with no orders
- Products never sold
- Employees with no assigned projects

### Example 3: LEFT JOIN with Aggregation
```sql
-- Count orders per customer (including customers with 0 orders)
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(o.amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY order_count DESC;
```

**Key Points:**
- `COUNT(o.order_id)` counts non-NULL order_ids (customers with no orders get 0)
- `COUNT(*)` would count all rows (customers with no orders would get 1) - WRONG!
- `COALESCE(SUM(o.amount), 0)` converts NULL to 0 for customers with no orders

### Example 4: Multiple LEFT JOINs
```sql
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    m.first_name AS manager_first_name,
    m.last_name AS manager_last_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

Gets employee info with their department and manager, showing all employees even if they lack a department or manager.

### When to Use LEFT JOIN?

✅ **Use LEFT JOIN when:**
- You want ALL rows from the left table
- You're looking for missing relationships (using IS NULL)
- You're doing aggregations and need to include zero counts
- You want to preserve the "main" entity regardless of related data

**Real-World Examples:**
- All customers (even those with no orders)
- All products (even those never sold)
- All students (even those not enrolled in courses)
- All blog posts (even those with no comments)

---

## RIGHT JOIN (RIGHT OUTER JOIN)

**RIGHT JOIN** returns **ALL** rows from the right table and matching rows from the left table. If there's no match, NULL values are returned for left table columns.

### Visual Representation
```
Table A          Table B          Result
-------          -------          ------
  A                 B              (A ∩ B) + B
 ███               ███                 ███
███████           ███████           ███████
 ███               ███               ███████
```
All of B is included, with matching parts of A.

### Basic Syntax
```sql
SELECT columns
FROM table1
RIGHT JOIN table2 ON table1.column = table2.column;
```

### Example 1: Basic RIGHT JOIN
```sql
-- Get ALL departments, with their employees (if they have any)
SELECT 
    e.first_name,
    e.last_name,
    d.department_id,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;
```

**Result:**
```
first_name | last_name | department_id | department_name
-----------|-----------|---------------|----------------
John       | Smith     | 10            | Sales
Bob        | Johnson   | 10            | Sales
Jane       | Doe       | 20            | Engineering
NULL       | NULL      | 30            | HR  (no employees)
```

**Key Point:** HR department appears even though it has no employees.

### Example 2: Find Departments Without Employees
```sql
SELECT 
    d.department_id,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE e.employee_id IS NULL;
```

**Result:**
```
department_id | department_name
--------------|----------------
30            | HR
```

### RIGHT JOIN vs LEFT JOIN

**Important:** `RIGHT JOIN` is rarely used because you can always rewrite it as a `LEFT JOIN` by swapping the table order.

```sql
-- These are equivalent:
SELECT * FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

SELECT * FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id;
```

**Best Practice:** Stick with `LEFT JOIN` for consistency and readability. Most developers find LEFT JOIN more intuitive.

### When to Use RIGHT JOIN?

⚠️ **Rarely needed** - Can always use LEFT JOIN instead

✅ **Use RIGHT JOIN when:**
- You're adding to an existing query and don't want to reorder tables
- Your organization has a standard of putting the "main" table first

**Recommendation:** Use LEFT JOIN and put your "main" table first for clarity.

---

## FULL OUTER JOIN

**FULL OUTER JOIN** returns **ALL** rows from both tables. When there's a match, it combines them. When there's no match, it includes the row with NULLs for the missing side.

### Visual Representation
```
Table A          Table B          Result
-------          -------          ------
  A                 B              A ∪ B (All of both)
 ███               ███            ███████
███████           ███████        ███████
 ███               ███            ███████
```

### MySQL Note
⚠️ **MySQL does NOT support FULL OUTER JOIN directly!**

But you can simulate it using `UNION`:

```sql
-- Simulate FULL OUTER JOIN
SELECT 
    e.employee_id,
    e.first_name,
    d.department_id,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id

UNION

SELECT 
    e.employee_id,
    e.first_name,
    d.department_id,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;
```

**Result includes:**
- All employees (with or without departments)
- All departments (with or without employees)

### When to Use FULL OUTER JOIN?

✅ **Use when you need:**
- Complete picture of both tables
- To find both unmatched employees AND unmatched departments
- Data reconciliation reports

**Real-World Example:**
```sql
-- Find all customers and all products, showing which combinations exist
SELECT 
    c.customer_name,
    p.product_name,
    o.quantity
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN products p ON o.product_id = p.product_id

UNION

SELECT 
    c.customer_name,
    p.product_name,
    o.quantity
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
RIGHT JOIN products p ON o.product_id = p.product_id;
```

---

## CROSS JOIN

**CROSS JOIN** returns the Cartesian product - every row from the first table combined with every row from the second table.

### Visual Representation
```
Table A (3 rows)    Table B (2 rows)    Result (6 rows)
----------------    ----------------    ---------------
A1                  B1                  A1-B1
A2                  B2                  A1-B2
A3                                      A2-B1
                                        A2-B2
                                        A3-B1
                                        A3-B2
```

### Basic Syntax
```sql
SELECT columns
FROM table1
CROSS JOIN table2;

-- Alternative syntax (same result)
SELECT columns
FROM table1, table2;
```

### Example 1: Basic CROSS JOIN
```sql
SELECT 
    e.first_name,
    d.department_name
FROM employees e
CROSS JOIN departments d;
```

If employees has 3 rows and departments has 4 rows, result has 3 × 4 = 12 rows.

### Example 2: Practical Use - All Combinations
```sql
-- Generate all possible size-color combinations for a product
SELECT 
    s.size_name,
    c.color_name,
    CONCAT(s.size_name, '-', c.color_name) AS sku_variant
FROM sizes s
CROSS JOIN colors c
WHERE s.size_name IN ('S', 'M', 'L', 'XL')
  AND c.color_name IN ('Red', 'Blue', 'Green');
```

**Result:**
```
size_name | color_name | sku_variant
----------|------------|------------
S         | Red        | S-Red
S         | Blue       | S-Blue
S         | Green      | S-Green
M         | Red        | M-Red
M         | Blue       | M-Blue
M         | Green      | M-Green
... (16 total rows)
```

### Example 3: Date Range with Employees
```sql
-- Create a row for each employee for each day in January
WITH RECURSIVE dates AS (
    SELECT DATE('2023-01-01') AS date
    UNION ALL
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM dates
    WHERE date < '2023-01-31'
)
SELECT 
    e.employee_id,
    e.first_name,
    d.date
FROM employees e
CROSS JOIN dates d
ORDER BY e.employee_id, d.date;
```

Related: [[CTE_Guide#Recursive CTEs|Recursive CTEs]]

### ⚠️ Warning: CROSS JOIN Can Be Dangerous!
```sql
-- BAD: Accidentally creating huge result sets
SELECT * 
FROM orders      -- 1 million rows
CROSS JOIN products;  -- 10,000 rows
-- Result: 10 BILLION rows! 🔥
```

**Always be careful with CROSS JOIN on large tables!**

### When to Use CROSS JOIN?

✅ **Use CROSS JOIN when:**
- You need every combination of two sets
- Generating test data
- Creating calendar tables
- Product variant generation

❌ **Don't use CROSS JOIN when:**
- You meant to use INNER JOIN (forgot the ON clause)
- Tables are large (result size = rows1 × rows2)

---

## SELF JOIN

A **SELF JOIN** is when a table is joined with itself. Useful for hierarchical data or comparing rows within the same table.

### Example 1: Employee-Manager Relationship
```sql
-- Find employees with their manager's name
SELECT 
    e.employee_id,
    e.first_name AS employee_name,
    e.manager_id,
    m.first_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

**Data:**
```
employees:
employee_id | first_name | manager_id
------------|------------|------------
1           | Alice      | NULL  (CEO)
2           | Bob        | 1
3           | Charlie    | 1
4           | David      | 2
```

**Result:**
```
employee_id | employee_name | manager_id | manager_name
------------|---------------|------------|-------------
1           | Alice         | NULL       | NULL
2           | Bob           | 1          | Alice
3           | Charlie       | 1          | Alice
4           | David         | 2          | Bob
```

### Example 2: Find Employees in Same Department
```sql
-- Find pairs of employees in the same department
SELECT 
    e1.first_name AS employee1,
    e2.first_name AS employee2,
    e1.department_id
FROM employees e1
INNER JOIN employees e2 
    ON e1.department_id = e2.department_id
    AND e1.employee_id < e2.employee_id  -- Avoid duplicates (A-B and B-A)
ORDER BY e1.department_id;
```

### Example 3: Compare Consecutive Records
```sql
-- Find products with price increases
SELECT 
    p1.product_name,
    p1.price AS old_price,
    p1.effective_date AS old_date,
    p2.price AS new_price,
    p2.effective_date AS new_date,
    (p2.price - p1.price) AS price_increase
FROM price_history p1
INNER JOIN price_history p2 
    ON p1.product_id = p2.product_id
    AND p2.effective_date = (
        SELECT MIN(effective_date)
        FROM price_history
        WHERE product_id = p1.product_id
        AND effective_date > p1.effective_date
    )
WHERE p2.price > p1.price;
```

### When to Use SELF JOIN?

✅ **Use SELF JOIN when:**
- Hierarchical data (employee-manager, category-subcategory)
- Comparing rows in the same table
- Finding duplicates or near-duplicates
- Temporal analysis (comparing current vs previous state)

**Related Topics:** [[CTE_Guide#Recursive CTEs|Recursive CTEs]] can be better for deep hierarchies.

---

## Multiple Table JOINS

You can join as many tables as needed.

### Example 1: Three Table JOIN
```sql
-- Get employee, department, and location info
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    l.city,
    l.country
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id;
```

### Example 2: Complex Multi-Table JOIN
```sql
-- Order details with customer, product, and category info
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    p.product_name,
    cat.category_name,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) AS line_total
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
INNER JOIN categories cat ON p.category_id = cat.category_id
WHERE o.order_date >= '2023-01-01';
```

### Example 3: Mixing JOIN Types
```sql
-- All employees with their departments, managers, and project count
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    m.first_name AS manager_name,
    COUNT(ep.project_id) AS project_count
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
LEFT JOIN employees m ON e.manager_id = m.employee_id
LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
GROUP BY e.employee_id, e.first_name, e.last_name, d.department_name, m.first_name;
```

### Join Order Matters (for readability, not results)
```sql
-- ✅ GOOD: Logical flow
FROM orders
JOIN customers ON ...
JOIN order_items ON ...
JOIN products ON ...

-- ❌ CONFUSING: Random order
FROM products
JOIN order_items ON ...
JOIN customers ON ...
JOIN orders ON ...
```

**Best Practice:** Start with your "main" table and join related tables in a logical order.

---

## JOIN Conditions and Performance

### 1. Always Use Indexed Columns in JOIN Conditions
```sql
-- ✅ GOOD: Joining on indexed foreign key
SELECT * FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- ❌ SLOW: Joining on non-indexed column
SELECT * FROM employees e
INNER JOIN departments d ON e.email = d.contact_email;
```

### 2. Multiple Join Conditions
```sql
-- Join on multiple columns
SELECT * FROM sales s
INNER JOIN targets t 
    ON s.employee_id = t.employee_id
    AND s.year = t.year
    AND s.quarter = t.quarter;
```

### 3. Filter Early
```sql
-- ✅ GOOD: Filter before joining
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN (
    SELECT * FROM departments WHERE location = 'New York'
) d ON e.department_id = d.department_id
WHERE e.status = 'Active';

-- ❌ SLOWER: Filter after joining
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE e.status = 'Active' AND d.location = 'New York';
```

Actually, modern MySQL optimizers handle both well, but the first is clearer with [[CTE_Guide|CTEs]]:

```sql
WITH ny_departments AS (
    SELECT * FROM departments WHERE location = 'New York'
)
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN ny_departments d ON e.department_id = d.department_id
WHERE e.status = 'Active';
```

### 4. USING Clause (when column names match)
```sql
-- Instead of:
SELECT * FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- You can use:
SELECT * FROM employees
INNER JOIN departments USING (department_id);
```

**Note:** Only works when join columns have the same name in both tables.

---

## Common Mistakes

### Mistake 1: Forgetting JOIN Condition (Accidental CROSS JOIN)
```sql
-- ❌ WRONG: Missing ON clause creates CROSS JOIN
SELECT * FROM employees, departments;

-- ✅ CORRECT: Include ON clause
SELECT * FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

### Mistake 2: Using COUNT(*) with LEFT JOIN
```sql
-- ❌ WRONG: Counts rows even for customers with no orders
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(*) AS order_count  -- WRONG!
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

-- ✅ CORRECT: Count only non-NULL orders
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count  -- CORRECT!
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

### Mistake 3: Wrong JOIN Type
```sql
-- ❌ WRONG: Using INNER JOIN when you need all customers
SELECT c.customer_name, COUNT(o.order_id) AS order_count
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id  -- Missing customers with 0 orders!
GROUP BY c.customer_id, c.customer_name;

-- ✅ CORRECT: Use LEFT JOIN
SELECT c.customer_name, COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

### Mistake 4: Not Handling NULLs from LEFT JOIN
```sql
-- ❌ WRONG: NULL arithmetic gives NULL
SELECT 
    e.first_name,
    e.salary + b.bonus AS total_compensation  -- NULL if no bonus!
FROM employees e
LEFT JOIN bonuses b ON e.employee_id = b.employee_id;

-- ✅ CORRECT: Use COALESCE
SELECT 
    e.first_name,
    e.salary + COALESCE(b.bonus, 0) AS total_compensation
FROM employees e
LEFT JOIN bonuses b ON e.employee_id = b.employee_id;
```

### Mistake 5: Ambiguous Column Names
```sql
-- ❌ WRONG: Column exists in both tables
SELECT employee_id, first_name, department_name
FROM employees
INNER JOIN departments ON employees.department_id = departments.department_id;
-- Error: Column 'employee_id' is ambiguous (if it exists in both)

-- ✅ CORRECT: Use table aliases
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

### Mistake 6: Cartesian Product from Multiple LEFT JOINs
```sql
-- ⚠️ CAREFUL: Can create unexpected duplicates
SELECT 
    e.first_name,
    p.project_name,
    c.certification_name
FROM employees e
LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
LEFT JOIN projects p ON ep.project_id = p.project_id
LEFT JOIN employee_certifications ec ON e.employee_id = ec.employee_id
LEFT JOIN certifications c ON ec.certification_id = c.certification_id;
```

If an employee has 3 projects and 2 certifications, they'll appear 6 times!

**Solution:** Separate queries or aggregate:
```sql
-- Better: Aggregate projects
WITH employee_projects_agg AS (
    SELECT 
        e.employee_id,
        GROUP_CONCAT(p.project_name) AS projects
    FROM employees e
    LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
    LEFT JOIN projects p ON ep.project_id = p.project_id
    GROUP BY e.employee_id
),
employee_certs_agg AS (
    SELECT 
        e.employee_id,
        GROUP_CONCAT(c.certification_name) AS certifications
    FROM employees e
    LEFT JOIN employee_certifications ec ON e.employee_id = ec.employee_id
    LEFT JOIN certifications c ON ec.certification_id = c.certification_id
    GROUP BY e.employee_id
)
SELECT 
    e.first_name,
    e.last_name,
    ep.projects,
    ec.certifications
FROM employees e
LEFT JOIN employee_projects_agg ep ON e.employee_id = ep.employee_id
LEFT JOIN employee_certs_agg ec ON e.employee_id = ec.employee_id;
```

---

## Visual Guide

### INNER JOIN
```
Employees            Departments          Result
-----------          -------------        -----------
ID  Dept             ID   Name            Emp  Dept
1   10               10   Sales           1    Sales
2   20               20   Eng             2    Eng
3   NULL             30   HR              
4   99 (orphan)

Only rows 1 and 2 appear (have matching departments)
```

### LEFT JOIN
```
Employees            Departments          Result
-----------          -------------        --------------
ID  Dept             ID   Name            Emp  Dept Name
1   10               10   Sales           1    10   Sales
2   20               20   Eng             2    20   Eng
3   NULL             30   HR              3    NULL NULL
4   99               -                    4    99   NULL

All employees appear. NULL for non-matching departments.
```

### RIGHT JOIN
```
Employees            Departments          Result
-----------          -------------        ----------------
ID  Dept             ID   Name            Emp  Dept  Name
1   10               10   Sales           1    10    Sales
2   20               20   Eng             2    20    Eng
3   NULL             30   HR              NULL 30    HR

All departments appear. NULL for HR (no employees).
```

### CROSS JOIN
```
Sizes      Colors       Result
------     ------       -----------------
S          Red          S - Red
M          Blue         S - Blue
L                       M - Red
                        M - Blue
                        L - Red
                        L - Blue

Every size × every color
```

---

## Real-World Examples

### Example 1: Sales Report with Multiple JOINs
```sql
SELECT 
    DATE_FORMAT(o.order_date, '%Y-%m') AS month,
    c.customer_name,
    p.product_name,
    SUM(oi.quantity) AS total_quantity,
    SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY month, c.customer_name, p.product_name
HAVING total_revenue > 1000
ORDER BY month, total_revenue DESC;
```

### Example 2: Finding Orphaned Records
```sql
-- Find orders with invalid customer IDs
SELECT o.order_id, o.customer_id, o.order_date
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;

-- Find products never ordered
SELECT p.product_id, p.product_name
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE oi.product_id IS NULL;
```

### Example 3: Hierarchy Reporting (Self-Join + Aggregation)
```sql
-- Count direct reports per manager
SELECT 
    m.employee_id AS manager_id,
    m.first_name AS manager_name,
    COUNT(e.employee_id) AS direct_reports
FROM employees m
LEFT JOIN employees e ON m.employee_id = e.manager_id
GROUP BY m.employee_id, m.first_name
HAVING direct_reports > 0
ORDER BY direct_reports DESC;
```

---

## Performance Tips

1. **Index foreign key columns** used in JOINs
2. **Filter early** with WHERE before joining large tables
3. **Use EXPLAIN** to analyze query execution
   ```sql
   EXPLAIN SELECT * FROM employees e
   INNER JOIN departments d ON e.department_id = d.department_id;
   ```
4. **Avoid SELECT *** - specify only needed columns
5. **Consider [[CTE_Guide|CTEs]]** for complex multi-table queries
6. **Be careful with LEFT JOIN** - can't be optimized as well as INNER JOIN

---

## Summary

| JOIN Type | Returns | Use Case |
|-----------|---------|----------|
| **INNER JOIN** | Only matching rows from both tables | Combining related data where both sides must exist |
| **LEFT JOIN** | All from left + matching from right | Keep all records from main table, even without matches |
| **RIGHT JOIN** | All from right + matching from left | Rarely used (use LEFT JOIN instead) |
| **FULL OUTER JOIN** | All from both tables | Find unmatched records in both tables (MySQL needs UNION) |
| **CROSS JOIN** | Every combination (A × B) | Generate all combinations |
| **SELF JOIN** | Table joined with itself | Hierarchies, comparing rows in same table |

### Quick Decision Tree
```
Do you need ALL rows from one table?
├─ YES: Use LEFT JOIN (or RIGHT JOIN)
│   └─ Put the "keep all" table on the left
└─ NO: Do you need matching rows only?
    ├─ YES: Use INNER JOIN
    └─ NO: Do you need all combinations?
        └─ YES: Use CROSS JOIN
```

---

## Next Steps

- Apply JOINs in complex [[SELECT_Complete_Guide|SELECT queries]]
- Simplify multi-table queries with [[CTE_Guide|CTEs]]
- Use JOINs with [[Window_Functions_Guide|Window Functions]] for advanced analytics
- Review [[SQL_Best_Practices|Best Practices]] for efficient queries

[[SQL_Master_Guide|← Back to Master Guide]]

