# Common Table Expressions (CTE) - WITH Clause
**Created by: Rohan Choudhary**

[[SQL_Master_Guide|← Back to Master Guide]]

---

## 📝 CTEs: Breaking Complex Problems Into Simple Steps!

**Ever tried to solve a math problem in your head that's too complex?** You grab paper and break it into steps:
1. First, calculate this part
2. Then, use that result for the next part
3. Finally, combine everything

**That's exactly what CTEs do for SQL!**

### 🎯 The Recipe Analogy

Imagine baking a cake:

**Without CTEs (Do everything at once):**
```
Mix flour+sugar+eggs+butter while preheating oven to 350 
while greasing pan while checking if you have vanilla...
← Confusing! Too many things at once!
```

**With CTEs (Step by step):**
```
Step 1: Prepare dry ingredients (flour, sugar)
Step 2: Prepare wet ingredients (eggs, butter)  
Step 3: Combine dry and wet
Step 4: Pour into greased pan
Step 5: Bake
← Clear! One step at a time!
```

### 💡 What Problem Do CTEs Solve?

**Bad (Nested Subqueries):**
```sql
SELECT * FROM (
    SELECT * FROM (
        SELECT * FROM orders WHERE amount > 100
    ) WHERE customer_id IN (...)
) WHERE order_date...
-- 😵 Hard to read! What's happening?
```

**Good (With CTE):**
```sql
WITH large_orders AS (
    SELECT * FROM orders WHERE amount > 100
),
priority_customers AS (
    SELECT * FROM customers WHERE vip = true
)
SELECT * FROM large_orders
JOIN priority_customers ON...
-- 😊 Clear! Each step has a name!
```

---

## Table of Contents
1. [[#What is a CTE?]]
2. [[#Basic CTE Syntax]]
3. [[#When to Use CTEs]]
4. [[#Simple CTE Examples]]
5. [[#Multiple CTEs]]
6. [[#Recursive CTEs]]
7. [[#CTEs vs Subqueries vs Temp Tables]]
8. [[#Performance Considerations]]
9. [[#Common Mistakes]]

---

## What is a CTE?

A **Common Table Expression (CTE)** is a temporary named result set that exists only during the execution of a single query. 

### 🎯 Think of CTE as a Post-It Note

You're working on a big calculation:
1. You calculate something and write it on a post-it note
2. You give the post-it a name: "Step 1 Results"
3. You use that post-it to do the next calculation
4. When you're done with the whole thing, the post-it gets thrown away

**That's a CTE!**
- Temporary (only exists during one query)
- Named (you give it a meaningful name)
- Reusable (can use it multiple times in the same query)
- Disposable (disappears after the query finishes)

### Key Characteristics:
- ✅ Defined using the `WITH` keyword (like saying "WITH this temporary result...")
- ✅ Exists only for the duration of one query (not saved permanently)
- ✅ Can be referenced multiple times in the same query (reusable!)
- ✅ Makes complex queries more readable (step-by-step clarity)
- ✅ Can be recursive (self-referencing - advanced!)

**🎓 Beginner Tip:** If "Common Table Expression" sounds scary, just call it "WITH query" - that's what it really is!

---

## Basic CTE Syntax

**The pattern is simple:**
```
WITH [name] AS (
    [your query]
)
SELECT * FROM [name];
```

**Read it like English:**
"WITH this temporary result called [name], do [query], then SELECT from that result"

### 🎯 Your First CTE - Super Simple!

**Problem:** Find employees who earn more than $50,000

**Without CTE (you already know this):**
```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary > 50000;
```

**With CTE (same result, just different structure):**
```sql
-- Step 1: Create a temporary result called "high_earners"
WITH high_earners AS (
    SELECT first_name, last_name, salary
    FROM employees
    WHERE salary > 50000
)
-- Step 2: Use that temporary result
SELECT * FROM high_earners;
```

**🎓 Beginner Tip:** This example doesn't really *need* a CTE - but it shows the syntax! CTEs shine when queries get complex.

### 🎯 A Better Example - Now CTEs Make Sense!

**Problem:** Find employees earning more than the company average

**Without CTE (harder to read):**
```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**With CTE (crystal clear):**
```sql
-- Step 1: Calculate the average (give it a name!)
WITH company_average AS (
    SELECT AVG(salary) AS avg_sal
    FROM employees
)
-- Step 2: Find employees above that average
SELECT e.first_name, e.last_name, e.salary
FROM employees e
CROSS JOIN company_average
WHERE e.salary > company_average.avg_sal;
```

**Why is this better?**
- 📝 Named step: "company_average" tells you what it is
- 🔄 Reusable: Can use company_average multiple times
- 👁️ Readable: Easy to see "first calculate average, then compare"

---

## When to Use CTEs

### ✅ Use CTEs When:

#### 1. **Improving Readability**
Complex queries become easier to understand when broken into logical steps.

```sql
-- Complex subquery (hard to read)
SELECT d.department_name, dept_stats.avg_salary
FROM departments d
JOIN (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    WHERE status = 'Active'
    GROUP BY department_id
) dept_stats ON d.department_id = dept_stats.department_id
WHERE dept_stats.avg_salary > 60000;

-- Same query with CTE (much clearer)
WITH active_employee_stats AS (
    SELECT 
        department_id, 
        AVG(salary) AS avg_salary
    FROM employees
    WHERE status = 'Active'
    GROUP BY department_id
)
SELECT d.department_name, aes.avg_salary
FROM departments d
JOIN active_employee_stats aes ON d.department_id = aes.department_id
WHERE aes.avg_salary > 60000;
```

#### 2. **Reusing the Same Subquery Multiple Times**
When you need to reference the same calculation in multiple places.

```sql
WITH monthly_sales AS (
    SELECT 
        MONTH(order_date) AS month,
        SUM(amount) AS total_sales
    FROM orders
    WHERE YEAR(order_date) = 2023
    GROUP BY MONTH(order_date)
)
SELECT 
    m1.month,
    m1.total_sales,
    m1.total_sales - (SELECT AVG(total_sales) FROM monthly_sales) AS variance_from_avg,
    (m1.total_sales / (SELECT SUM(total_sales) FROM monthly_sales)) * 100 AS percent_of_total
FROM monthly_sales m1
ORDER BY m1.month;
```

**Why CTE here?** The `monthly_sales` calculation is done once and reused multiple times.

#### 3. **Building Complex Reports Step-by-Step**
Breaking down complex logic into manageable steps.

```sql
WITH 
-- Step 1: Get active customers
active_customers AS (
    SELECT customer_id, customer_name
    FROM customers
    WHERE status = 'Active'
),
-- Step 2: Calculate their total orders
customer_orders AS (
    SELECT 
        ac.customer_id,
        ac.customer_name,
        COUNT(o.order_id) AS order_count,
        SUM(o.amount) AS total_spent
    FROM active_customers ac
    LEFT JOIN orders o ON ac.customer_id = o.customer_id
    GROUP BY ac.customer_id, ac.customer_name
),
-- Step 3: Categorize customers
customer_segments AS (
    SELECT 
        customer_id,
        customer_name,
        order_count,
        total_spent,
        CASE 
            WHEN total_spent > 10000 THEN 'VIP'
            WHEN total_spent > 5000 THEN 'Regular'
            WHEN total_spent > 0 THEN 'Occasional'
            ELSE 'No Orders'
        END AS segment
    FROM customer_orders
)
-- Final result
SELECT * FROM customer_segments
ORDER BY total_spent DESC;
```

#### 4. **Hierarchical Data (Recursive CTEs)**
For parent-child relationships, organizational charts, bill of materials, etc.

```sql
-- Employee hierarchy: Find all reports under a manager
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: Start with the top manager
    SELECT employee_id, first_name, last_name, manager_id, 1 AS level
    FROM employees
    WHERE employee_id = 100  -- CEO
    
    UNION ALL
    
    -- Recursive: Get their direct reports
    SELECT e.employee_id, e.first_name, e.last_name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy
ORDER BY level, last_name;
```

#### 5. **Data Transformation Pipeline**
When you need to transform data in multiple stages.

```sql
WITH 
raw_data AS (
    SELECT order_id, customer_id, amount, order_date
    FROM orders
    WHERE order_date >= '2023-01-01'
),
cleaned_data AS (
    SELECT 
        order_id,
        customer_id,
        amount,
        order_date,
        CASE WHEN amount < 0 THEN 0 ELSE amount END AS cleaned_amount
    FROM raw_data
),
aggregated_data AS (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(cleaned_amount) AS total_amount,
        AVG(cleaned_amount) AS avg_amount
    FROM cleaned_data
    GROUP BY customer_id
)
SELECT * FROM aggregated_data
WHERE total_amount > 1000;
```

---

## Simple CTE Examples

### Example 1: Department Summary
```sql
WITH dept_summary AS (
    SELECT 
        department,
        COUNT(*) AS emp_count,
        AVG(salary) AS avg_salary,
        MAX(salary) AS max_salary
    FROM employees
    GROUP BY department
)
SELECT * FROM dept_summary
WHERE avg_salary > 60000
ORDER BY avg_salary DESC;
```

### Example 2: Year-over-Year Comparison
```sql
WITH sales_2022 AS (
    SELECT SUM(amount) AS total
    FROM orders
    WHERE YEAR(order_date) = 2022
),
sales_2023 AS (
    SELECT SUM(amount) AS total
    FROM orders
    WHERE YEAR(order_date) = 2023
)
SELECT 
    s22.total AS sales_2022,
    s23.total AS sales_2023,
    s23.total - s22.total AS growth,
    ((s23.total - s22.total) / s22.total) * 100 AS growth_percent
FROM sales_2022 s22, sales_2023 s23;
```

### Example 3: Filtering Based on Aggregated Data
```sql
-- Find products that sold more than average in the last month
WITH product_sales AS (
    SELECT 
        product_id,
        SUM(quantity) AS total_quantity
    FROM order_items
    WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
    GROUP BY product_id
),
avg_sales AS (
    SELECT AVG(total_quantity) AS avg_qty
    FROM product_sales
)
SELECT ps.product_id, ps.total_quantity
FROM product_sales ps
CROSS JOIN avg_sales
WHERE ps.total_quantity > avg_sales.avg_qty;
```

---

## Multiple CTEs

You can define multiple CTEs in a single query by separating them with commas.

```sql
WITH 
cte1 AS (
    SELECT ...
),
cte2 AS (
    SELECT ...
),
cte3 AS (
    SELECT ...
    FROM cte1  -- Can reference previous CTEs!
    JOIN cte2 ON ...
)
SELECT * FROM cte3;
```

### Real Example: Customer Analysis
```sql
WITH 
-- CTE 1: Active customers
active_customers AS (
    SELECT customer_id, customer_name, registration_date
    FROM customers
    WHERE status = 'Active'
),
-- CTE 2: Customer orders (uses CTE 1)
customer_order_summary AS (
    SELECT 
        ac.customer_id,
        ac.customer_name,
        COUNT(o.order_id) AS order_count,
        COALESCE(SUM(o.amount), 0) AS total_spent
    FROM active_customers ac
    LEFT JOIN orders o ON ac.customer_id = o.customer_id
    GROUP BY ac.customer_id, ac.customer_name
),
-- CTE 3: Customer lifetime (uses CTE 1)
customer_lifetime AS (
    SELECT 
        customer_id,
        DATEDIFF(CURDATE(), registration_date) AS days_as_customer
    FROM active_customers
),
-- CTE 4: Final calculation (uses CTE 2 and 3)
customer_metrics AS (
    SELECT 
        cos.customer_id,
        cos.customer_name,
        cos.order_count,
        cos.total_spent,
        cl.days_as_customer,
        CASE 
            WHEN cl.days_as_customer > 0 
            THEN cos.total_spent / cl.days_as_customer 
            ELSE 0 
        END AS avg_daily_value
    FROM customer_order_summary cos
    JOIN customer_lifetime cl ON cos.customer_id = cl.customer_id
)
-- Final query
SELECT * FROM customer_metrics
WHERE order_count > 5
ORDER BY avg_daily_value DESC;
```

**Key Points:**
- Later CTEs can reference earlier CTEs
- Makes complex logic readable and maintainable
- Each CTE is like a building block

---

## Recursive CTEs

Recursive CTEs reference themselves and are perfect for hierarchical or graph data.

### Syntax
```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member (base case)
    SELECT ...
    
    UNION ALL
    
    -- Recursive member (references cte_name)
    SELECT ...
    FROM cte_name
    WHERE termination_condition
)
SELECT * FROM cte_name;
```

### Example 1: Number Series
```sql
-- Generate numbers 1 to 10
WITH RECURSIVE numbers AS (
    SELECT 1 AS n  -- Anchor: start at 1
    
    UNION ALL
    
    SELECT n + 1   -- Recursive: add 1
    FROM numbers
    WHERE n < 10   -- Termination: stop at 10
)
SELECT * FROM numbers;
```

**Output:**
```
n
--
1
2
3
...
10
```

### Example 2: Employee Hierarchy
```sql
-- Find all employees reporting to manager ID 100 (directly or indirectly)
WITH RECURSIVE employee_tree AS (
    -- Anchor: Direct reports of manager 100
    SELECT 
        employee_id, 
        first_name, 
        last_name, 
        manager_id,
        1 AS level,
        CAST(first_name AS CHAR(1000)) AS path
    FROM employees
    WHERE manager_id = 100
    
    UNION ALL
    
    -- Recursive: Their reports
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        et.level + 1,
        CONCAT(et.path, ' > ', e.first_name)
    FROM employees e
    JOIN employee_tree et ON e.manager_id = et.employee_id
    WHERE et.level < 10  -- Prevent infinite loops
)
SELECT 
    level,
    CONCAT(REPEAT('  ', level - 1), first_name, ' ', last_name) AS employee,
    path
FROM employee_tree
ORDER BY level, last_name;
```

**Output:**
```
level | employee          | path
------|-------------------|------------------
1     | John Smith        | John
1     | Jane Doe          | Jane
2     |   Bob Johnson     | Jane > Bob
2     |   Alice Brown     | John > Alice
3     |     Charlie Davis | John > Alice > Charlie
```

### Example 3: Category Hierarchy
```sql
-- Product categories with parent-child relationships
WITH RECURSIVE category_tree AS (
    -- Anchor: Top-level categories
    SELECT 
        category_id,
        category_name,
        parent_category_id,
        1 AS level,
        category_name AS full_path
    FROM categories
    WHERE parent_category_id IS NULL
    
    UNION ALL
    
    -- Recursive: Subcategories
    SELECT 
        c.category_id,
        c.category_name,
        c.parent_category_id,
        ct.level + 1,
        CONCAT(ct.full_path, ' > ', c.category_name)
    FROM categories c
    JOIN category_tree ct ON c.parent_category_id = ct.category_id
)
SELECT * FROM category_tree
ORDER BY full_path;
```

### Example 4: Bill of Materials (BOM)
```sql
-- Find all components needed to build a product
WITH RECURSIVE bom AS (
    -- Anchor: Top-level product
    SELECT 
        component_id,
        component_name,
        quantity,
        1 AS level
    FROM product_components
    WHERE product_id = 'BIKE-001'
    
    UNION ALL
    
    -- Recursive: Sub-components
    SELECT 
        pc.component_id,
        pc.component_name,
        bom.quantity * pc.quantity,  -- Multiply quantities up the tree
        bom.level + 1
    FROM product_components pc
    JOIN bom ON pc.product_id = bom.component_id
    WHERE bom.level < 20
)
SELECT 
    level,
    CONCAT(REPEAT('  ', level - 1), component_name) AS component,
    SUM(quantity) AS total_quantity_needed
FROM bom
GROUP BY level, component_id, component_name
ORDER BY level, component_name;
```

### ⚠️ Important: Prevent Infinite Loops
```sql
WITH RECURSIVE bad_example AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM bad_example  -- No termination condition!
)
SELECT * FROM bad_example;  -- This will run forever!
```

**Always include:**
1. A termination condition in the WHERE clause
2. A level counter with a maximum depth check
3. Protection against circular references

---

## CTEs vs Subqueries vs Temp Tables

### CTE (Common Table Expression)
```sql
WITH cte AS (SELECT ...)
SELECT * FROM cte;
```
**Pros:**
- ✅ Easy to read
- ✅ Can be referenced multiple times in same query
- ✅ No cleanup needed
- ✅ Supports recursion

**Cons:**
- ❌ Only exists for one query
- ❌ Can't be indexed
- ❌ May not be optimized as well as temp tables

### Subquery
```sql
SELECT * FROM (SELECT ...) AS subquery;
```
**Pros:**
- ✅ Simple for one-time use
- ✅ Inline definition

**Cons:**
- ❌ Hard to read when nested
- ❌ Can't reuse in same query
- ❌ No recursion support

### Temporary Table
```sql
CREATE TEMPORARY TABLE temp_table AS SELECT ...;
SELECT * FROM temp_table;
DROP TEMPORARY TABLE temp_table;
```
**Pros:**
- ✅ Can be indexed
- ✅ Reusable across multiple queries
- ✅ Better for large datasets
- ✅ Can have statistics

**Cons:**
- ❌ Requires cleanup (DROP)
- ❌ More verbose
- ❌ Overhead of creation

### When to Use What?

| Scenario | Best Choice |
|----------|-------------|
| One-time simple calculation | Subquery |
| Complex query needing readability | CTE |
| Reusing result in same query multiple times | CTE |
| Hierarchical/recursive data | Recursive CTE |
| Large intermediate result set (millions of rows) | Temp Table |
| Need to index intermediate results | Temp Table |
| Multiple queries using same intermediate data | Temp Table |

---

## Performance Considerations

### 1. CTEs are NOT Materialized (by default)
In MySQL, CTEs are typically inlined like subqueries.

```sql
WITH expensive_calc AS (
    SELECT * FROM large_table WHERE complex_condition
)
SELECT * FROM expensive_calc  -- Query 1
UNION ALL
SELECT * FROM expensive_calc; -- Query 2 - expensive_calc runs AGAIN!
```

**Solution:** Use a temporary table for expensive calculations used multiple times.

```sql
CREATE TEMPORARY TABLE expensive_calc AS
    SELECT * FROM large_table WHERE complex_condition;

SELECT * FROM expensive_calc  -- Uses cached result
UNION ALL
SELECT * FROM expensive_calc; -- Uses same cached result

DROP TEMPORARY TABLE expensive_calc;
```

### 2. CTE Optimization
```sql
-- ❌ DON'T: Filter after CTE if you can filter inside
WITH all_orders AS (
    SELECT * FROM orders  -- Gets ALL orders
)
SELECT * FROM all_orders
WHERE order_date >= '2023-01-01';  -- Then filters

-- ✅ DO: Filter inside CTE
WITH recent_orders AS (
    SELECT * FROM orders
    WHERE order_date >= '2023-01-01'  -- Filter early
)
SELECT * FROM recent_orders;
```

### 3. Recursive CTE Depth
```sql
-- Set a reasonable recursion limit
SET SESSION cte_max_recursion_depth = 1000;  -- Default is often 1000

WITH RECURSIVE deep_hierarchy AS (
    SELECT id, parent_id, 1 AS level FROM table1 WHERE parent_id IS NULL
    UNION ALL
    SELECT t.id, t.parent_id, dh.level + 1
    FROM table1 t
    JOIN deep_hierarchy dh ON t.parent_id = dh.id
    WHERE dh.level < 100  -- Safety limit
)
SELECT * FROM deep_hierarchy;
```

---

## Common Mistakes

### Mistake 1: Forgetting AS Keyword
```sql
-- ❌ WRONG
WITH cte (SELECT ...)  -- Missing AS

-- ✅ CORRECT
WITH cte AS (SELECT ...)
```

### Mistake 2: Not Naming CTE Columns When Needed
```sql
-- ❌ Can cause issues with complex expressions
WITH cte AS (
    SELECT COUNT(*), AVG(salary) FROM employees
)
SELECT * FROM cte;  -- Column names are COUNT(*) and AVG(salary)

-- ✅ Better
WITH cte AS (
    SELECT 
        COUNT(*) AS emp_count, 
        AVG(salary) AS avg_sal 
    FROM employees
)
SELECT * FROM cte;
```

### Mistake 3: Circular Reference in Multiple CTEs
```sql
-- ❌ WRONG: cte1 references cte2, cte2 references cte1
WITH 
cte1 AS (SELECT * FROM cte2),  -- Error!
cte2 AS (SELECT * FROM cte1)
SELECT * FROM cte1;

-- ✅ CORRECT: Linear dependency
WITH 
cte1 AS (SELECT * FROM base_table),
cte2 AS (SELECT * FROM cte1)  -- OK
SELECT * FROM cte2;
```

### Mistake 4: No Termination in Recursive CTE
```sql
-- ❌ WRONG: Infinite recursion
WITH RECURSIVE bad_cte AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM bad_cte  -- Never stops!
)
SELECT * FROM bad_cte;

-- ✅ CORRECT: Termination condition
WITH RECURSIVE good_cte AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM good_cte
    WHERE n < 100  -- Stops at 100
)
SELECT * FROM good_cte;
```

### Mistake 5: Using CTE for Large Data Without Indexes
```sql
-- ❌ WRONG: CTE can't be indexed
WITH huge_cte AS (
    SELECT * FROM orders  -- 10 million rows
)
SELECT * FROM huge_cte WHERE customer_id = 12345;  -- Slow!

-- ✅ BETTER: Use temp table with index
CREATE TEMPORARY TABLE huge_temp AS SELECT * FROM orders;
CREATE INDEX idx_customer ON huge_temp(customer_id);
SELECT * FROM huge_temp WHERE customer_id = 12345;  -- Fast!
DROP TEMPORARY TABLE huge_temp;
```

---

## Real-World Use Cases

### Use Case 1: Running Totals
```sql
WITH daily_sales AS (
    SELECT 
        order_date,
        SUM(amount) AS daily_total
    FROM orders
    WHERE YEAR(order_date) = 2023
    GROUP BY order_date
)
SELECT 
    order_date,
    daily_total,
    SUM(daily_total) OVER (ORDER BY order_date) AS running_total
FROM daily_sales
ORDER BY order_date;
```

Related: [[Window_Functions_Guide|Window Functions]] for running totals.

### Use Case 2: Data Quality Check
```sql
WITH 
valid_emails AS (
    SELECT COUNT(*) AS count FROM customers WHERE email LIKE '%@%.%'
),
invalid_emails AS (
    SELECT COUNT(*) AS count FROM customers WHERE email NOT LIKE '%@%.%'
),
total_customers AS (
    SELECT COUNT(*) AS count FROM customers
)
SELECT 
    tc.count AS total,
    ve.count AS valid,
    ie.count AS invalid,
    (ve.count / tc.count) * 100 AS valid_percent
FROM total_customers tc, valid_emails ve, invalid_emails ie;
```

### Use Case 3: Gap Analysis
```sql
-- Find missing order IDs
WITH RECURSIVE all_ids AS (
    SELECT MIN(order_id) AS id FROM orders
    UNION ALL
    SELECT id + 1 FROM all_ids
    WHERE id < (SELECT MAX(order_id) FROM orders)
)
SELECT ai.id AS missing_order_id
FROM all_ids ai
LEFT JOIN orders o ON ai.id = o.order_id
WHERE o.order_id IS NULL;
```

---

## Summary

### Key Takeaways:
1. **CTEs improve readability** - Break complex queries into logical steps
2. **Use WITH keyword** - Defines temporary named result sets
3. **Can reference multiple times** - Unlike subqueries
4. **Supports recursion** - Perfect for hierarchical data
5. **Not materialized** - Consider temp tables for expensive repeated operations
6. **Always terminate recursive CTEs** - Prevent infinite loops

### Quick Decision Guide:
- **Need readability?** → Use CTE
- **Hierarchical data?** → Use Recursive CTE
- **Large data with repeated access?** → Use Temp Table
- **Simple one-time calculation?** → Use Subquery

---

## Next Steps

- Apply CTEs to complex [[SELECT_Complete_Guide|SELECT queries]]
- Combine CTEs with [[JOINS_Guide|JOINs]] for powerful data analysis
- Use CTEs with [[Window_Functions_Guide|Window Functions]] for advanced analytics
- Review [[SQL_Best_Practices|Best Practices]] for production queries

[[SQL_Master_Guide|← Back to Master Guide]]

