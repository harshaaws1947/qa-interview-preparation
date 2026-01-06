# SQL Interview Questions for QA/SDET

## SQL Basics

### 1. What is SQL and its types?
**SQL (Structured Query Language)** is used to manage relational databases.

**Types of SQL commands:**
| Type | Commands | Purpose |
|------|----------|---------|
| DDL | CREATE, ALTER, DROP, TRUNCATE | Define database structure |
| DML | INSERT, UPDATE, DELETE | Manipulate data |
| DQL | SELECT | Query data |
| DCL | GRANT, REVOKE | Control access |
| TCL | COMMIT, ROLLBACK, SAVEPOINT | Manage transactions |

### 2. What is the difference between DELETE, TRUNCATE, and DROP?

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Type | DML | DDL | DDL |
| WHERE clause | Yes | No | No |
| Rollback | Yes | No (most DBs) | No |
| Speed | Slow | Fast | Fast |
| Table structure | Keeps | Keeps | Removes |
| Triggers | Fires | Doesn't fire | N/A |

### 3. What are ACID properties?
- **Atomicity**: Transaction is all or nothing
- **Consistency**: Database remains in valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data persists

### 4. What is the difference between WHERE and HAVING?

| WHERE | HAVING |
|-------|--------|
| Filters rows before grouping | Filters groups after GROUP BY |
| Cannot use aggregate functions | Can use aggregate functions |
| Used with SELECT, UPDATE, DELETE | Used only with SELECT + GROUP BY |

```sql
-- WHERE: Filter before grouping
SELECT department_id, COUNT(*)
FROM employees
WHERE salary > 5000
GROUP BY department_id;

-- HAVING: Filter after grouping
SELECT department_id, COUNT(*) as emp_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;
```

### 5. What is the order of SQL query execution?
```
1. FROM / JOIN
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. DISTINCT
7. ORDER BY
8. LIMIT / OFFSET
```

## JOINs

### 6. Explain different types of JOINs

```sql
-- INNER JOIN: Only matching rows
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- LEFT JOIN: All from left + matching from right
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- RIGHT JOIN: All from right + matching from left
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- FULL OUTER JOIN: All rows from both tables
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- CROSS JOIN: Cartesian product
SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d;

-- SELF JOIN: Table joined with itself
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

### 7. What is the difference between INNER JOIN and LEFT JOIN?

| INNER JOIN | LEFT JOIN |
|------------|-----------|
| Only matching rows | All rows from left table |
| Excludes non-matching | NULL for non-matching right rows |
| Smaller result set | Larger or equal result set |

### 8. How to find employees without a department?
```sql
-- Using LEFT JOIN
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;

-- Using NOT EXISTS
SELECT e.name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM departments d
    WHERE d.department_id = e.department_id
);

-- Using NOT IN (careful with NULLs)
SELECT name FROM employees
WHERE department_id NOT IN (
    SELECT department_id FROM departments
    WHERE department_id IS NOT NULL
);
```

## Subqueries

### 9. What are different types of subqueries?

```sql
-- Scalar Subquery (returns single value)
SELECT name, salary,
    (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;

-- Row Subquery (returns single row)
SELECT * FROM employees
WHERE (department_id, salary) = (
    SELECT department_id, MAX(salary)
    FROM employees GROUP BY department_id LIMIT 1
);

-- Table Subquery (returns multiple rows)
SELECT * FROM employees
WHERE department_id IN (
    SELECT department_id FROM departments
    WHERE location = 'New York'
);

-- Correlated Subquery (references outer query)
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees
    WHERE department_id = e.department_id
);
```

### 10. What is the difference between IN and EXISTS?

| IN | EXISTS |
|----|--------|
| Compares values | Checks existence |
| Better for small subquery results | Better for large tables |
| Can have issues with NULL | Handles NULL properly |

```sql
-- IN
SELECT * FROM employees
WHERE department_id IN (SELECT department_id FROM departments);

-- EXISTS
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d
    WHERE d.department_id = e.department_id
);
```

## Aggregate Functions

### 11. What are aggregate functions?
```sql
SELECT
    COUNT(*) as total_employees,           -- Count rows
    COUNT(DISTINCT department_id) as depts,-- Count unique
    SUM(salary) as total_salary,           -- Sum values
    AVG(salary) as avg_salary,             -- Average
    MIN(salary) as min_salary,             -- Minimum
    MAX(salary) as max_salary              -- Maximum
FROM employees;
```

### 12. How to handle NULL in aggregates?
```sql
-- COUNT(*) counts all rows, COUNT(column) ignores NULLs
SELECT COUNT(*), COUNT(commission) FROM employees;

-- COALESCE to replace NULL
SELECT AVG(COALESCE(commission, 0)) FROM employees;

-- NULLIF to avoid division by zero
SELECT total / NULLIF(count, 0) FROM sales;
```

## Window Functions

### 13. What are window functions?
Window functions perform calculations across a set of rows related to current row.

```sql
-- ROW_NUMBER: Unique sequential number
SELECT name, salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as row_num
FROM employees;

-- RANK: Same rank for ties, gaps after ties
SELECT name, salary,
    RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- DENSE_RANK: Same rank for ties, no gaps
SELECT name, salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank
FROM employees;

-- PARTITION BY: Window within groups
SELECT name, department_id, salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) as dept_rank
FROM employees;
```

### 14. Explain LAG and LEAD functions
```sql
-- LAG: Access previous row
SELECT name, salary,
    LAG(salary, 1) OVER (ORDER BY hire_date) as prev_salary,
    salary - LAG(salary, 1) OVER (ORDER BY hire_date) as salary_diff
FROM employees;

-- LEAD: Access next row
SELECT name, hire_date,
    LEAD(hire_date) OVER (ORDER BY hire_date) as next_hire_date
FROM employees;
```

### 15. How to calculate running total?
```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) as running_total,
    SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date) as customer_running_total
FROM orders;
```

## Common Interview Queries

### 16. Find Nth highest salary
```sql
-- Using LIMIT OFFSET (MySQL)
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;  -- 2nd highest

-- Using ROW_NUMBER
SELECT salary FROM (
    SELECT salary, ROW_NUMBER() OVER (ORDER BY salary DESC) as rn
    FROM employees
) ranked
WHERE rn = 2;

-- Using DENSE_RANK (handles ties)
SELECT DISTINCT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as dr
    FROM employees
) ranked
WHERE dr = 2;
```

### 17. Find duplicate records
```sql
-- Find duplicates by email
SELECT email, COUNT(*) as count
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;

-- Find all duplicate rows
SELECT * FROM employees e1
WHERE EXISTS (
    SELECT 1 FROM employees e2
    WHERE e1.email = e2.email AND e1.id > e2.id
);
```

### 18. Delete duplicate records
```sql
-- Keep one row, delete duplicates (MySQL)
DELETE e1 FROM employees e1
INNER JOIN employees e2
ON e1.email = e2.email AND e1.id > e2.id;

-- Using CTE (SQL Server, PostgreSQL)
WITH cte AS (
    SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) as rn
    FROM employees
)
DELETE FROM cte WHERE rn > 1;
```

### 19. Find employees earning more than their manager
```sql
SELECT e.name AS employee, e.salary AS emp_salary,
       m.name AS manager, m.salary AS mgr_salary
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

### 20. Find department with highest average salary
```sql
SELECT d.department_name, AVG(e.salary) as avg_salary
FROM departments d
JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name
ORDER BY avg_salary DESC
LIMIT 1;
```

### 21. Find employees who joined in the same month
```sql
SELECT e1.name, e2.name, e1.hire_date
FROM employees e1
JOIN employees e2 ON MONTH(e1.hire_date) = MONTH(e2.hire_date)
    AND YEAR(e1.hire_date) = YEAR(e2.hire_date)
    AND e1.id < e2.id;
```

### 22. Find consecutive dates
```sql
-- Find users with consecutive login dates
SELECT DISTINCT user_id
FROM (
    SELECT user_id, login_date,
        login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) * INTERVAL '1 day' as grp
    FROM logins
) t
GROUP BY user_id, grp
HAVING COUNT(*) >= 3;  -- 3+ consecutive days
```

### 23. Pivot table (rows to columns)
```sql
-- MySQL
SELECT
    product,
    SUM(CASE WHEN month = 'Jan' THEN sales ELSE 0 END) as Jan,
    SUM(CASE WHEN month = 'Feb' THEN sales ELSE 0 END) as Feb,
    SUM(CASE WHEN month = 'Mar' THEN sales ELSE 0 END) as Mar
FROM monthly_sales
GROUP BY product;
```

### 24. Common Table Expressions (CTE)
```sql
-- Simple CTE
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 100000
)
SELECT department_id, COUNT(*)
FROM high_earners
GROUP BY department_id;

-- Recursive CTE (hierarchy)
WITH RECURSIVE emp_hierarchy AS (
    SELECT id, name, manager_id, 1 as level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN emp_hierarchy h ON e.manager_id = h.id
)
SELECT * FROM emp_hierarchy;
```

### 25. Find gaps in sequence
```sql
SELECT prev_id + 1 as gap_start, id - 1 as gap_end
FROM (
    SELECT id, LAG(id) OVER (ORDER BY id) as prev_id
    FROM orders
) t
WHERE id - prev_id > 1;
```

---

## Quick Tips for Interview

1. **Know the basics** - JOINs, WHERE vs HAVING, GROUP BY
2. **Practice window functions** - ROW_NUMBER, RANK, LAG, LEAD
3. **Understand execution order** - FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
4. **Handle NULLs** - COALESCE, IS NULL, proper NULL comparisons
5. **Write efficient queries** - Use indexes, avoid SELECT *, use EXISTS vs IN appropriately
