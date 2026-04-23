# SQL Advanced Queries

Basic `SELECT` gets you surprisingly far. The features in this chapter — window functions, CTEs, recursive queries, CASE expressions, and analytical functions — are what separate a working analyst from a power user. Most production analytics SQL relies heavily on these tools.

## Common Table Expressions (CTEs)

A CTE is a named, temporary result set that exists only for the query it appears in. Syntax: `WITH name AS ( ... )`.

```sql
WITH monthly_sales AS (
    SELECT   DATE_TRUNC('month', order_date) AS month,
             SUM(total_amount)               AS revenue
    FROM     orders
    WHERE    status = 'paid'
    GROUP BY 1
)
SELECT  month,
        revenue,
        revenue - LAG(revenue) OVER (ORDER BY month) AS mom_change
FROM    monthly_sales
ORDER BY month;
```

Why use CTEs instead of nested subqueries?

- They read top-to-bottom like a pipeline
- You can reference them multiple times
- They are easier to debug (run the CTE alone)

## Window functions

A window function performs a calculation across a *set of rows related to the current row* — without collapsing the result like `GROUP BY` does.

```sql
SELECT customer_id,
       order_id,
       order_date,
       total_amount,
       SUM(total_amount) OVER (PARTITION BY customer_id ORDER BY order_date)
           AS running_total,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date)
           AS order_seq
FROM   orders;
```

Categories:

- **Ranking** — `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE(n)`
- **Offset** — `LAG(col, n)`, `LEAD(col, n)`, `FIRST_VALUE(col)`, `LAST_VALUE(col)`
- **Aggregate-as-window** — `SUM`, `AVG`, `COUNT`, `MIN`, `MAX` with `OVER (...)`

The `OVER` clause has three parts: `PARTITION BY` (reset groups), `ORDER BY` (within-group order), and an optional frame (`ROWS BETWEEN ... AND ...` or `RANGE BETWEEN ...`).

## CASE expressions

`CASE` adds conditional logic anywhere an expression is legal.

```sql
SELECT customer_id,
       SUM(total_amount)                                   AS lifetime_value,
       CASE
           WHEN SUM(total_amount) > 10000 THEN 'platinum'
           WHEN SUM(total_amount) >  1000 THEN 'gold'
           WHEN SUM(total_amount) >   100 THEN 'silver'
           ELSE                                'bronze'
       END                                                 AS tier
FROM   orders
GROUP BY customer_id;
```

Useful patterns:

- Pivoting with `SUM(CASE WHEN ... THEN ... END)` — turn rows into columns without a PIVOT operator
- Conditional aggregation (`COUNT(CASE WHEN status = 'paid' THEN 1 END)`) — count a subset within a single query

## Recursive CTEs

When your data represents a hierarchy (employees → managers, categories → subcategories, bill-of-materials), a recursive CTE walks the tree:

```sql
WITH RECURSIVE org_chart AS (
    -- anchor: top of the tree
    SELECT employee_id, manager_id, name, 1 AS level
    FROM   employees
    WHERE  manager_id IS NULL

    UNION ALL

    -- recursive step: one level down
    SELECT  e.employee_id, e.manager_id, e.name, oc.level + 1
    FROM    employees e
    JOIN    org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart ORDER BY level, name;
```

## Views and materialized views

- **View** — a saved `SELECT`; executes every time you query it. Great for simplifying access and for row/column-level security.
- **Materialized view** — a stored snapshot of the query's result; fast to read, must be refreshed.

Use views to hide joins and business logic behind a clean interface; use materialized views when the underlying query is expensive and freshness requirements are relaxed.

## Indexes — a quick performance aside

Indexes aren't query syntax, but they control whether your queries take 10 ms or 10 seconds. Index columns you:

- Filter on (`WHERE column = ...`)
- Join on (`JOIN ... ON column = ...`)
- Order by or group by in large queries

Indexes cost writes (every insert/update has to maintain them), so don't index everything.

## Learning outcomes

- Refactor nested subqueries into readable CTEs
- Apply window functions for running totals, rankings, and period-over-period comparisons
- Use `CASE` for conditional aggregation and simple pivots
- Traverse hierarchical data with recursive CTEs
- Decide between a view, a materialized view, and a direct query

