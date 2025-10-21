# SELECT - The Complete Guide
**Created by: Rohan Choudhary**

[[SQL_Master_Guide|← Back to Master Guide]]

---

## Table of Contents
1. [[#Basic SELECT Syntax]]
2. [[#WHERE Clause - Filtering Data]]
3. [[#ORDER BY - Sorting Results]]
4. [[#LIMIT and OFFSET - Pagination]]
5. [[#Aggregation Functions]]
6. [[#GROUP BY and HAVING]]
7. [[#DISTINCT]]
8. [[#Subqueries]]
9. [[#CASE Expressions]]
10. [[#Common Mistakes to Avoid]]

---

## Basic SELECT Syntax

### The Simplest Form
```sql
-- Select all columns from a table
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary 
FROM employees;

-- Select with column aliases
SELECT 
    first_name AS 'First Name',
    last_name AS 'Last Name',
    salary AS 'Annual Salary'
FROM employees;
```

### ✅ DO: Use explicit column names in production
```sql
SELECT employee_id, first_name, last_name, email
FROM employees;
```

### ❌ DON'T: Use SELECT * in production code
```sql
-- Avoid this - it's slow and pulls unnecessary data
SELECT * FROM employees;
```

**Why?** `SELECT *` fetches all columns, even those you don't need, wasting bandwidth and memory.

---

## WHERE Clause - Filtering Data

The `WHERE` clause filters rows based on conditions.

### Basic Comparison Operators
```sql
-- Equality
SELECT * FROM employees WHERE department = 'Sales';

-- Inequality
SELECT * FROM employees WHERE salary != 50000;
SELECT * FROM employees WHERE salary <> 50000;  -- Same as above

-- Greater than, less than
SELECT * FROM employees WHERE salary > 60000;
SELECT * FROM employees WHERE age < 30;

-- Greater/Less than or equal
SELECT * FROM employees WHERE salary >= 50000;
SELECT * FROM employees WHERE hire_date <= '2023-01-01';
```

### Logical Operators (AND, OR, NOT)
```sql
-- AND - Both conditions must be true
SELECT * FROM employees 
WHERE department = 'Sales' AND salary > 50000;

-- OR - At least one condition must be true
SELECT * FROM employees 
WHERE department = 'Sales' OR department = 'Marketing';

-- NOT - Negates a condition
SELECT * FROM employees 
WHERE NOT department = 'Sales';

-- Complex combinations
SELECT * FROM employees 
WHERE (department = 'Sales' OR department = 'Marketing')
  AND salary > 50000
  AND hire_date >= '2020-01-01';
```

### IN Operator
```sql
-- Instead of multiple ORs
SELECT * FROM employees 
WHERE department IN ('Sales', 'Marketing', 'Engineering');

-- With subquery
SELECT * FROM employees 
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE budget > 100000
);

-- NOT IN
SELECT * FROM employees 
WHERE department NOT IN ('HR', 'Legal');
```

### BETWEEN Operator
```sql
-- Inclusive range
SELECT * FROM employees 
WHERE salary BETWEEN 50000 AND 80000;

-- Date ranges
SELECT * FROM orders 
WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';

-- NOT BETWEEN
SELECT * FROM employees 
WHERE salary NOT BETWEEN 40000 AND 60000;
```

### LIKE Operator (Pattern Matching)
```sql
-- % matches any sequence of characters
SELECT * FROM employees WHERE first_name LIKE 'John%';  -- Starts with 'John'
SELECT * FROM employees WHERE first_name LIKE '%son';   -- Ends with 'son'
SELECT * FROM employees WHERE first_name LIKE '%ann%';  -- Contains 'ann'

-- _ matches exactly one character
SELECT * FROM employees WHERE phone LIKE '555-____-1234';

-- Case insensitivity (MySQL default)
SELECT * FROM employees WHERE email LIKE '%@GMAIL.COM';  -- Finds @gmail.com too

-- NOT LIKE
SELECT * FROM employees WHERE email NOT LIKE '%@company.com';
```

### IS NULL and IS NOT NULL
```sql
-- Find NULL values
SELECT * FROM employees WHERE middle_name IS NULL;

-- Find non-NULL values
SELECT * FROM employees WHERE manager_id IS NOT NULL;

-- ❌ DON'T: Use = NULL (doesn't work!)
SELECT * FROM employees WHERE middle_name = NULL;  -- WRONG!

-- ✅ DO: Use IS NULL
SELECT * FROM employees WHERE middle_name IS NULL;  -- CORRECT!
```

**Why?** `NULL` represents "unknown", and you can't compare with unknown using `=`.

---

## ORDER BY - Sorting Results

### Basic Sorting
```sql
-- Ascending order (default)
SELECT * FROM employees ORDER BY salary;
SELECT * FROM employees ORDER BY salary ASC;  -- Same as above

-- Descending order
SELECT * FROM employees ORDER BY salary DESC;

-- Multiple columns
SELECT * FROM employees 
ORDER BY department ASC, salary DESC;
-- First sorts by department A-Z, then within each department by salary high-to-low
```

### Advanced Sorting
```sql
-- Sort by column position (not recommended)
SELECT first_name, last_name, salary 
FROM employees 
ORDER BY 3 DESC;  -- Orders by 3rd column (salary)

-- Sort by calculated values
SELECT first_name, last_name, (salary * 1.1) AS new_salary
FROM employees 
ORDER BY new_salary DESC;

-- Sort with NULL handling
SELECT * FROM employees 
ORDER BY commission_pct DESC NULLS LAST;  -- NULLs at the end

-- Sort by CASE expression
SELECT * FROM employees 
ORDER BY 
    CASE department
        WHEN 'Executive' THEN 1
        WHEN 'Management' THEN 2
        ELSE 3
    END,
    salary DESC;
```

### ✅ DO: Use meaningful ordering
```sql
SELECT * FROM employees 
ORDER BY hire_date DESC, last_name ASC;
```

### ❌ DON'T: Rely on default ordering
```sql
-- Without ORDER BY, row order is unpredictable
SELECT * FROM employees;  -- Order not guaranteed!
```

---

## LIMIT and OFFSET - Pagination

### Basic LIMIT
```sql
-- Get first 10 rows
SELECT * FROM employees LIMIT 10;

-- Get top 5 highest paid employees
SELECT * FROM employees 
ORDER BY salary DESC 
LIMIT 5;
```

### LIMIT with OFFSET
```sql
-- Skip first 10 rows, get next 10
SELECT * FROM employees LIMIT 10 OFFSET 10;

-- Alternative syntax
SELECT * FROM employees LIMIT 10, 10;  -- LIMIT offset, count

-- Pagination example
-- Page 1 (rows 1-20)
SELECT * FROM employees ORDER BY employee_id LIMIT 20 OFFSET 0;

-- Page 2 (rows 21-40)
SELECT * FROM employees ORDER BY employee_id LIMIT 20 OFFSET 20;

-- Page 3 (rows 41-60)
SELECT * FROM employees ORDER BY employee_id LIMIT 20 OFFSET 40;
```

### ✅ DO: Always use ORDER BY with LIMIT
```sql
SELECT * FROM employees 
ORDER BY employee_id 
LIMIT 10;
```

### ❌ DON'T: Use LIMIT without ORDER BY
```sql
-- Results will be unpredictable
SELECT * FROM employees LIMIT 10;  -- Which 10 rows?
```

---

## Aggregation Functions

Aggregation functions perform calculations on multiple rows and return a single value.

### COUNT
```sql
-- Count all rows
SELECT COUNT(*) FROM employees;

-- Count non-NULL values
SELECT COUNT(commission_pct) FROM employees;

-- Count distinct values
SELECT COUNT(DISTINCT department) FROM employees;

-- Count with condition
SELECT COUNT(*) FROM employees WHERE salary > 50000;
```

### SUM
```sql
-- Total salary
SELECT SUM(salary) FROM employees;

-- Total with filter
SELECT SUM(salary) FROM employees WHERE department = 'Sales';

-- ❌ DON'T: Sum NULL values are ignored
SELECT SUM(commission_pct) FROM employees;  -- NULLs ignored, not counted as 0
```

### AVG (Average)
```sql
-- Average salary
SELECT AVG(salary) FROM employees;

-- Average with rounding
SELECT ROUND(AVG(salary), 2) FROM employees;

-- Note: NULLs are excluded from average calculation
SELECT AVG(commission_pct) FROM employees;  -- Only counts non-NULL values
```

### MIN and MAX
```sql
-- Minimum and maximum salary
SELECT MIN(salary), MAX(salary) FROM employees;

-- Works with dates
SELECT MIN(hire_date), MAX(hire_date) FROM employees;

-- Works with text (alphabetical)
SELECT MIN(last_name), MAX(last_name) FROM employees;
```

### Multiple Aggregations
```sql
SELECT 
    COUNT(*) AS total_employees,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    SUM(salary) AS total_payroll
FROM employees;
```

---

## GROUP BY and HAVING

### GROUP BY Basics
```sql
-- Count employees per department
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- Average salary per department
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

-- Multiple grouping columns
SELECT department, job_title, COUNT(*) AS count
FROM employees
GROUP BY department, job_title;
```

### GROUP BY Rules

**✅ DO: Include all non-aggregated columns in GROUP BY**
```sql
SELECT department, job_title, AVG(salary)
FROM employees
GROUP BY department, job_title;  -- Both non-aggregated columns listed
```

**❌ DON'T: Select non-aggregated columns not in GROUP BY**
```sql
-- This will cause an error or unexpected results
SELECT department, job_title, AVG(salary)
FROM employees
GROUP BY department;  -- Missing job_title!
```

### HAVING Clause

`HAVING` filters groups (used with `GROUP BY`), while `WHERE` filters individual rows.

```sql
-- Departments with more than 10 employees
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;

-- Departments with average salary > 60000
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 60000;

-- Combined WHERE and HAVING
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= '2020-01-01'  -- Filter rows first
GROUP BY department
HAVING AVG(salary) > 50000;      -- Then filter groups
```

### WHERE vs HAVING

```sql
-- ✅ DO: Use WHERE to filter rows before grouping
SELECT department, COUNT(*) AS count
FROM employees
WHERE status = 'Active'  -- Filter before grouping (efficient)
GROUP BY department;

-- ❌ DON'T: Use HAVING for row-level filters
SELECT department, COUNT(*) AS count
FROM employees
GROUP BY department
HAVING status = 'Active';  -- WRONG! status is not aggregated
```

**Rule of Thumb:**
- `WHERE` filters **rows** (before grouping)
- `HAVING` filters **groups** (after grouping)

---

## DISTINCT

Removes duplicate rows from results.

```sql
-- Get unique departments
SELECT DISTINCT department FROM employees;

-- Multiple columns - unique combinations
SELECT DISTINCT department, job_title FROM employees;

-- DISTINCT with COUNT
SELECT COUNT(DISTINCT department) FROM employees;
```

### ✅ DO: Use DISTINCT when you need unique values
```sql
SELECT DISTINCT country FROM customers;
```

### ❌ DON'T: Use DISTINCT as a "fix" for bad joins
```sql
-- If you get duplicates, fix the join, don't just add DISTINCT
SELECT DISTINCT e.employee_id, e.first_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

---

## Subqueries

A query nested inside another query.

### Subquery in WHERE Clause
```sql
-- Employees earning more than average
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Employees in departments with budget > 100000
SELECT * FROM employees
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE budget > 100000
);
```

### Subquery in FROM Clause (Derived Table)
```sql
-- Average of department averages
SELECT AVG(dept_avg_salary) AS company_wide_avg
FROM (
    SELECT department, AVG(salary) AS dept_avg_salary
    FROM employees
    GROUP BY department
) AS dept_averages;

-- Must have an alias!
```

### Subquery in SELECT Clause (Scalar Subquery)
```sql
-- Compare each employee's salary to company average
SELECT 
    first_name,
    last_name,
    salary,
    (SELECT AVG(salary) FROM employees) AS company_avg,
    salary - (SELECT AVG(salary) FROM employees) AS difference
FROM employees;
```

### Correlated Subquery
```sql
-- Employees earning more than their department's average
SELECT e1.first_name, e1.last_name, e1.salary, e1.department
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department  -- Correlated!
);
```

### EXISTS and NOT EXISTS
```sql
-- Departments that have employees
SELECT * FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e 
    WHERE e.department_id = d.department_id
);

-- Departments with no employees
SELECT * FROM departments d
WHERE NOT EXISTS (
    SELECT 1 FROM employees e 
    WHERE e.department_id = d.department_id
);
```

**Related Topics:** [[CTE_Guide|CTEs]] can often replace complex subqueries for better readability.

---

## CASE Expressions

Conditional logic in SQL (like if-else statements).

### Simple CASE
```sql
SELECT 
    first_name,
    last_name,
    department,
    CASE department
        WHEN 'Sales' THEN 'Revenue Generator'
        WHEN 'Engineering' THEN 'Product Builder'
        WHEN 'HR' THEN 'People Operations'
        ELSE 'Other'
    END AS department_category
FROM employees;
```

### Searched CASE
```sql
SELECT 
    first_name,
    last_name,
    salary,
    CASE 
        WHEN salary < 40000 THEN 'Entry Level'
        WHEN salary BETWEEN 40000 AND 70000 THEN 'Mid Level'
        WHEN salary BETWEEN 70001 AND 100000 THEN 'Senior Level'
        ELSE 'Executive Level'
    END AS salary_grade
FROM employees;
```

### CASE in Aggregation
```sql
-- Count employees by salary range
SELECT 
    COUNT(CASE WHEN salary < 50000 THEN 1 END) AS low_salary,
    COUNT(CASE WHEN salary BETWEEN 50000 AND 80000 THEN 1 END) AS mid_salary,
    COUNT(CASE WHEN salary > 80000 THEN 1 END) AS high_salary
FROM employees;

-- Pivot-like behavior
SELECT 
    department,
    SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS male_count,
    SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female_count
FROM employees
GROUP BY department;
```

### CASE in ORDER BY
```sql
-- Custom sort order
SELECT * FROM employees
ORDER BY 
    CASE department
        WHEN 'Executive' THEN 1
        WHEN 'Management' THEN 2
        WHEN 'Engineering' THEN 3
        ELSE 4
    END;
```

---

## Common Mistakes to Avoid

### 1. Using SELECT * in Production
```sql
-- ❌ DON'T
SELECT * FROM huge_table;

-- ✅ DO
SELECT id, name, email FROM huge_table;
```

### 2. Not Using WHERE When Needed
```sql
-- ❌ DON'T: Pull all data and filter in application
SELECT * FROM orders;  -- Then filter in Python/Java/etc.

-- ✅ DO: Filter in database
SELECT * FROM orders WHERE order_date >= '2023-01-01';
```

### 3. Using = NULL Instead of IS NULL
```sql
-- ❌ DON'T
WHERE column = NULL

-- ✅ DO
WHERE column IS NULL
```

### 4. Not Using ORDER BY with LIMIT
```sql
-- ❌ DON'T: Unpredictable results
SELECT * FROM employees LIMIT 10;

-- ✅ DO
SELECT * FROM employees ORDER BY employee_id LIMIT 10;
```

### 5. Selecting Non-Grouped Columns
```sql
-- ❌ DON'T
SELECT department, job_title, COUNT(*)
FROM employees
GROUP BY department;  -- job_title not grouped!

-- ✅ DO
SELECT department, job_title, COUNT(*)
FROM employees
GROUP BY department, job_title;
```

### 6. Using HAVING for Row-Level Filters
```sql
-- ❌ DON'T
SELECT department, COUNT(*) 
FROM employees 
GROUP BY department
HAVING hire_date > '2020-01-01';  -- WRONG!

-- ✅ DO
SELECT department, COUNT(*) 
FROM employees 
WHERE hire_date > '2020-01-01'  -- Filter rows first
GROUP BY department;
```

---

## Query Execution Order

Understanding the order helps you write better queries:

```
1. FROM          -- Get the table(s)
2. WHERE         -- Filter rows
3. GROUP BY      -- Group rows
4. HAVING        -- Filter groups
5. SELECT        -- Choose columns
6. DISTINCT      -- Remove duplicates
7. ORDER BY      -- Sort results
8. LIMIT/OFFSET  -- Limit results
```

**Example:**
```sql
SELECT department, AVG(salary) AS avg_salary  -- 5. Select & calculate
FROM employees                                 -- 1. From table
WHERE status = 'Active'                        -- 2. Filter rows
GROUP BY department                            -- 3. Group
HAVING AVG(salary) > 60000                     -- 4. Filter groups
ORDER BY avg_salary DESC                       -- 7. Sort
LIMIT 5;                                       -- 8. Limit
```

---

## Next Steps

- Learn how to simplify complex queries with [[CTE_Guide|Common Table Expressions (CTEs)]]
- Combine data from multiple tables using [[JOINS_Guide|SQL JOINs]]
- Master advanced analytics with [[Window_Functions_Guide|Window Functions]]
- Review [[SQL_Best_Practices|Best Practices]] for production-ready queries

[[SQL_Master_Guide|← Back to Master Guide]]

