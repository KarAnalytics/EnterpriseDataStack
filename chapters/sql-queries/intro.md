# SQL Queries

The single most-used SQL statement in an analyst's life is `SELECT`. This chapter builds fluency with the full query vocabulary: projection, filtering, sorting, grouping, aggregation, and joins across multiple tables.

## The anatomy of a SELECT

Every `SELECT` statement has a canonical clause order. You *write* them in this order, but the database *executes* them in a different logical order (below each clause).

```sql
SELECT     column_list          -- 5. project columns
FROM       table_list            -- 1. pick source tables
WHERE      row_filter            -- 2. filter rows
GROUP BY   grouping_columns      -- 3. collapse rows into groups
HAVING     group_filter          -- 4. filter groups
ORDER BY   sort_columns          -- 6. sort the result
LIMIT      n                     -- 7. keep first n rows
```

Knowing the logical execution order explains many otherwise-confusing error messages (e.g., "why can't I reference an alias in WHERE?").

## Filtering — WHERE

```sql
SELECT *
FROM   orders
WHERE  order_date >= '2025-01-01'
  AND  status IN ('paid', 'shipped')
  AND  total_amount > 100;
```

- Comparison operators: `= <> < <= > >=`
- Logical: `AND`, `OR`, `NOT`
- Set membership: `IN`, `NOT IN`
- Range: `BETWEEN low AND high`
- Pattern matching: `LIKE 'A%'` (starts with A), `LIKE '_a%'` (second letter a)
- Null handling: `IS NULL`, `IS NOT NULL` — *never* `= NULL`

## Aggregation — GROUP BY and HAVING

Aggregate functions collapse many rows into one: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.

```sql
SELECT   customer_id,
         COUNT(*)         AS order_count,
         SUM(total_amount) AS lifetime_value
FROM     orders
WHERE    status = 'paid'
GROUP BY customer_id
HAVING   SUM(total_amount) > 1000
ORDER BY lifetime_value DESC;
```

Rules of thumb:

- Every non-aggregated column in `SELECT` must appear in `GROUP BY`
- `WHERE` filters rows *before* aggregation; `HAVING` filters groups *after*
- `COUNT(*)` counts rows; `COUNT(column)` counts non-null values; `COUNT(DISTINCT column)` counts unique non-null values

## Joining tables

The relational model normalizes data into multiple tables. Joins stitch them back together.

- **INNER JOIN** — only rows that match in both tables
- **LEFT JOIN** — all rows from the left table; `NULL` on the right where no match
- **RIGHT JOIN** — symmetric to `LEFT`
- **FULL OUTER JOIN** — all rows from both sides
- **CROSS JOIN** — Cartesian product (rarely what you want)

```sql
SELECT c.customer_id, c.first_name, o.order_id, o.total_amount
FROM   customers c
LEFT JOIN orders o
       ON o.customer_id = c.customer_id
WHERE  c.created_at >= '2025-01-01'
ORDER BY c.customer_id, o.order_id;
```

Join pitfalls to watch for:

- Forgetting the `ON` clause produces an accidental cross join — your row count explodes
- A `WHERE` filter on the right-hand table of a `LEFT JOIN` silently converts it to an `INNER JOIN`; push such filters into the `ON` clause if you want to keep unmatched left rows

## Subqueries

A subquery is a `SELECT` nested inside another statement:

```sql
SELECT *
FROM   customers
WHERE  customer_id IN (
    SELECT customer_id
    FROM   orders
    WHERE  total_amount > 500
);
```

Forms you'll see often:

- **Scalar subquery** — returns one row, one column (usable in `SELECT` or `WHERE`)
- **Row subquery** — returns one row
- **Table subquery** — returns many rows (usable in `FROM`, or with `IN`/`EXISTS`)
- **Correlated subquery** — references columns from the outer query; re-evaluated per outer row

`EXISTS` is often clearer and faster than `IN` for large result sets:

```sql
SELECT *
FROM   customers c
WHERE  EXISTS (
    SELECT 1 FROM orders o
    WHERE  o.customer_id = c.customer_id
      AND  o.total_amount > 500
);
```

## Set operations

- `UNION` — rows from either query (deduplicates)
- `UNION ALL` — rows from either query (keeps duplicates; faster)
- `INTERSECT` — rows in both
- `EXCEPT` / `MINUS` — rows in the first but not the second

Each side must have matching column counts and compatible types.

## Learning outcomes

- Read and write multi-clause `SELECT` statements
- Choose the right join type for a given question
- Aggregate with `GROUP BY` / `HAVING` and avoid the common "ambiguous column" trap
- Decompose complex questions using subqueries and `EXISTS`
- Combine result sets with `UNION`, `INTERSECT`, and `EXCEPT`

