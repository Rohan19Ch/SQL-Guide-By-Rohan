# Window Functions & PARTITION BY - Complete Guide
**Created by: Rohan Choudhary**

[[SQL_Master_Guide|← Back to Master Guide]]

---

## 🪟 Window Functions: See the Big Picture!

**Don't panic!** Window functions sound complicated but they're actually super useful once you "get it".

### 🎯 The Classroom Analogy

Imagine you're in a classroom with 30 students. The teacher asks:
- "What's your test score?" → You look at YOUR paper (normal SQL)
- "What's the average score?" → You look at EVERYONE's papers (GROUP BY)
- "What's your score AND how does it compare to the class average?" → **WINDOW FUNCTION!**

**Window functions let you see BOTH:**
- 👤 Your individual row (like "your test score")
- 📊 Information about related rows (like "class average")

**All in ONE query, without losing any rows!**

### 💡 Key Difference: Window vs GROUP BY

**GROUP BY Example:**
```
Before:              After GROUP BY:
Student | Score      Class     | Avg Score
--------|------      ----------|-----------
Alice   | 95         Room 101  | 87
Bob     | 80         Room 102  | 92
Charlie | 75         
David   | 90         
(4 rows)             (2 rows) ← Collapsed!
```

**Window Function Example:**
```
Before:              After Window:
Student | Score      Student | Score | Class Avg
--------|------      --------|-------|----------
Alice   | 95         Alice   | 95    | 85
Bob     | 80         Bob     | 80    | 85
Charlie | 75         Charlie | 75    | 85
David   | 90         David   | 90    | 85
(4 rows)             (4 rows) ← Same! Just added info!
```

**🎓 Remember:** 
- GROUP BY → Collapses rows (fewer rows)
- Window Functions → Keeps all rows (same rows, more columns)

---

## Table of Contents
1. [[#What are Window Functions?]]
2. [[#Understanding PARTITION BY]]
3. [[#Understanding ORDER BY in Window Functions]]
4. [[#RANK Functions Explained]]
5. [[#ROW_NUMBER]]
6. [[#RANK]]
7. [[#DENSE_RANK]]
8. [[#Comparison: ROW_NUMBER vs RANK vs DENSE_RANK]]
9. [[#Other Window Functions]]
10. [[#ROWS and RANGE Frames]]
11. [[#Real-World Examples]]
12. [[#Common Mistakes]]

---

## What are Window Functions?

Imagine you're looking through a window at your data. You can see each row, but you can also see related rows around it and perform calculations across them - **without collapsing the rows like GROUP BY does**.

### Key Difference from GROUP BY

**GROUP BY:**
```sql
-- Collapses rows into groups
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```
**Result:** One row per department
```
department  | avg_salary
------------|------------
Sales       | 55000
Engineering | 75000
HR          | 50000
```

**Window Function:**
```sql
-- Keeps all rows, adds calculated column
SELECT 
    employee_id,
    first_name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees;
```
**Result:** All rows preserved, with department average added
```
employee_id | first_name | department  | salary | dept_avg_salary
------------|------------|-------------|--------|----------------
1           | John       | Sales       | 50000  | 55000
2           | Jane       | Sales       | 60000  | 55000
3           | Bob        | Engineering | 80000  | 75000
4           | Alice      | Engineering | 70000  | 75000
5           | Charlie    | HR          | 50000  | 50000
```

### Window Function Syntax
```sql
function_name(column) OVER (
    [PARTITION BY partition_column]
    [ORDER BY order_column]
    [ROWS/RANGE frame_specification]
)
```

**Components:**
- **Function**: What to calculate (RANK, SUM, AVG, etc.)
- **OVER**: Keyword that makes it a window function
- **PARTITION BY**: Divides data into groups (like GROUP BY, but doesn't collapse)
- **ORDER BY**: Defines ordering within each partition
- **ROWS/RANGE**: Defines which rows to include in calculation (optional)

---

## Understanding PARTITION BY

**PARTITION BY = Create separate groups, calculate within each group**

### 🎯 The School Class Analogy

Imagine a school with multiple classes:
- **Without PARTITION BY:** Calculate average for the ENTIRE school
- **With PARTITION BY class:** Calculate average for EACH class separately

**Real example:**
```
Students without partition:
Name    | Class  | Score | School Avg
--------|--------|-------|------------
Alice   | 10A    | 95    | 80  ← Everyone sees same number
Bob     | 10A    | 85    | 80
Charlie | 10B    | 70    | 80
David   | 10B    | 75    | 80

Students WITH PARTITION BY class:
Name    | Class  | Score | Class Avg
--------|--------|-------|------------
Alice   | 10A    | 95    | 90  ← 10A average
Bob     | 10A    | 85    | 90  ← 10A average
Charlie | 10B    | 70    | 72.5  ← 10B average
David   | 10B    | 75    | 72.5  ← 10B average
```

### 💡 Think of PARTITION BY as:
- 🏠 Separate apartments in a building (each apartment has its own calculation)
- 📚 Separate chapters in a book (analyze each chapter independently)
- 🏪 Separate departments in a store (sales per department)
- 🎬 Separate seasons of a TV show (rank episodes within each season)

### Example 1: Without PARTITION BY
```sql
-- Average salary across ALL employees
SELECT 
    employee_id,
    first_name,
    salary,
    AVG(salary) OVER () AS company_avg
FROM employees;
```

**Result:**
```
employee_id | first_name | salary | company_avg
------------|------------|--------|-------------
1           | John       | 50000  | 60000
2           | Jane       | 60000  | 60000
3           | Bob        | 80000  | 60000
4           | Alice      | 70000  | 60000
5           | Charlie    | 50000  | 60000
```
Every row has the same company_avg (60000).

### Example 2: With PARTITION BY
```sql
-- Average salary per department
SELECT 
    employee_id,
    first_name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

**Result:**
```
employee_id | first_name | department  | salary | dept_avg
------------|------------|-------------|--------|----------
1           | John       | Sales       | 50000  | 55000
2           | Jane       | Sales       | 60000  | 55000
3           | Bob        | Engineering | 80000  | 75000
4           | Alice      | Engineering | 70000  | 75000
5           | Charlie    | HR          | 50000  | 50000
```

Sales employees see Sales average (55000), Engineering sees Engineering average (75000), etc.

### Example 3: Multiple Partitions
```sql
-- Average salary by department AND job title
SELECT 
    employee_id,
    first_name,
    department,
    job_title,
    salary,
    AVG(salary) OVER (PARTITION BY department, job_title) AS group_avg
FROM employees;
```

Creates partitions for each unique combination of department + job_title.

### Visual Representation
```
Without PARTITION BY:
┌─────────────────────────────────────────┐
│ All rows in one window                   │
│ ├─ Row 1                                │
│ ├─ Row 2                                │
│ ├─ Row 3                                │
│ └─ Row 4                                │
│ Calculation: Across all rows            │
└─────────────────────────────────────────┘

With PARTITION BY department:
┌─────────────────────┐  ┌─────────────────────┐  ┌──────────────────┐
│ Sales partition     │  │ Engineering part.   │  │ HR partition     │
│ ├─ Row 1           │  │ ├─ Row 3           │  │ ├─ Row 5         │
│ └─ Row 2           │  │ └─ Row 4           │  │ Calc: HR only   │
│ Calc: Sales only   │  │ Calc: Eng only     │  └──────────────────┘
└─────────────────────┘  └─────────────────────┘
```

---

## Understanding ORDER BY in Window Functions

**ORDER BY** within a window function determines:
1. The order of rows within each partition
2. Which rows to include in calculations (for cumulative operations)

### Example 1: ORDER BY Creates Cumulative Behavior
```sql
-- Running total of sales
SELECT 
    order_date,
    order_amount,
    SUM(order_amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

**Result:**
```
order_date  | order_amount | running_total
------------|--------------|---------------
2023-01-01  | 100          | 100        (100)
2023-01-02  | 150          | 250        (100+150)
2023-01-03  | 200          | 450        (100+150+200)
2023-01-04  | 50           | 500        (100+150+200+50)
```

**Without ORDER BY** (total of all rows):
```sql
SELECT 
    order_date,
    order_amount,
    SUM(order_amount) OVER () AS total
FROM orders;
```

**Result:**
```
order_date  | order_amount | total
------------|--------------|------
2023-01-01  | 100          | 500
2023-01-02  | 150          | 500
2023-01-03  | 200          | 500
2023-01-04  | 50           | 500
```

### Example 2: ORDER BY with PARTITION BY
```sql
-- Running total per customer
SELECT 
    customer_id,
    order_date,
    order_amount,
    SUM(order_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS customer_running_total
FROM orders
ORDER BY customer_id, order_date;
```

**Result:**
```
customer_id | order_date  | order_amount | customer_running_total
------------|-------------|--------------|------------------------
101         | 2023-01-01  | 100          | 100
101         | 2023-01-05  | 200          | 300
101         | 2023-01-10  | 150          | 450
102         | 2023-01-02  | 50           | 50     ← Resets for new customer
102         | 2023-01-08  | 75           | 125
```

The running total resets for each customer!

---

## RANK Functions Explained

**The Three Ranking Functions:** ROW_NUMBER, RANK, and DENSE_RANK

### 🎯 The Olympic Medals Analogy

Imagine an Olympic race with these finish times:

```
Runner    | Time (seconds)
----------|---------------
Alice     | 10.5  ← 1st place (fastest)
Bob       | 10.8  ← Tied for 2nd
Charlie   | 10.8  ← Tied for 2nd
David     | 11.0  ← What place is this?
Eve       | 11.0  ← What place is this?
Frank     | 11.2  ← What place is this?
```

**Question:** If Bob and Charlie tie for 2nd, what place is David?
- **Option A:** Call him "4th" (skip 3rd because two people tied for 2nd) ← RANK
- **Option B:** Call him "3rd" (next sequential number) ← DENSE_RANK
- **Option C:** Give everyone unique numbers regardless of ties ← ROW_NUMBER

### Sample Data: Test Scores
```
student_id | student_name | score
-----------|--------------|-------
1          | Alice        | 95   ← Highest
2          | Bob          | 90   ← Tied for 2nd
3          | Charlie      | 90   ← Tied for 2nd
4          | David        | 85   ← Tied for 4th? or 3rd?
5          | Eve          | 85   ← Tied for 4th? or 3rd?
6          | Frank        | 80   ← 6th? 5th? or 4th?
```

**🤔 Question:** How do we rank them when there are ties?

Let's see the three different approaches...

---

## ROW_NUMBER

**ROW_NUMBER = Everyone gets a unique number (1, 2, 3, 4...) even if they tie**

### 🎯 The Race Bib Analogy

Think of marathon runners with race bib numbers:
- Everyone gets a UNIQUE bib number (no duplicates!)
- Even if two runners finish at the exact same time, they have different bibs
- The numbers are just 1, 2, 3, 4, 5... in order

**Key:** ROW_NUMBER doesn't care about ties - it just counts rows!

### Syntax
```sql
ROW_NUMBER() OVER (
    [PARTITION BY column]
    ORDER BY column
)
```

**Read it as:** "Number the rows, ordered by [column]"

### Example 1: Basic ROW_NUMBER
```sql
SELECT 
    student_name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num
FROM students;
```

**Result:**
```
student_name | score | row_num
-------------|-------|--------
Alice        | 95    | 1
Bob          | 90    | 2    ← Tied with Charlie, but gets 2
Charlie      | 90    | 3    ← Tied with Bob, but gets 3
David        | 85    | 4
Eve          | 85    | 5
Frank        | 80    | 6
```

**Key Point:** Even with ties (90, 90 and 85, 85), ROW_NUMBER assigns unique numbers. The order among ties depends on the underlying row order (unpredictable without additional ORDER BY).

### When to Use ROW_NUMBER?

✅ **Use ROW_NUMBER when:**
- You need unique identifiers for each row
- You want to select the "top N" of each group
- You're implementing pagination
- Ties should be broken arbitrarily

### Example 2: ROW_NUMBER with PARTITION BY
```sql
-- Number students within each class
SELECT 
    class_id,
    student_name,
    score,
    ROW_NUMBER() OVER (PARTITION BY class_id ORDER BY score DESC) AS rank_in_class
FROM students;
```

**Result:**
```
class_id | student_name | score | rank_in_class
---------|--------------|-------|---------------
A        | Alice        | 95    | 1
A        | Bob          | 90    | 2
A        | Charlie      | 85    | 3
B        | David        | 92    | 1    ← Resets for new partition
B        | Eve          | 88    | 2
B        | Frank        | 80    | 3
```

### Example 3: Getting Top N Per Group
```sql
-- Top 3 students per class
WITH ranked_students AS (
    SELECT 
        class_id,
        student_name,
        score,
        ROW_NUMBER() OVER (PARTITION BY class_id ORDER BY score DESC) AS rn
    FROM students
)
SELECT class_id, student_name, score
FROM ranked_students
WHERE rn <= 3;
```

Related: [[CTE_Guide|Common Table Expressions]]

### Example 4: Removing Duplicates
```sql
-- Keep only the first occurrence of each email
WITH ranked_users AS (
    SELECT 
        user_id,
        email,
        created_date,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_date) AS rn
    FROM users
)
SELECT user_id, email, created_date
FROM ranked_users
WHERE rn = 1;
```

---

## RANK

**RANK()** assigns the same rank to tied values, and **skips ranks** after ties.

### Syntax
```sql
RANK() OVER (
    [PARTITION BY column]
    ORDER BY column
)
```

### Example 1: Basic RANK
```sql
SELECT 
    student_name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank
FROM students;
```

**Result:**
```
student_name | score | rank
-------------|-------|------
Alice        | 95    | 1
Bob          | 90    | 2    ← Tied with Charlie
Charlie      | 90    | 2    ← Tied with Bob, gets same rank
David        | 85    | 4    ← Rank jumps to 4 (skips 3)
Eve          | 85    | 4    ← Tied with David
Frank        | 80    | 6    ← Rank jumps to 6 (skips 5)
```

**Key Point:** 
- Bob and Charlie both get rank 2
- Next rank is 4 (not 3) because two people tied for 2nd place
- David and Eve both get rank 4
- Next rank is 6 (not 5)

**Formula:** If N people tie for rank R, the next rank is R + N.

### When to Use RANK?

✅ **Use RANK when:**
- Ties should have the same rank
- You want to preserve the "gap" after ties (traditional ranking)
- Think: Olympic medals - if two people tie for silver, the next is bronze (3rd place)

### Example 2: Sales Performance Ranking
```sql
SELECT 
    salesperson_name,
    total_sales,
    RANK() OVER (ORDER BY total_sales DESC) AS performance_rank
FROM sales_summary;
```

**Result:**
```
salesperson_name | total_sales | performance_rank
-----------------|-------------|------------------
John             | 100000      | 1
Jane             | 95000       | 2
Bob              | 95000       | 2
Alice            | 90000       | 4
Charlie          | 85000       | 5
```

John gets 1st place, Jane and Bob share 2nd, Alice is 4th (not 3rd).

### Example 3: RANK with PARTITION BY
```sql
-- Rank products by sales within each category
SELECT 
    category,
    product_name,
    units_sold,
    RANK() OVER (PARTITION BY category ORDER BY units_sold DESC) AS category_rank
FROM products;
```

**Result:**
```
category    | product_name | units_sold | category_rank
------------|--------------|------------|---------------
Electronics | Laptop       | 500        | 1
Electronics | Phone        | 450        | 2
Electronics | Tablet       | 450        | 2
Electronics | Headphones   | 300        | 4
Books       | Novel A      | 1000       | 1    ← Resets for new category
Books       | Novel B      | 800        | 2
```

---

## DENSE_RANK

**DENSE_RANK()** assigns the same rank to tied values, but **does NOT skip ranks** after ties.

### Syntax
```sql
DENSE_RANK() OVER (
    [PARTITION BY column]
    ORDER BY column
)
```

### Example 1: Basic DENSE_RANK
```sql
SELECT 
    student_name,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students;
```

**Result:**
```
student_name | score | dense_rank
-------------|-------|------------
Alice        | 95    | 1
Bob          | 90    | 2
Charlie      | 90    | 2
David        | 85    | 3    ← No gap! Next rank is 3
Eve          | 85    | 3
Frank        | 80    | 4    ← No gap! Next rank is 4
```

**Key Point:**
- Bob and Charlie both get rank 2
- Next rank is 3 (NOT 4) - no gaps
- David and Eve both get rank 3
- Next rank is 4 (NOT 6) - no gaps

### When to Use DENSE_RANK?

✅ **Use DENSE_RANK when:**
- Ties should have the same rank
- You want **consecutive rankings** (no gaps)
- Think: Top 10 list - you want exactly 10 distinct ranks (or fewer with ties)

### Example 2: Top Products by Revenue
```sql
-- Get products ranked in top 5 by revenue (no gaps in ranking)
WITH product_ranks AS (
    SELECT 
        product_name,
        revenue,
        DENSE_RANK() OVER (ORDER BY revenue DESC) AS revenue_rank
    FROM products
)
SELECT product_name, revenue, revenue_rank
FROM product_ranks
WHERE revenue_rank <= 5;
```

If there are ties, you might get more than 5 products, but they'll have ranks 1-5 only.

### Example 3: Grade Assignment
```sql
-- Assign letter grades based on score ranges
SELECT 
    student_name,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS rank,
    CASE 
        WHEN DENSE_RANK() OVER (ORDER BY score DESC) <= 2 THEN 'A'
        WHEN DENSE_RANK() OVER (ORDER BY score DESC) <= 4 THEN 'B'
        ELSE 'C'
    END AS grade
FROM students;
```

DENSE_RANK ensures that if multiple students have the same score, they get the same rank and grade.

---

## Comparison: ROW_NUMBER vs RANK vs DENSE_RANK

### 🏅 Side-by-Side: The Complete Picture

Let's see all three at once with our test scores:

```sql
SELECT 
    student_name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
    RANK() OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students;
```

**Result:**
```
student_name | score | row_num | rank | dense_rank | Explanation
-------------|-------|---------|------|------------|-------------
Alice        | 95    | 1       | 1    | 1          | Highest score
Bob          | 90    | 2       | 2    | 2          | Tied for 2nd
Charlie      | 90    | 3       | 2    | 2          | Tied for 2nd (same rank!)
David        | 85    | 4       | 4    | 3          | row_num & rank skip 3
Eve          | 85    | 5       | 4    | 3          | dense_rank is 3 (no skip)
Frank        | 80    | 6       | 6    | 4          | All different numbers!
George       | 80    | 7       | 6    | 4          | 
Helen        | 75    | 8       | 8    | 5          |
```

### 🎯 The Easy Way to Remember

**Imagine announcing race results:**

**ROW_NUMBER:** "Finisher #1, finisher #2, finisher #3..."
- Just counting finishers (1,2,3,4,5,6,7,8)
- Doesn't care about ties

**RANK:** "1st place, TWO people tied for 2nd, so next is 4th..."
- Traditional sports ranking
- Skips numbers after ties (1,2,2,4,4,6,6,8)

**DENSE_RANK:** "1st place, 2nd place (tied), 3rd place, 4th place..."
- Every distinct score gets the next number
- Never skips numbers (1,2,2,3,3,4,4,5)

### Comparison Table

| Function | Handles Ties? | Skips Ranks? | Use Case |
|----------|---------------|--------------|----------|
| **ROW_NUMBER** | No - assigns unique numbers | N/A | Unique sequential numbering, pagination, deduplication |
| **RANK** | Yes - same rank for ties | Yes - gaps after ties | Traditional ranking (Olympics style), where gaps make sense |
| **DENSE_RANK** | Yes - same rank for ties | No - consecutive ranks | Top N lists, grade assignment, where you want no gaps |

### Decision Tree
```
Do you want ties to have the same number?
├─ NO: Use ROW_NUMBER()
│   └─ Each row gets unique number
│
└─ YES: Are gaps acceptable after ties?
    ├─ YES: Use RANK()
    │   └─ Ties share rank, next rank skips (1,2,2,4,5...)
    │
    └─ NO: Use DENSE_RANK()
        └─ Ties share rank, next rank consecutive (1,2,2,3,4...)
```

---

## Other Window Functions

### 1. Aggregate Window Functions

All aggregate functions can be used as window functions.

#### SUM
```sql
-- Running total
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

#### AVG
```sql
-- Compare each salary to department average
SELECT 
    employee_name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg
FROM employees;
```

#### COUNT
```sql
-- Count employees per department (shown on each row)
SELECT 
    employee_name,
    department,
    COUNT(*) OVER (PARTITION BY department) AS dept_size
FROM employees;
```

#### MIN and MAX
```sql
-- Compare each product price to category min/max
SELECT 
    product_name,
    category,
    price,
    MIN(price) OVER (PARTITION BY category) AS category_min,
    MAX(price) OVER (PARTITION BY category) AS category_max
FROM products;
```

### 2. Value Window Functions

#### LAG - Access Previous Row
```sql
-- Compare each month's sales to previous month
SELECT 
    month,
    sales,
    LAG(sales, 1) OVER (ORDER BY month) AS previous_month_sales,
    sales - LAG(sales, 1) OVER (ORDER BY month) AS month_over_month_change
FROM monthly_sales;
```

**Result:**
```
month   | sales | previous_month_sales | month_over_month_change
--------|-------|----------------------|------------------------
2023-01 | 1000  | NULL                 | NULL
2023-02 | 1200  | 1000                 | 200
2023-03 | 1100  | 1200                 | -100
```

#### LEAD - Access Next Row
```sql
-- Compare current price to next price change
SELECT 
    product_id,
    effective_date,
    price,
    LEAD(price, 1) OVER (ORDER BY effective_date) AS next_price,
    LEAD(effective_date, 1) OVER (ORDER BY effective_date) AS next_change_date
FROM price_history;
```

#### FIRST_VALUE and LAST_VALUE
```sql
-- Compare each order to first and last order of the customer
SELECT 
    customer_id,
    order_id,
    order_date,
    amount,
    FIRST_VALUE(amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS first_order_amount,
    LAST_VALUE(amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_order_amount
FROM orders;
```

**⚠️ Note:** `LAST_VALUE` often requires explicit frame specification to work as expected.

#### NTH_VALUE
```sql
-- Get the 2nd highest salary in each department
SELECT 
    employee_name,
    department,
    salary,
    NTH_VALUE(salary, 2) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_highest_salary
FROM employees;
```

### 3. Statistical Window Functions

#### PERCENT_RANK
```sql
-- Percentile rank (0 to 1)
SELECT 
    student_name,
    score,
    PERCENT_RANK() OVER (ORDER BY score DESC) AS percentile
FROM students;
```

#### CUME_DIST
```sql
-- Cumulative distribution (0 to 1)
SELECT 
    employee_name,
    salary,
    CUME_DIST() OVER (ORDER BY salary) AS cumulative_dist
FROM employees;
```

#### NTILE
```sql
-- Divide into N equal groups (quartiles, deciles, etc.)
SELECT 
    student_name,
    score,
    NTILE(4) OVER (ORDER BY score DESC) AS quartile
FROM students;
```

**Result:**
```
student_name | score | quartile
-------------|-------|----------
Alice        | 95    | 1    ← Top 25%
Bob          | 90    | 1
Charlie      | 85    | 2    ← 25-50%
David        | 80    | 2
Eve          | 75    | 3    ← 50-75%
Frank        | 70    | 3
George       | 65    | 4    ← Bottom 25%
Helen        | 60    | 4
```

---

## ROWS and RANGE Frames

**Frame specification** defines which rows are included in the window calculation relative to the current row.

### Default Frames
```sql
-- With ORDER BY: From start to current row (RANGE UNBOUNDED PRECEDING)
SUM(amount) OVER (ORDER BY date)

-- Without ORDER BY: All rows in partition
SUM(amount) OVER (PARTITION BY category)
```

### ROWS Frame
```sql
-- Current row and 2 preceding rows (3-row moving average)
SELECT 
    date,
    sales,
    AVG(sales) OVER (
        ORDER BY date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3day
FROM daily_sales;
```

**Result:**
```
date        | sales | moving_avg_3day
------------|-------|----------------
2023-01-01  | 100   | 100          (only 1 row available)
2023-01-02  | 150   | 125          (100+150)/2
2023-01-03  | 200   | 150          (100+150+200)/3
2023-01-04  | 180   | 176.67       (150+200+180)/3
2023-01-05  | 220   | 200          (200+180+220)/3
```

### Common Frame Patterns

```sql
-- All rows from start to current (running total)
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- All rows in partition
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

-- Current row only
ROWS BETWEEN CURRENT ROW AND CURRENT ROW

-- 3-row centered window (1 before, current, 1 after)
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING

-- From current to end
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING

-- Last 7 rows including current
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
```

### Example: Moving Average
```sql
-- 7-day moving average of sales
SELECT 
    date,
    sales,
    AVG(sales) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM daily_sales;
```

### Example: Running vs Total
```sql
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    SUM(amount) OVER (
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS grand_total,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) * 100.0 / SUM(amount) OVER (
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS percent_of_total
FROM orders;
```

---

## Real-World Examples

### Example 1: Top 3 Products Per Category
```sql
WITH ranked_products AS (
    SELECT 
        category,
        product_name,
        revenue,
        ROW_NUMBER() OVER (
            PARTITION BY category 
            ORDER BY revenue DESC
        ) AS rank
    FROM products
)
SELECT category, product_name, revenue
FROM ranked_products
WHERE rank <= 3
ORDER BY category, rank;
```

### Example 2: Year-over-Year Comparison
```sql
SELECT 
    product_id,
    year,
    revenue,
    LAG(revenue, 1) OVER (PARTITION BY product_id ORDER BY year) AS prev_year_revenue,
    revenue - LAG(revenue, 1) OVER (PARTITION BY product_id ORDER BY year) AS yoy_change,
    CASE 
        WHEN LAG(revenue, 1) OVER (PARTITION BY product_id ORDER BY year) IS NOT NULL
        THEN ((revenue - LAG(revenue, 1) OVER (PARTITION BY product_id ORDER BY year)) 
              / LAG(revenue, 1) OVER (PARTITION BY product_id ORDER BY year)) * 100
        ELSE NULL
    END AS yoy_growth_percent
FROM annual_revenue
ORDER BY product_id, year;
```

### Example 3: Customer Lifetime Value Tracking
```sql
SELECT 
    customer_id,
    order_date,
    amount,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS order_number,
    SUM(amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS lifetime_value,
    AVG(amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS avg_order_value,
    DATEDIFF(
        order_date,
        FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date)
    ) AS days_since_first_order
FROM orders
ORDER BY customer_id, order_date;
```

### Example 4: Identifying Gaps in Sequences
```sql
-- Find missing order IDs
WITH order_sequence AS (
    SELECT 
        order_id,
        LAG(order_id) OVER (ORDER BY order_id) AS prev_order_id,
        order_id - LAG(order_id) OVER (ORDER BY order_id) AS gap
    FROM orders
)
SELECT 
    prev_order_id AS last_order_before_gap,
    order_id AS first_order_after_gap,
    gap - 1 AS number_of_missing_orders
FROM order_sequence
WHERE gap > 1;
```

### Example 5: Ranking with Percentage
```sql
-- Sales performance with percentile
SELECT 
    salesperson_name,
    total_sales,
    RANK() OVER (ORDER BY total_sales DESC) AS sales_rank,
    COUNT(*) OVER () AS total_salespeople,
    PERCENT_RANK() OVER (ORDER BY total_sales DESC) AS percentile,
    CASE 
        WHEN PERCENT_RANK() OVER (ORDER BY total_sales DESC) < 0.2 THEN 'Top 20%'
        WHEN PERCENT_RANK() OVER (ORDER BY total_sales DESC) < 0.5 THEN 'Above Average'
        WHEN PERCENT_RANK() OVER (ORDER BY total_sales DESC) < 0.8 THEN 'Below Average'
        ELSE 'Bottom 20%'
    END AS performance_tier
FROM sales_summary;
```

### Example 6: Detect Streaks
```sql
-- Find customers with 3+ consecutive months of orders
WITH monthly_orders AS (
    SELECT 
        customer_id,
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id, month
),
streaks AS (
    SELECT 
        customer_id,
        month,
        order_count,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY month) AS row_num,
        DATE_SUB(
            STR_TO_DATE(CONCAT(month, '-01'), '%Y-%m-%d'),
            INTERVAL ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY month) MONTH
        ) AS streak_group
    FROM monthly_orders
)
SELECT 
    customer_id,
    MIN(month) AS streak_start,
    MAX(month) AS streak_end,
    COUNT(*) AS streak_length
FROM streaks
GROUP BY customer_id, streak_group
HAVING COUNT(*) >= 3
ORDER BY customer_id, streak_start;
```

---

## Common Mistakes

### Mistake 1: Forgetting ORDER BY When Needed
```sql
-- ❌ WRONG: RANK without ORDER BY
SELECT 
    student_name,
    score,
    RANK() OVER () AS rank  -- All get rank 1!
FROM students;

-- ✅ CORRECT
SELECT 
    student_name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank
FROM students;
```

### Mistake 2: Using COUNT(*) Instead of COUNT(column) with Aggregates
```sql
-- ❌ WRONG: Counts rows, not non-NULL values
SELECT 
    customer_id,
    COUNT(*) OVER (PARTITION BY customer_id) AS order_count
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id;

-- ✅ CORRECT: Counts actual orders
SELECT 
    customer_id,
    COUNT(order_id) OVER (PARTITION BY customer_id) AS order_count
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id;
```

### Mistake 3: Not Understanding LAST_VALUE Default Frame
```sql
-- ❌ WRONG: LAST_VALUE doesn't work as expected with default frame
SELECT 
    employee_name,
    salary,
    LAST_VALUE(salary) OVER (ORDER BY salary) AS max_salary  -- Wrong!
FROM employees;

-- ✅ CORRECT: Specify full frame
SELECT 
    employee_name,
    salary,
    LAST_VALUE(salary) OVER (
        ORDER BY salary
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS max_salary
FROM employees;
```

### Mistake 4: Confusing Window Functions with GROUP BY
```sql
-- ❌ WRONG: Can't mix aggregation types without proper handling
SELECT 
    department,
    employee_name,  -- Error: not in GROUP BY
    AVG(salary) AS dept_avg
FROM employees
GROUP BY department;

-- ✅ CORRECT: Use window function to keep all rows
SELECT 
    department,
    employee_name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

### Mistake 5: Using Window Functions in WHERE Clause
```sql
-- ❌ WRONG: Can't use window functions in WHERE
SELECT 
    student_name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank
FROM students
WHERE RANK() OVER (ORDER BY score DESC) <= 3;  -- ERROR!

-- ✅ CORRECT: Use subquery or CTE
WITH ranked_students AS (
    SELECT 
        student_name,
        score,
        RANK() OVER (ORDER BY score DESC) AS rank
    FROM students
)
SELECT * FROM ranked_students
WHERE rank <= 3;
```

Related: [[CTE_Guide|CTEs make this pattern clean and readable]]

---

## Performance Tips

### 1. Window Functions vs Subqueries
Window functions are often more efficient than correlated subqueries.

```sql
-- ❌ SLOWER: Correlated subquery
SELECT 
    e1.employee_name,
    e1.salary,
    (SELECT AVG(e2.salary) 
     FROM employees e2 
     WHERE e2.department = e1.department) AS dept_avg
FROM employees e1;

-- ✅ FASTER: Window function
SELECT 
    employee_name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

### 2. Reuse Window Definitions
```sql
-- ❌ VERBOSE: Repeat window definition
SELECT 
    product_name,
    revenue,
    RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rank,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS dense_rank
FROM products;

-- ✅ CLEANER: Use WINDOW clause
SELECT 
    product_name,
    revenue,
    RANK() OVER w AS rank,
    DENSE_RANK() OVER w AS dense_rank
FROM products
WINDOW w AS (PARTITION BY category ORDER BY revenue DESC);
```

### 3. Index Support
Window functions can benefit from indexes on:
- PARTITION BY columns
- ORDER BY columns

```sql
-- Add index to improve this query
CREATE INDEX idx_emp_dept_salary ON employees(department, salary DESC);

SELECT 
    employee_name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
```

---

## Summary

### Quick Reference

| Function | What It Does | Ties | Gaps | Example Output |
|----------|-------------|------|------|----------------|
| **ROW_NUMBER** | Unique sequential number | No same values | N/A | 1,2,3,4,5,6,7,8 |
| **RANK** | Rank with gaps for ties | Same value for ties | Yes | 1,2,2,4,5,6,6,8 |
| **DENSE_RANK** | Rank without gaps | Same value for ties | No | 1,2,2,3,4,5,5,6 |
| **LAG** | Value from previous row | - | - | Compare to last month |
| **LEAD** | Value from next row | - | - | Compare to next month |
| **SUM/AVG/etc** | Aggregate over window | - | - | Running totals, moving averages |

### When to Use What?

- **Need unique row numbers?** → `ROW_NUMBER()`
- **Traditional ranking (with gaps)?** → `RANK()`
- **Top N (no gaps)?** → `DENSE_RANK()`
- **Compare to previous/next?** → `LAG()` / `LEAD()`
- **Running totals?** → `SUM() OVER (ORDER BY ...)`
- **Moving averages?** → `AVG() OVER (... ROWS BETWEEN ...)`
- **Separate calculations per group?** → `PARTITION BY`

### Key Concepts to Remember

1. **OVER()** makes a function a window function
2. **PARTITION BY** divides data into groups (like GROUP BY but keeps all rows)
3. **ORDER BY** determines row order and affects cumulative calculations
4. **ROWS/RANGE** defines which rows to include in calculations
5. Window functions **cannot** be used in WHERE (use CTE or subquery)
6. Window functions are calculated **after** WHERE and GROUP BY

---

## Next Steps

- Apply window functions to complex [[SELECT_Complete_Guide|SELECT queries]]
- Combine with [[CTE_Guide|CTEs]] for readable complex analytics
- Use with [[JOINS_Guide|JOINs]] for multi-table analysis
- Review [[SQL_Best_Practices|Best Practices]] for performance optimization

[[SQL_Master_Guide|← Back to Master Guide]]

