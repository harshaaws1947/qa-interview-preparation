# SQL Cheatsheet

## SELECT Basics

```sql
-- Basic SELECT
SELECT column1, column2 FROM table_name;
SELECT * FROM table_name;

-- Aliases
SELECT first_name AS name FROM employees;
SELECT e.first_name FROM employees e;

-- DISTINCT
SELECT DISTINCT department FROM employees;

-- LIMIT
SELECT * FROM employees LIMIT 10;
SELECT * FROM employees LIMIT 10 OFFSET 5;  -- Skip first 5
```

## WHERE Clause

```sql
-- Comparison operators
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE salary >= 50000;
SELECT * FROM employees WHERE salary <> 50000;  -- Not equal

-- AND, OR, NOT
SELECT * FROM employees WHERE salary > 50000 AND department = 'IT';
SELECT * FROM employees WHERE department = 'IT' OR department = 'HR';
SELECT * FROM employees WHERE NOT department = 'IT';

-- IN, NOT IN
SELECT * FROM employees WHERE department IN ('IT', 'HR', 'Sales');
SELECT * FROM employees WHERE department NOT IN ('IT', 'HR');

-- BETWEEN
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;
SELECT * FROM employees WHERE hire_date BETWEEN '2020-01-01' AND '2020-12-31';

-- LIKE
SELECT * FROM employees WHERE name LIKE 'J%';      -- Starts with J
SELECT * FROM employees WHERE name LIKE '%son';    -- Ends with son
SELECT * FROM employees WHERE name LIKE '%am%';    -- Contains am
SELECT * FROM employees WHERE name LIKE 'J_n';     -- J?n (single char)

-- NULL
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

## ORDER BY

```sql
SELECT * FROM employees ORDER BY salary;              -- ASC (default)
SELECT * FROM employees ORDER BY salary DESC;         -- DESC
SELECT * FROM employees ORDER BY department, salary DESC;
SELECT * FROM employees ORDER BY 2;                   -- By column position
```

## Aggregate Functions

```sql
SELECT COUNT(*) FROM employees;                       -- Count rows
SELECT COUNT(DISTINCT department) FROM employees;     -- Count unique
SELECT SUM(salary) FROM employees;                    -- Sum
SELECT AVG(salary) FROM employees;                    -- Average
SELECT MIN(salary) FROM employees;                    -- Minimum
SELECT MAX(salary) FROM employees;                    -- Maximum
```

## GROUP BY & HAVING

```sql
-- GROUP BY
SELECT department, COUNT(*) AS count
FROM employees
GROUP BY department;

-- Multiple columns
SELECT department, job_title, AVG(salary)
FROM employees
GROUP BY department, job_title;

-- HAVING (filter groups)
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;

-- WHERE vs HAVING
SELECT department, AVG(salary)
FROM employees
WHERE status = 'active'          -- Filter rows before grouping
GROUP BY department
HAVING AVG(salary) > 50000;      -- Filter groups after grouping
```

## JOINs

```sql
-- INNER JOIN (matching rows only)
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- LEFT JOIN (all from left + matching from right)
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- RIGHT JOIN (all from right + matching from left)
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;

-- FULL OUTER JOIN (all from both)
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;

-- SELF JOIN
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- CROSS JOIN (cartesian product)
SELECT * FROM colors CROSS JOIN sizes;

-- Multiple JOINs
SELECT e.name, d.department_name, l.city
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN locations l ON d.location_id = l.id;
```

## Subqueries

```sql
-- In WHERE
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- IN subquery
SELECT * FROM employees
WHERE department_id IN (SELECT id FROM departments WHERE location = 'NYC');

-- EXISTS subquery
SELECT * FROM departments d
WHERE EXISTS (SELECT 1 FROM employees WHERE department_id = d.id);

-- In FROM (derived table)
SELECT dept_avg.department, dept_avg.avg_salary
FROM (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees GROUP BY department
) AS dept_avg
WHERE dept_avg.avg_salary > 50000;

-- Correlated subquery
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees
    WHERE department_id = e.department_id
);
```

## Window Functions

```sql
-- ROW_NUMBER
SELECT name, salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;

-- RANK (gaps after ties)
SELECT name, salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK (no gaps)
SELECT name, salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- PARTITION BY
SELECT name, department, salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- LAG / LEAD
SELECT name, salary,
    LAG(salary) OVER (ORDER BY hire_date) AS prev_salary,
    LEAD(salary) OVER (ORDER BY hire_date) AS next_salary
FROM employees;

-- Running total
SELECT name, salary,
    SUM(salary) OVER (ORDER BY hire_date) AS running_total
FROM employees;

-- NTILE (divide into buckets)
SELECT name, salary,
    NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;
```

## CTE (Common Table Expression)

```sql
-- Basic CTE
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 100000
)
SELECT department, COUNT(*) FROM high_earners GROUP BY department;

-- Multiple CTEs
WITH
    dept_totals AS (
        SELECT department_id, SUM(salary) AS total
        FROM employees GROUP BY department_id
    ),
    dept_avg AS (
        SELECT AVG(total) AS avg_total FROM dept_totals
    )
SELECT d.department_id, d.total
FROM dept_totals d, dept_avg a
WHERE d.total > a.avg_total;

-- Recursive CTE
WITH RECURSIVE hierarchy AS (
    SELECT id, name, manager_id, 1 AS level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN hierarchy h ON e.manager_id = h.id
)
SELECT * FROM hierarchy;
```

## Data Modification

```sql
-- INSERT
INSERT INTO employees (name, department, salary)
VALUES ('John', 'IT', 50000);

-- INSERT multiple rows
INSERT INTO employees (name, department, salary) VALUES
    ('John', 'IT', 50000),
    ('Jane', 'HR', 55000);

-- INSERT from SELECT
INSERT INTO archive_employees
SELECT * FROM employees WHERE status = 'inactive';

-- UPDATE
UPDATE employees SET salary = 55000 WHERE id = 1;
UPDATE employees SET salary = salary * 1.1 WHERE department = 'IT';

-- DELETE
DELETE FROM employees WHERE id = 1;
DELETE FROM employees WHERE status = 'inactive';
```

## Table Operations

```sql
-- CREATE TABLE
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    salary DECIMAL(10, 2),
    department_id INT,
    hire_date DATE DEFAULT CURRENT_DATE,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- ALTER TABLE
ALTER TABLE employees ADD COLUMN phone VARCHAR(20);
ALTER TABLE employees DROP COLUMN phone;
ALTER TABLE employees MODIFY COLUMN name VARCHAR(200);
ALTER TABLE employees RENAME COLUMN name TO full_name;

-- DROP TABLE
DROP TABLE employees;
DROP TABLE IF EXISTS employees;

-- TRUNCATE (faster delete all)
TRUNCATE TABLE employees;

-- CREATE INDEX
CREATE INDEX idx_salary ON employees(salary);
CREATE UNIQUE INDEX idx_email ON employees(email);
DROP INDEX idx_salary ON employees;
```

## Useful Functions

### String Functions
```sql
CONCAT(str1, str2)              -- Concatenate
LENGTH(str)                      -- String length
UPPER(str), LOWER(str)          -- Case conversion
TRIM(str)                        -- Remove whitespace
SUBSTRING(str, start, length)    -- Extract substring
REPLACE(str, old, new)           -- Replace
LEFT(str, n), RIGHT(str, n)      -- Extract from left/right
```

### Date Functions
```sql
NOW(), CURRENT_TIMESTAMP         -- Current datetime
CURDATE(), CURRENT_DATE          -- Current date
DATE(datetime)                   -- Extract date
YEAR(date), MONTH(date), DAY(date)
DATE_ADD(date, INTERVAL 1 DAY)
DATE_SUB(date, INTERVAL 1 MONTH)
DATEDIFF(date1, date2)           -- Difference in days
DATE_FORMAT(date, '%Y-%m-%d')
```

### Numeric Functions
```sql
ROUND(num, decimals)
CEILING(num), FLOOR(num)
ABS(num)
MOD(num, divisor)
POWER(num, exponent)
```

### NULL Handling
```sql
COALESCE(val1, val2, ...)       -- First non-null
IFNULL(val, default)             -- If null, use default (MySQL)
NULLIF(val1, val2)               -- NULL if val1 = val2
```

## Common Patterns

### Find Nth Highest
```sql
-- Using LIMIT OFFSET
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC LIMIT 1 OFFSET 1;  -- 2nd highest

-- Using DENSE_RANK
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t WHERE rnk = 2;
```

### Find Duplicates
```sql
SELECT email, COUNT(*) FROM employees
GROUP BY email HAVING COUNT(*) > 1;
```

### Delete Duplicates
```sql
DELETE e1 FROM employees e1
INNER JOIN employees e2
ON e1.email = e2.email AND e1.id > e2.id;
```

### Pivot Table
```sql
SELECT product,
    SUM(CASE WHEN month = 'Jan' THEN sales END) AS Jan,
    SUM(CASE WHEN month = 'Feb' THEN sales END) AS Feb
FROM sales GROUP BY product;
```

---

## Quick Reference

| Clause | Purpose |
|--------|---------|
| `SELECT` | Choose columns |
| `FROM` | Specify table(s) |
| `WHERE` | Filter rows |
| `GROUP BY` | Group rows |
| `HAVING` | Filter groups |
| `ORDER BY` | Sort results |
| `LIMIT` | Restrict rows |

### Execution Order
```
1. FROM / JOIN
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. DISTINCT
7. ORDER BY
8. LIMIT
```
