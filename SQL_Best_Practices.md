# SQL Best Practices & Anti-Patterns
**Created by: Rohan Choudhary**

[[SQL_Master_Guide|← Back to Master Guide]]

---

## Table of Contents
1. [[#Writing Readable Queries]]
2. [[#Performance Optimization]]
3. [[#Common Anti-Patterns]]
4. [[#Security Best Practices]]
5. [[#Data Integrity]]
6. [[#Indexing Guidelines]]
7. [[#Query Optimization Techniques]]
8. [[#Troubleshooting Slow Queries]]
9. [[#Production-Ready Code]]

---

## Writing Readable Queries

### 1. Use Consistent Formatting

**✅ GOOD: Formatted and readable**
```sql
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    e.salary
FROM employees e
INNER JOIN departments d 
    ON e.department_id = d.department_id
WHERE e.status = 'Active'
    AND e.hire_date >= '2020-01-01'
    AND d.location IN ('New York', 'San Francisco')
ORDER BY d.department_name, e.last_name;
```

**❌ BAD: Hard to read**
```sql
select e.employee_id,e.first_name,e.last_name,d.department_name,e.salary from employees e inner join departments d on e.department_id=d.department_id where e.status='Active' and e.hire_date>='2020-01-01' and d.location in ('New York','San Francisco') order by d.department_name,e.last_name;
```

### 2. Use Meaningful Aliases

**✅ GOOD: Clear aliases**
```sql
SELECT 
    emp.first_name,
    mgr.first_name AS manager_name,
    dept.department_name
FROM employees emp
LEFT JOIN employees mgr ON emp.manager_id = mgr.employee_id
INNER JOIN departments dept ON emp.department_id = dept.department_id;
```

**❌ BAD: Cryptic aliases**
```sql
SELECT a.first_name, b.first_name, c.department_name
FROM employees a
LEFT JOIN employees b ON a.manager_id = b.employee_id
INNER JOIN departments c ON a.department_id = c.department_id;
```

### 3. Break Complex Queries with CTEs

**✅ GOOD: Readable with CTEs**
```sql
WITH active_employees AS (
    SELECT employee_id, first_name, last_name, department_id, salary
    FROM employees
    WHERE status = 'Active'
),
department_averages AS (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary
    FROM active_employees
    GROUP BY department_id
)
SELECT 
    ae.first_name,
    ae.last_name,
    ae.salary,
    da.avg_salary,
    ae.salary - da.avg_salary AS diff_from_avg
FROM active_employees ae
JOIN department_averages da ON ae.department_id = da.department_id
WHERE ae.salary > da.avg_salary;
```

Related: [[CTE_Guide|Common Table Expressions]]

**❌ BAD: Nested subqueries**
```sql
SELECT 
    ae.first_name,
    ae.last_name,
    ae.salary,
    da.avg_salary,
    ae.salary - da.avg_salary AS diff_from_avg
FROM (
    SELECT employee_id, first_name, last_name, department_id, salary
    FROM employees
    WHERE status = 'Active'
) ae
JOIN (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary
    FROM (
        SELECT employee_id, department_id, salary
        FROM employees
        WHERE status = 'Active'
    ) sub
    GROUP BY department_id
) da ON ae.department_id = da.department_id
WHERE ae.salary > da.avg_salary;
```

### 4. Add Comments for Complex Logic

```sql
-- Calculate customer lifetime value with recency weighting
-- Recent purchases count more than older ones
WITH customer_purchases AS (
    SELECT 
        customer_id,
        order_date,
        amount,
        -- Decay factor: 1.0 for current month, decreasing for older months
        amount * POW(0.9, TIMESTAMPDIFF(MONTH, order_date, CURDATE())) AS weighted_amount
    FROM orders
    WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 12 MONTH)
)
SELECT 
    customer_id,
    SUM(amount) AS total_spent,
    SUM(weighted_amount) AS weighted_lifetime_value
FROM customer_purchases
GROUP BY customer_id;
```

### 5. Capitalize SQL Keywords

**✅ GOOD: Keywords capitalized**
```sql
SELECT first_name, last_name
FROM employees
WHERE status = 'Active'
ORDER BY last_name;
```

**❌ BAD: Inconsistent case**
```sql
select first_name, last_name
From employees
WHERE status = 'Active'
order by last_name;
```

---

## Performance Optimization

### 1. Select Only Needed Columns

**✅ GOOD: Specific columns**
```sql
SELECT employee_id, first_name, last_name, email
FROM employees
WHERE department_id = 10;
```

**❌ BAD: SELECT ***
```sql
SELECT *
FROM employees
WHERE department_id = 10;
```

**Why?** 
- Wastes bandwidth and memory
- Breaks applications if table structure changes
- Prevents covering indexes

### 2. Filter Early and Often

**✅ GOOD: Filter before joining**
```sql
WITH recent_orders AS (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE order_date >= '2023-01-01'
)
SELECT 
    c.customer_name,
    SUM(ro.amount) AS total_spent
FROM customers c
INNER JOIN recent_orders ro ON c.customer_id = ro.customer_id
GROUP BY c.customer_id, c.customer_name;
```

**❌ BAD: Filter after joining**
```sql
SELECT 
    c.customer_name,
    SUM(o.amount) AS total_spent
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2023-01-01'
GROUP BY c.customer_id, c.customer_name;
```

While modern optimizers often handle both cases, explicit early filtering makes intent clearer.

### 3. Use EXISTS Instead of IN for Subqueries

**✅ GOOD: EXISTS (stops at first match)**
```sql
SELECT customer_id, customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.order_date >= '2023-01-01'
);
```

**❌ BAD: IN (processes all results)**
```sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    WHERE order_date >= '2023-01-01'
);
```

**When to use IN:** When subquery returns small, distinct list.
**When to use EXISTS:** When checking for existence with potentially many matches.

### 4. Avoid Functions on Indexed Columns in WHERE

**✅ GOOD: Use index**
```sql
SELECT * FROM employees
WHERE hire_date >= '2023-01-01'
    AND hire_date < '2024-01-01';
```

**❌ BAD: Function prevents index use**
```sql
SELECT * FROM employees
WHERE YEAR(hire_date) = 2023;  -- Can't use index on hire_date!
```

**More Examples:**
```sql
-- ❌ BAD
WHERE LOWER(email) = 'john@example.com'
WHERE SUBSTRING(phone, 1, 3) = '555'
WHERE salary * 1.1 > 50000

-- ✅ GOOD
WHERE email = 'john@example.com'  -- Assumes email stored lowercase
WHERE phone LIKE '555%'
WHERE salary > 50000 / 1.1
```

### 5. Limit Result Sets

**✅ GOOD: Use LIMIT**
```sql
SELECT employee_id, first_name, last_name
FROM employees
ORDER BY employee_id
LIMIT 100;
```

**✅ GOOD: Use pagination properly**
```sql
-- Page 1
SELECT * FROM products ORDER BY product_id LIMIT 20 OFFSET 0;

-- Page 2
SELECT * FROM products ORDER BY product_id LIMIT 20 OFFSET 20;
```

**⚠️ WARNING: Avoid large OFFSETs**
```sql
-- ❌ SLOW: MySQL scans through 1 million rows
SELECT * FROM products ORDER BY product_id LIMIT 20 OFFSET 1000000;

-- ✅ BETTER: Use keyset pagination
SELECT * FROM products 
WHERE product_id > 1000000  -- Last seen ID
ORDER BY product_id 
LIMIT 20;
```

### 6. Use JOIN Instead of Multiple Queries

**✅ GOOD: One query with JOIN**
```sql
SELECT 
    e.employee_id,
    e.first_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

**❌ BAD: Multiple queries in application code**
```python
# Fetching in Python (N+1 query problem)
employees = execute("SELECT * FROM employees")
for emp in employees:
    dept = execute(f"SELECT * FROM departments WHERE id = {emp.dept_id}")
    print(f"{emp.name} - {dept.name}")
```

Related: [[JOINS_Guide|SQL JOINs]]

---

## Common Anti-Patterns

### 1. SELECT * in Production

**Problem:** Wastes resources, breaks code when schema changes, prevents optimization.

**❌ ANTI-PATTERN**
```sql
SELECT * FROM employees;
```

**✅ SOLUTION**
```sql
SELECT employee_id, first_name, last_name, email, hire_date
FROM employees;
```

### 2. Using OR in WHERE with Different Columns

**Problem:** Can't use indexes efficiently.

**❌ ANTI-PATTERN**
```sql
SELECT * FROM employees
WHERE department_id = 10 OR salary > 100000;
```

**✅ SOLUTION: Use UNION**
```sql
SELECT * FROM employees WHERE department_id = 10
UNION
SELECT * FROM employees WHERE salary > 100000;
```

Or if you need duplicates:
```sql
SELECT * FROM employees WHERE department_id = 10
UNION ALL
SELECT * FROM employees WHERE salary > 100000 AND department_id != 10;
```

### 3. NOT IN with NULLs

**Problem:** `NOT IN` with NULL values returns empty set.

**❌ ANTI-PATTERN**
```sql
-- If subquery returns any NULL, result is empty!
SELECT * FROM employees
WHERE employee_id NOT IN (
    SELECT manager_id FROM employees  -- Contains NULLs!
);
```

**✅ SOLUTION: Use NOT EXISTS**
```sql
SELECT * FROM employees e1
WHERE NOT EXISTS (
    SELECT 1 FROM employees e2
    WHERE e2.manager_id = e1.employee_id
);
```

Or filter NULLs:
```sql
SELECT * FROM employees
WHERE employee_id NOT IN (
    SELECT manager_id 
    FROM employees 
    WHERE manager_id IS NOT NULL
);
```

### 4. Using HAVING for Non-Aggregate Filters

**Problem:** Less efficient than WHERE.

**❌ ANTI-PATTERN**
```sql
SELECT department_id, COUNT(*) AS emp_count
FROM employees
GROUP BY department_id
HAVING status = 'Active';  -- status is not aggregated!
```

**✅ SOLUTION: Use WHERE**
```sql
SELECT department_id, COUNT(*) AS emp_count
FROM employees
WHERE status = 'Active'
GROUP BY department_id;
```

**Remember:**
- `WHERE` filters **rows** (before grouping)
- `HAVING` filters **groups** (after aggregation)

Related: [[SELECT_Complete_Guide#GROUP BY and HAVING|GROUP BY and HAVING]]

### 5. Correlated Subqueries in SELECT

**Problem:** Executes once per row (N+1 problem).

**❌ ANTI-PATTERN**
```sql
SELECT 
    e.first_name,
    e.salary,
    (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id) AS dept_avg
FROM employees e;
```

**✅ SOLUTION: Use window function or JOIN**
```sql
-- Option 1: Window function
SELECT 
    first_name,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
FROM employees;

-- Option 2: JOIN with aggregation
SELECT 
    e.first_name,
    e.salary,
    da.dept_avg
FROM employees e
INNER JOIN (
    SELECT department_id, AVG(salary) AS dept_avg
    FROM employees
    GROUP BY department_id
) da ON e.department_id = da.department_id;
```

Related: [[Window_Functions_Guide|Window Functions]]

### 6. Implicit Type Conversion

**Problem:** Prevents index use, causes unexpected behavior.

**❌ ANTI-PATTERN**
```sql
-- employee_id is INT, but comparing to string
SELECT * FROM employees WHERE employee_id = '123';

-- hire_date is DATE, but comparing to number
SELECT * FROM employees WHERE hire_date = 20230101;
```

**✅ SOLUTION: Use correct types**
```sql
SELECT * FROM employees WHERE employee_id = 123;
SELECT * FROM employees WHERE hire_date = '2023-01-01';
```

### 7. DISTINCT as a Band-Aid

**Problem:** Using DISTINCT to hide duplicate issues from bad joins.

**❌ ANTI-PATTERN**
```sql
SELECT DISTINCT e.employee_id, e.first_name
FROM employees e
INNER JOIN employee_projects ep ON e.employee_id = ep.employee_id;
-- DISTINCT hides that employees with multiple projects appear multiple times
```

**✅ SOLUTION: Fix the query logic**
```sql
-- If you want employees with projects (unique)
SELECT e.employee_id, e.first_name
FROM employees e
WHERE EXISTS (
    SELECT 1 FROM employee_projects ep
    WHERE ep.employee_id = e.employee_id
);

-- If you want employees with project count
SELECT e.employee_id, e.first_name, COUNT(ep.project_id) AS project_count
FROM employees e
LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
GROUP BY e.employee_id, e.first_name;
```

### 8. Storing Comma-Separated Values

**Problem:** Violates normalization, can't index, hard to query.

**❌ ANTI-PATTERN**
```sql
-- employees table
employee_id | skills
------------|------------------
1           | 'Java,Python,SQL'
2           | 'Python,JavaScript'

-- Querying is a nightmare
SELECT * FROM employees 
WHERE skills LIKE '%Python%';  -- Also matches "DjangoPython"!
```

**✅ SOLUTION: Normalize with junction table**
```sql
-- employees table
employee_id | name
------------|-------
1           | John
2           | Jane

-- skills table
skill_id | skill_name
---------|------------
1        | Java
2        | Python
3        | SQL

-- employee_skills junction table
employee_id | skill_id
------------|----------
1           | 1
1           | 2
1           | 3
2           | 2

-- Easy to query
SELECT e.name, s.skill_name
FROM employees e
INNER JOIN employee_skills es ON e.employee_id = es.employee_id
INNER JOIN skills s ON es.skill_id = s.skill_id
WHERE s.skill_name = 'Python';
```

---

## Security Best Practices

### 1. Never Concatenate User Input (SQL Injection)

**❌ DANGEROUS: SQL Injection vulnerability**
```python
# Python example
user_id = request.GET['user_id']
query = f"SELECT * FROM users WHERE user_id = {user_id}"
execute(query)

# If user_id = "1 OR 1=1", query becomes:
# SELECT * FROM users WHERE user_id = 1 OR 1=1
# Returns ALL users!
```

**✅ SAFE: Use parameterized queries**
```python
# Python with parameterized query
user_id = request.GET['user_id']
query = "SELECT * FROM users WHERE user_id = ?"
execute(query, (user_id,))
```

```java
// Java with PreparedStatement
String userId = request.getParameter("userId");
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE user_id = ?"
);
stmt.setString(1, userId);
ResultSet rs = stmt.executeQuery();
```

### 2. Principle of Least Privilege

**✅ GOOD: Grant minimum necessary permissions**
```sql
-- Create read-only user for reporting
CREATE USER 'report_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT SELECT ON database_name.* TO 'report_user'@'localhost';

-- Create application user with limited access
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT SELECT, INSERT, UPDATE ON database_name.orders TO 'app_user'@'localhost';
GRANT SELECT ON database_name.products TO 'app_user'@'localhost';
```

**❌ BAD: Overly permissive**
```sql
GRANT ALL PRIVILEGES ON *.* TO 'app_user'@'localhost';
```

### 3. Protect Sensitive Data

**✅ GOOD: Hash passwords, encrypt sensitive data**
```sql
-- Never store plain text passwords
INSERT INTO users (username, password_hash)
VALUES ('john', SHA2('user_password', 256));

-- Use application-level encryption for sensitive data
-- Example in application code:
encrypted_ssn = encrypt(ssn, encryption_key)
INSERT INTO employees (name, ssn_encrypted) VALUES ('John', encrypted_ssn)
```

**❌ BAD: Storing plain text**
```sql
INSERT INTO users (username, password)
VALUES ('john', 'mypassword123');  -- NEVER DO THIS!
```

---

## Data Integrity

### 1. Use Transactions for Related Operations

**✅ GOOD: Atomic operations**
```sql
START TRANSACTION;

-- Transfer money between accounts
UPDATE accounts 
SET balance = balance - 100 
WHERE account_id = 1;

UPDATE accounts 
SET balance = balance + 100 
WHERE account_id = 2;

-- Only commit if both succeed
COMMIT;

-- If error occurs, rollback
-- ROLLBACK;
```

**❌ BAD: No transaction**
```sql
-- If second query fails, first already executed!
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
```

### 2. Use Foreign Keys for Referential Integrity

**✅ GOOD: Foreign keys enforce relationships**
```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department_id INT,
    FOREIGN KEY (department_id) 
        REFERENCES departments(department_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

**Benefits:**
- Can't insert employee with invalid department_id
- Can't delete department that has employees (with RESTRICT)
- Updates cascade automatically

### 3. Use NOT NULL Where Appropriate

**✅ GOOD: Enforce required fields**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20),  -- Optional
    hire_date DATE NOT NULL
);
```

### 4. Use CHECK Constraints

**✅ GOOD: Enforce business rules**
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INT NOT NULL,
    CONSTRAINT chk_price CHECK (price > 0),
    CONSTRAINT chk_stock CHECK (stock_quantity >= 0)
);
```

---

## Indexing Guidelines

### 1. Index Foreign Keys

**✅ GOOD: Index columns used in JOINs**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department_id INT,
    manager_id INT,
    INDEX idx_department (department_id),
    INDEX idx_manager (manager_id)
);
```

### 2. Index Columns Used in WHERE, ORDER BY, GROUP BY

**✅ GOOD: Index frequently queried columns**
```sql
-- Frequently used queries:
-- SELECT * FROM orders WHERE customer_id = ? ORDER BY order_date DESC
-- SELECT customer_id, COUNT(*) FROM orders WHERE status = 'Pending' GROUP BY customer_id

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    status VARCHAR(20),
    amount DECIMAL(10, 2),
    INDEX idx_customer_date (customer_id, order_date),
    INDEX idx_status (status)
);
```

### 3. Use Composite Indexes Wisely

**✅ GOOD: Column order matters**
```sql
-- Query: WHERE status = 'Active' AND department_id = 10
CREATE INDEX idx_status_dept ON employees(status, department_id);

-- This index can be used for:
-- WHERE status = 'Active'
-- WHERE status = 'Active' AND department_id = 10

-- But NOT efficiently for:
-- WHERE department_id = 10 (only uses first column)
```

**Rule of Thumb:** Most selective columns first, or columns used in equality first.

### 4. Don't Over-Index

**❌ BAD: Too many indexes**
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10, 2),
    stock INT,
    INDEX idx_name (product_name),
    INDEX idx_category (category),
    INDEX idx_price (price),
    INDEX idx_stock (stock),
    INDEX idx_name_category (product_name, category),
    INDEX idx_name_price (product_name, price),
    INDEX idx_category_price (category, price)
    -- Too many indexes slow down writes!
);
```

**Balance:**
- Indexes speed up reads
- Indexes slow down writes (INSERT, UPDATE, DELETE)
- Each index uses disk space

### 5. Cover Queries with Covering Indexes

**✅ GOOD: Covering index includes all needed columns**
```sql
-- Query: SELECT customer_id, order_date, amount 
--        FROM orders WHERE status = 'Shipped'

-- Covering index (query doesn't need to access table)
CREATE INDEX idx_status_covering 
ON orders(status, customer_id, order_date, amount);
```

---

## Query Optimization Techniques

### 1. Use EXPLAIN to Analyze Queries

```sql
EXPLAIN SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 50000;
```

**Look for:**
- **type**: ALL (table scan) is bad, ref/eq_ref/const is good
- **rows**: Number of rows examined
- **Extra**: "Using filesort" or "Using temporary" indicates inefficiency

### 2. Rewrite Subqueries as JOINs When Possible

**❌ SLOWER: Subquery**
```sql
SELECT *
FROM products
WHERE category_id IN (
    SELECT category_id 
    FROM categories 
    WHERE parent_category = 'Electronics'
);
```

**✅ FASTER: JOIN**
```sql
SELECT p.*
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
WHERE c.parent_category = 'Electronics';
```

### 3. Use UNION ALL Instead of UNION When Possible

**✅ FASTER: UNION ALL (no deduplication)**
```sql
SELECT customer_id, order_date FROM orders_2023
UNION ALL
SELECT customer_id, order_date FROM orders_2024;
```

**❌ SLOWER: UNION (deduplicates)**
```sql
SELECT customer_id, order_date FROM orders_2023
UNION
SELECT customer_id, order_date FROM orders_2024;
```

Use `UNION` only when you need to remove duplicates.

### 4. Optimize COUNT Queries

**❌ SLOW: COUNT(*) on large tables**
```sql
SELECT COUNT(*) FROM orders;  -- Scans entire table
```

**✅ FASTER: Approximate count**
```sql
-- For InnoDB tables
SELECT TABLE_ROWS 
FROM information_schema.TABLES 
WHERE TABLE_SCHEMA = 'your_database' 
AND TABLE_NAME = 'orders';
-- Approximate, but instant
```

**✅ FASTER: Maintain count in separate table**
```sql
-- Update count table on INSERT/DELETE
CREATE TABLE table_counts (
    table_name VARCHAR(50) PRIMARY KEY,
    row_count INT
);
```

---

## Troubleshooting Slow Queries

### 1. Identify Slow Queries

**Enable Slow Query Log:**
```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- Log queries taking > 2 seconds
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow-query.log';
```

### 2. Check Current Running Queries

```sql
SHOW PROCESSLIST;
-- Or for more detail:
SELECT * FROM information_schema.PROCESSLIST
WHERE TIME > 10  -- Running for > 10 seconds
ORDER BY TIME DESC;
```

### 3. Kill Long-Running Query

```sql
KILL QUERY process_id;  -- Stops the query
KILL process_id;        -- Kills the connection
```

### 4. Analyze Table Statistics

```sql
-- Update table statistics for optimizer
ANALYZE TABLE employees;

-- Check index usage
SHOW INDEX FROM employees;
```

### 5. Common Causes of Slow Queries

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| Full table scan | Missing index | Add index on WHERE/JOIN columns |
| Using filesort | No index on ORDER BY | Add index on ORDER BY columns |
| Using temporary | Complex GROUP BY | Simplify query, add indexes |
| Large OFFSET | Pagination issue | Use keyset pagination |
| Nested subqueries | Correlated subquery | Use JOIN or window function |
| Many rows examined | Too broad WHERE | Add more selective filters |

---

## Production-Ready Code

### 1. Handle NULLs Explicitly

**✅ GOOD: Explicit NULL handling**
```sql
SELECT 
    first_name,
    COALESCE(middle_name, '') AS middle_name,  -- Empty string if NULL
    last_name,
    COALESCE(phone, 'N/A') AS phone,
    IFNULL(bonus, 0) AS bonus  -- 0 if NULL
FROM employees;
```

### 2. Use Appropriate Data Types

**✅ GOOD: Right-sized types**
```sql
CREATE TABLE products (
    product_id INT UNSIGNED,           -- 0 to 4 billion
    sku VARCHAR(20),                   -- Not VARCHAR(255) if max is 20
    price DECIMAL(10, 2),              -- Not FLOAT for money
    is_active BOOLEAN,                 -- Not VARCHAR(10)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    stock_quantity SMALLINT UNSIGNED   -- 0 to 65535
);
```

**❌ BAD: Wasteful types**
```sql
CREATE TABLE products (
    product_id VARCHAR(255),  -- Waste of space if numeric
    price FLOAT,              -- Rounding errors with money!
    is_active VARCHAR(10),    -- Waste of space
    created_at VARCHAR(50)    -- Can't do date math
);
```

### 3. Test with Realistic Data Volume

```sql
-- Don't just test with 10 rows, test with millions
-- Generate test data
INSERT INTO orders (customer_id, order_date, amount)
SELECT 
    FLOOR(1 + RAND() * 10000) AS customer_id,
    DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 1460) DAY) AS order_date,
    ROUND(10 + RAND() * 990, 2) AS amount
FROM 
    (SELECT 1 UNION SELECT 2 UNION SELECT 3) t1,
    (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4) t2,
    (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t3;
-- Generates thousands of test rows
```

### 4. Document Complex Queries

```sql
/**
 * Customer Lifetime Value Calculation
 * 
 * Calculates weighted LTV based on recency of purchases.
 * Recent purchases have higher weight (decay factor 0.9 per month).
 * 
 * Author: Rohan Choudhary
 * Date: 2024-01-15
 * Dependencies: orders, customers tables
 */
WITH weighted_purchases AS (
    SELECT 
        customer_id,
        amount,
        amount * POW(0.9, TIMESTAMPDIFF(MONTH, order_date, CURDATE())) AS weighted_amount
    FROM orders
    WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 12 MONTH)
)
SELECT 
    c.customer_id,
    c.customer_name,
    COALESCE(SUM(wp.weighted_amount), 0) AS lifetime_value
FROM customers c
LEFT JOIN weighted_purchases wp ON c.customer_id = wp.customer_id
GROUP BY c.customer_id, c.customer_name;
```

### 5. Version Control Your Schema Changes

```sql
-- migration_001_create_employees.sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- migration_002_add_email_to_employees.sql
ALTER TABLE employees 
ADD COLUMN email VARCHAR(100) NOT NULL UNIQUE AFTER last_name;

-- migration_003_create_index_employee_name.sql
CREATE INDEX idx_employee_name ON employees(last_name, first_name);
```

---

## Quick Reference Checklist

### Before Running in Production:

- [ ] Tested with production-sized data
- [ ] Used EXPLAIN to verify query plan
- [ ] Avoided SELECT *
- [ ] Used appropriate indexes
- [ ] No functions on indexed columns in WHERE
- [ ] Used parameterized queries (no SQL injection)
- [ ] Handled NULLs explicitly
- [ ] Used appropriate data types
- [ ] Added appropriate constraints (NOT NULL, FOREIGN KEY, CHECK)
- [ ] Used transactions where needed
- [ ] Added comments for complex logic
- [ ] Formatted for readability
- [ ] Tested error cases

### Performance Warning Signs:

- ⚠️ EXPLAIN shows type = ALL (full table scan)
- ⚠️ EXPLAIN shows "Using filesort" or "Using temporary"
- ⚠️ Examining millions of rows to return hundreds
- ⚠️ Query time increases non-linearly with data size
- ⚠️ Using DISTINCT to fix duplicate problems
- ⚠️ Correlated subqueries in SELECT
- ⚠️ Large OFFSET in pagination
- ⚠️ No WHERE clause on large tables

---

## Summary

### Golden Rules:

1. **Readability First** - Code is read more than written
2. **Select What You Need** - Avoid SELECT *
3. **Filter Early** - WHERE before JOIN when possible
4. **Index Wisely** - Balance read vs write performance
5. **Use the Right Tool** - CTEs, window functions, JOINs each have their place
6. **Security Always** - Never concatenate user input
7. **Test at Scale** - 10 rows != 10 million rows
8. **Explain Everything** - Use EXPLAIN for all important queries

### Learning Resources in This Guide:

- [[SELECT_Complete_Guide|Master SELECT queries]]
- [[CTE_Guide|Simplify with CTEs]]
- [[JOINS_Guide|Combine tables effectively]]
- [[Window_Functions_Guide|Advanced analytics]]

---

**Remember:** Good SQL is like good code - it should be correct, efficient, secure, and maintainable. When in doubt, optimize for readability first, then performance.

[[SQL_Master_Guide|← Back to Master Guide]]

