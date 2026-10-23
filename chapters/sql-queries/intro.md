# SQL Queries

Querying is asking a database meaningful questions in a way it understands. Almost everything an analyst does with a relational database starts with `SELECT`. This chapter builds up the `SELECT` statement clause by clause: choosing columns and rows, functions and expressions, sorting, aggregating with `GROUP BY` and `HAVING`, and combining tables with joins.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Write `SELECT` statements with `WHERE`, `ORDER BY`, aliases and row limits.
2. Use aggregate, string and date functions, predicates (`BETWEEN`, `IN`, `LIKE`, `IS NULL`) and `CASE`.
3. Explain the order in which a query's clauses are processed.
4. Summarize data with `GROUP BY` and filter groups with `HAVING`.
5. Combine tables with inner, left, right and full outer joins, and recognize a Cartesian product.
:::

## The example database: suppliers and parts

Most examples use a small, classic teaching database of suppliers, the parts they supply, and the shipments between them. Its relational model:

> **SUPPLIERS** ({u}`SID`, Sname, Status, Scity)
> **PARTS** ({u}`PID`, Pname, Color, Weight, Pcity)
> **SHIPMENTS** ({u}`SPID`, {u}`SID`, {u}`PID`, Quantity)
> FK (SID) ref SUPPLIERS, FK (PID) ref PARTS

**SUPPLIERS**

| SID | Sname | Status | Scity |
|---|---|---|---|
| S1 | Smith | 20 | London |
| S2 | Jones | 10 | Paris |
| S3 | Blake | 30 | Paris |
| S4 | Clark | 20 | London |
| S5 | Adams | 30 | Athens |
| S6 | Kevin | 25 | Lawrence |

**PARTS**

| PID | Pname | Color | Weight | Pcity |
|---|---|---|---|---|
| P1 | Nut | Red | 12 | London |
| P2 | Bolt | Green | 17 | Paris |
| P3 | Washer | Blue | 17 | Rome |
| P4 | Washer | Red | 14 | London |
| P5 | Cam | Blue | 12 | Paris |
| P6 | Cog | Red | 19 | London |
| P7 | Spipe | Black | 20 | Kansas City |
| P8 | Spipe | Gray | 15 | Topeka |

**SHIPMENTS**

| SPID | SID | PID | Quantity |
|---|---|---|---|
| SP0301 | S1 | P1 | 300 |
| SP0301 | S1 | P2 | 200 |
| SP0301 | S1 | P3 | 400 |
| SP0302 | S1 | P4 | 200 |
| SP0302 | S1 | P5 | 100 |
| SP0303 | S1 | P6 | 100 |
| SP0301 | S2 | P1 | 300 |
| SP0302 | S2 | P1 | 800 |
| SP0302 | S2 | P2 | 400 |
| SP0301 | S3 | P2 | 1200 |
| SP0301 | S4 | P2 | 200 |
| SP0302 | S4 | P2 | 200 |
| SP0303 | S4 | P2 | 200 |
| SP0302 | S4 | P4 | 300 |
| SP0302 | S4 | P5 | 400 |

To follow along in Oracle Cloud, create the three tables with the data types you think fit (see [SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md)), then load the rows.

:::{note} The data as INSERT statements
:class: dropdown
```sql
INSERT INTO suppliers VALUES ('S1', 'Smith', 20, 'London');
INSERT INTO suppliers VALUES ('S2', 'Jones', 10, 'Paris');
INSERT INTO suppliers VALUES ('S3', 'Blake', 30, 'Paris');
INSERT INTO suppliers VALUES ('S4', 'Clark', 20, 'London');
INSERT INTO suppliers VALUES ('S5', 'Adams', 30, 'Athens');
INSERT INTO suppliers VALUES ('S6', 'Kevin', 25, 'Lawrence');

INSERT INTO parts VALUES ('P1', 'Nut',    'Red',   12, 'London');
INSERT INTO parts VALUES ('P2', 'Bolt',   'Green', 17, 'Paris');
INSERT INTO parts VALUES ('P3', 'Washer', 'Blue',  17, 'Rome');
INSERT INTO parts VALUES ('P4', 'Washer', 'Red',   14, 'London');
INSERT INTO parts VALUES ('P5', 'Cam',    'Blue',  12, 'Paris');
INSERT INTO parts VALUES ('P6', 'Cog',    'Red',   19, 'London');
INSERT INTO parts VALUES ('P7', 'Spipe',  'Black', 20, 'Kansas City');
INSERT INTO parts VALUES ('P8', 'Spipe',  'Gray',  15, 'Topeka');

INSERT INTO shipments VALUES ('SP0301', 'S1', 'P1', 300);
INSERT INTO shipments VALUES ('SP0301', 'S1', 'P2', 200);
INSERT INTO shipments VALUES ('SP0301', 'S1', 'P3', 400);
INSERT INTO shipments VALUES ('SP0302', 'S1', 'P4', 200);
INSERT INTO shipments VALUES ('SP0302', 'S1', 'P5', 100);
INSERT INTO shipments VALUES ('SP0303', 'S1', 'P6', 100);
INSERT INTO shipments VALUES ('SP0301', 'S2', 'P1', 300);
INSERT INTO shipments VALUES ('SP0302', 'S2', 'P1', 800);
INSERT INTO shipments VALUES ('SP0302', 'S2', 'P2', 400);
INSERT INTO shipments VALUES ('SP0301', 'S3', 'P2', 1200);
INSERT INTO shipments VALUES ('SP0301', 'S4', 'P2', 200);
INSERT INTO shipments VALUES ('SP0302', 'S4', 'P2', 200);
INSERT INTO shipments VALUES ('SP0303', 'S4', 'P2', 200);
INSERT INTO shipments VALUES ('SP0302', 'S4', 'P4', 300);
INSERT INTO shipments VALUES ('SP0302', 'S4', 'P5', 400);
```
If you load the data from a spreadsheet instead, check the key columns for stray spaces: a value such as `'S6 '` looks like `'S6'` but will not match it in a join or a `WHERE` clause.
:::

## The SELECT statement

| Clause | Purpose |
|---|---|
| `SELECT` | the columns and expressions to return |
| `FROM` | the table(s) or view(s) to read |
| `WHERE` | the conditions a row must meet to be included |
| `GROUP BY` | how to group rows into categories |
| `HAVING` | the conditions a group must meet to be included |
| `ORDER BY` | how to sort the result |

Only `SELECT` and `FROM` are required. **Every query returns a table**, which is why queries can be nested inside other queries ([Advanced SQL Queries](../sql-advanced/intro.md)).

```sql
SELECT sname FROM suppliers;                 -- one column
SELECT sid, quantity FROM shipments;         -- several columns
SELECT * FROM parts;                         -- all columns
```

### Limiting rows

On large tables, `SELECT *` can run for a long time and return far more than you need. Look at the first few rows instead:

```sql
SELECT * FROM sh.sales
FETCH FIRST 5 ROWS ONLY;          -- Oracle (and standard SQL)
```

MySQL, PostgreSQL and SQLite use `LIMIT 5` instead.

### Aliases

A **column alias** gives a result column a readable name; a **table alias** shortens table names and removes ambiguity when several tables are involved.

```sql
SELECT s.sname AS "Supplier", s.scity AS "Supplier City"
FROM   suppliers s
WHERE  s.scity = 'Paris';
```

Double quotes keep spaces and capitalization in an alias; single quotes are for text values.

## Functions and expressions

**Aggregate and numeric functions**

| Function | Returns |
|---|---|
| `COUNT` | the number of rows (`COUNT(*)`) or of non-NULL values (`COUNT(col)`) |
| `SUM`, `AVG` | the total and the average of a numeric column |
| `MIN`, `MAX` | the smallest and largest value (also works on text and dates) |
| `ROUND(x, n)`, `TRUNC(x, n)` | x rounded or truncated to n decimal places |
| `MOD(a, b)` | the remainder of a divided by b |

```sql
SELECT COUNT(pid) FROM shipments;              -- 15 shipment rows
SELECT DISTINCT pid FROM shipments;            -- the six parts that were shipped
SELECT COUNT(DISTINCT pid) FROM shipments;     -- 6
SELECT AVG(weight) FROM parts;                 -- 15.75
SELECT MAX(sname) FROM suppliers;              -- 'Smith': the last name alphabetically
```

**String functions**

| Function | Example | Result |
|---|---|---|
| `LOWER`, `UPPER`, `INITCAP` | `INITCAP('kansas city')` | `Kansas City` |
| `SUBSTR(s, start, length)` | `SUBSTR('Washer', 3, 4)` | `sher` |
| `CONCAT(a, b)` or `a \|\| b` | `pid \|\| ': ' \|\| pname` | `P1: Nut` |
| `TRIM`, `LENGTH`, `REPLACE` | `TRIM('  S6 ')` | `S6` |
| `COALESCE(a, b, ...)` | `COALESCE(NULL, 'none')` | `none` (the first non-NULL value) |

**Date functions**

| Function | Example | Result |
|---|---|---|
| `NEXT_DAY(d, day)` | `NEXT_DAY(DATE '2026-09-15', 'SUNDAY')` | `2026-09-20` |
| `ADD_MONTHS(d, n)` | `ADD_MONTHS(DATE '2026-01-31', 1)` | `2026-02-28` |
| `MONTHS_BETWEEN(d1, d2)` | `MONTHS_BETWEEN(DATE '2026-07-01', DATE '2026-01-01')` | `6` |
| `EXTRACT(YEAR FROM d)` | `EXTRACT(YEAR FROM DATE '2026-09-15')` | `2026` |
| `TO_CHAR(d, format)` | `TO_CHAR(DATE '2026-09-15', 'YYYY-MM')` | `2026-09` |

To evaluate an expression without a table, select from **`DUAL`**, a one-row table that Oracle provides for this purpose: `SELECT NEXT_DAY(DATE '2026-09-15', 'SUNDAY') FROM dual;`

## Filtering rows

Conditions in `WHERE` use comparison operators (`=`, `<>` or `!=`, `>`, `>=`, `<`, `<=`) combined with `AND`, `OR` and `NOT`:

```sql
SELECT sid, status
FROM   suppliers
WHERE  scity = 'Paris' AND status > 20;        -- S3
```

**Predicates** make common conditions shorter:

```sql
WHERE quantity BETWEEN 100 AND 300         -- inclusive at both ends
WHERE quantity IN (100, 400)               -- any value in the list
WHERE pname LIKE 'W%'                      -- % = any number of characters (Washer)
WHERE sname LIKE 'Jo_es'                   -- _ = exactly one character (Jones)
WHERE pcity IS NULL                        -- NULL needs IS NULL, never = NULL
```

### Use parentheses

`AND` is evaluated before `OR`, so these two queries differ:

```sql
SELECT sname FROM suppliers
WHERE  (status >= 30 OR scity = 'Paris') AND sname = 'Jones';     -- Jones

SELECT sname FROM suppliers
WHERE  status >= 30 OR scity = 'Paris' AND sname = 'Jones';       -- Jones, Blake, Adams
```

The second is read as `status >= 30 OR (scity = 'Paris' AND sname = 'Jones')`. When `AND` and `OR` are mixed, always add parentheses.

### CASE

`CASE` is SQL's if/then logic. It has at least one `WHEN ... THEN` pair, an optional `ELSE`, and always ends with `END`:

```sql
SELECT sid, sname, status,
       CASE WHEN status >= 30 THEN 'Highly reliable'
            WHEN status >= 20 THEN 'Reliable'
            ELSE 'Satisfactory'
       END AS "Supplier Reputation"
FROM   suppliers;
```

Blake and Adams are highly reliable, Jones is satisfactory, and the others are reliable. Conditions are checked in order, so the second `WHEN` only sees statuses below 30.

### ORDER BY

```sql
SELECT scity, sname
FROM   suppliers
WHERE  status > 15
ORDER  BY scity, sname;            -- by city, then by name within a city
```

Sorting is ascending by default; add `DESC` for descending: `ORDER BY sname DESC`.

## How a query is processed

A query is written in the order `SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY`, but the database processes it in a different order:

1. `FROM`: gather the rows of the tables (and joins).
2. `WHERE`: keep only the rows that meet the conditions.
3. `GROUP BY`: form groups.
4. `HAVING`: keep only the groups that meet the conditions.
5. `SELECT`: compute the output columns.
6. `ORDER BY`: sort the result.


:::{figure} images/query_order.png
:label: fig-query-order
:alt: Two rows of boxes: the written order SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY, and the processing order FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY.
The order in which a query is written versus the order in which it is processed. Source: {download}`query_order.drawio <images/query_order.drawio>`.
:::

This explains several rules. `WHERE` cannot use an aggregate such as `SUM(quantity)` (groups do not exist yet), which is why `HAVING` exists. And `ORDER BY` can use column aliases defined in `SELECT`, but `WHERE` cannot.

## Aggregate queries

Suppose you want the number of suppliers in each city. This does not work:

```sql
SELECT scity, COUNT(sid) FROM suppliers;      -- error: not a single-group group function
```

The query asks for one count, but also for every city, and SQL has not been told how to split the count by city. `COUNT` without `GROUP BY` is a **scalar aggregate**: one value for the whole table. Adding `GROUP BY` makes it a **vector aggregate**: one value per group.

```sql
SELECT pid, SUM(quantity) AS total
FROM   shipments
GROUP  BY pid;
```

The database splits the shipments into one group per part, then sums each group:

| PID | total |
|---|---|
| P1 | 1400 |
| P2 | 2400 |
| P3 | 400 |
| P4 | 500 |
| P5 | 500 |
| P6 | 100 |

**Every column in `SELECT` must either be in `GROUP BY` or be inside an aggregate function.** `GROUP BY` can list several columns, and a query can compute several aggregates:

```sql
SELECT pcity, color,
       COUNT(*)              AS "Number of parts",
       SUM(weight * weight)  AS "Sum of squared weights"
FROM   parts
GROUP  BY pcity, color
ORDER  BY "Sum of squared weights" DESC;
```

The first row is London's three red parts (12² + 14² + 19² = 701).

### HAVING

`HAVING` filters groups, as `WHERE` filters rows:

```sql
SELECT pid, SUM(quantity)
FROM   shipments
GROUP  BY pid
HAVING SUM(quantity) > 500;          -- P1 (1400) and P2 (2400)
```

## Joins

A **join** combines two or more tables with a common domain into a single result. Joins are among the most important operations of an RDBMS, because normalized data is spread over many tables. Joins need **common columns**: usually the primary key of the parent table and the foreign key of the child. Here, `SUPPLIERS.SID` joins `SHIPMENTS.SID`, and `PARTS.PID` joins `SHIPMENTS.PID`. The common columns do not need to have the same names.

```sql
SELECT columns
FROM   table1 t1
[INNER | LEFT OUTER | RIGHT OUTER | FULL OUTER] JOIN table2 t2
       ON t1.common_column = t2.common_column;
```


:::{figure} images/join_types.png
:label: fig-join-types
:alt: Four pairs of overlapping circles A and B. Inner join shades only the overlap; left outer join shades all of A; right outer join shades all of B; full outer join shades both circles.
The four kinds of join. Shaded areas are the rows returned: matched rows are in the overlap, and unmatched rows of A or B are outside it.
:::

### Inner join

An **inner join** (or simply `JOIN`) returns only the rows whose common column values appear in **both** tables:

```sql
SELECT s.sid, s.sname, sp.pid, SUM(sp.quantity) AS total
FROM   suppliers s
JOIN   shipments sp ON s.sid = sp.sid
GROUP  BY s.sid, s.sname, sp.pid;
```

S5 and S6 have no shipments, so they do not appear.

### Left, right and full outer joins

An **outer join** also keeps the rows that have no match, filling the missing columns with NULL.

- A **left outer join** keeps every row of the left table:

```sql
SELECT s.sid, s.sname, SUM(sp.quantity) AS total
FROM   suppliers s
LEFT OUTER JOIN shipments sp ON s.sid = sp.sid
GROUP  BY s.sid, s.sname;
```

S1 to S4 have totals of 1300, 1500, 1200 and 1300; S5 and S6 appear with a NULL total. To show a message instead of NULL, use `COALESCE`, converting the number to text so both values have the same type:

```sql
COALESCE(TO_CHAR(SUM(sp.quantity)), 'Did not supply any parts') AS "Quantity"
```

- A **right outer join** keeps every row of the right table. Listing every part with its total quantity, including parts never shipped:

```sql
SELECT p.pid, p.pname, SUM(sp.quantity) AS total
FROM   shipments sp
RIGHT OUTER JOIN parts p ON sp.pid = p.pid
GROUP  BY p.pid, p.pname
ORDER  BY total DESC NULLS LAST;
```

P2 (2400) and P1 (1400) lead; P7 and P8 were never shipped and have NULL totals. (Writing it as `parts LEFT JOIN shipments` gives the same result.)

- A **full outer join** keeps the unmatched rows of **both** tables: here, parts never shipped, and any shipments of part IDs missing from `PARTS` (none, thanks to the foreign key).

### Cartesian product

If no join condition is given, every row of one table is paired with every row of the other: a **Cartesian product** of m × n rows.

```sql
SELECT p.pname, sp.quantity
FROM   parts p, shipments sp;            -- 8 × 15 = 120 rows, most of them meaningless
```

A Cartesian product is almost always a mistake, a forgotten join condition, and it can be enormous on real tables.

### Joining three tables

**Which cities did suppliers from Paris ship parts to?** Looking at the data: S2 and S3 are in Paris; they shipped P1 and P2; those parts are in London and Paris. In SQL, join all three tables:

```sql
SELECT DISTINCT p.pcity AS "Cities supplied to"
FROM   parts p
JOIN   shipments sp ON p.pid = sp.pid
JOIN   suppliers s  ON s.sid = sp.sid
WHERE  s.scity = 'Paris';                -- London, Paris
```

### The older join syntax

Joins can also be written with the tables listed in `FROM` and the join condition in `WHERE`, and Oracle marks the optional side of an outer join with `(+)`:

```sql
SELECT s.sname, sp.quantity
FROM   suppliers s, shipments sp
WHERE  s.sid = sp.sid;                   -- inner join

SELECT s.sname, sp.quantity
FROM   suppliers s, shipments sp
WHERE  s.sid = sp.sid (+);               -- left outer join: keep every supplier
```

You will meet this style in older code. The `JOIN ... ON` style is clearer and harder to get wrong (a forgotten `WHERE` condition silently becomes a Cartesian product), so prefer it.

Things to remember about joins: use a table alias for every table; joins can involve any number of tables; and a table can be joined to itself (a **self-join**, for example employees and their managers).

## Queries on a real database

The same techniques work unchanged on large tables. Total sales by channel class for each year in the SH schema ([The SH Sample Schema](../appendix/sh-schema.md)):

```sql
SELECT t.calendar_year,
       ch.channel_class,
       ROUND(SUM(s.amount_sold)) AS sales
FROM   sh.sales s
JOIN   sh.channels ch ON s.channel_id = ch.channel_id
JOIN   sh.times t     ON s.time_id = t.time_id
GROUP  BY t.calendar_year, ch.channel_class
ORDER  BY t.calendar_year, sales DESC;
```

## Common mistakes

- **A column in `SELECT` that is neither grouped nor aggregated** (Oracle: "not a GROUP BY expression").
- **Aggregates in `WHERE`**: use `HAVING`.
- **`= NULL`** instead of `IS NULL`. Nothing equals NULL, not even NULL.
- **Forgetting a join condition**, producing a Cartesian product.
- **An inner join where an outer join was needed**, silently dropping suppliers without shipments.
- **Mixing `AND` and `OR` without parentheses.**
- **`COUNT(column)` when you meant `COUNT(*)`**: the first ignores NULLs.

:::{warning} Check the AI
GenAI assistants write SQL that runs, and SQL that runs is not necessarily correct. The most common silent errors are a join that multiplies rows (joining two child tables to the same parent and then summing, so every amount is counted several times), an inner join that drops rows that should appear with zero, and filtering in `WHERE` on a column from the outer-joined table, which quietly turns an outer join back into an inner join. Check every generated query against a number you can verify independently: the table's row count, a total from another report, or a few rows checked by hand.
:::

## Practice

Questions 1 to 4 use the suppliers and parts tables; question 5 uses SH.

**1. Filtering.** List the PID, name and weight of red or blue parts weighing at least 14, heaviest first.

:::{note} Answer
:class: dropdown
```sql
SELECT pid, pname, weight
FROM   parts
WHERE  color IN ('Red', 'Blue') AND weight >= 14
ORDER  BY weight DESC;
```
P6 Cog (19), P3 Washer (17), P4 Washer (14).
:::

**2. Grouping with a join.** For each part color, show the number of shipments and the total quantity shipped, largest total first.

:::{note} Answer
:class: dropdown
```sql
SELECT p.color, COUNT(*) AS shipments, SUM(sp.quantity) AS total
FROM   parts p
JOIN   shipments sp ON p.pid = sp.pid
GROUP  BY p.color
ORDER  BY total DESC;
```
Green: 6 shipments, 2400. Red: 6, 2000. Blue: 3, 900. Black and Gray parts were never shipped, so they do not appear.
:::

**3. HAVING.** Which parts were shipped by more than one supplier? Show each part and its number of suppliers.

:::{note} Answer
:class: dropdown
```sql
SELECT pid, COUNT(DISTINCT sid) AS suppliers
FROM   shipments
GROUP  BY pid
HAVING COUNT(DISTINCT sid) > 1;
```
P1 (2 suppliers), P2 (4), P4 (2), P5 (2). `DISTINCT` matters: S2 shipped P1 twice, and S4 shipped P2 three times.
:::

**4. Outer join.** List the parts that have never been shipped, using a join.

:::{note} Answer
:class: dropdown
```sql
SELECT p.pid, p.pname
FROM   parts p
LEFT JOIN shipments sp ON p.pid = sp.pid
WHERE  sp.pid IS NULL;
```
P7 and P8 (the two Spipes).
:::

**5. SH.** Show the average list price and the number of products in each product category of `SH.PRODUCTS`, rounded to two decimals, highest average first. Then add a condition to show only categories whose average list price is above $100.

:::{note} One possible answer
:class: dropdown
```sql
SELECT prod_category,
       COUNT(*)                       AS products,
       ROUND(AVG(prod_list_price), 2) AS avg_list_price
FROM   sh.products
GROUP  BY prod_category
HAVING AVG(prod_list_price) > 100
ORDER  BY avg_list_price DESC;
```
:::

## Key terms

query, SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY, alias, FETCH FIRST, DISTINCT, aggregate function, scalar and vector aggregates, string function, date function, DUAL, predicate (BETWEEN, IN, LIKE, IS NULL), wildcard, CASE, logical processing order, join, common column, inner join, left/right/full outer join, Cartesian product, self-join, COALESCE

## Further reading

- Oracle, [SQL Language Reference: SELECT](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html).
- Hoffer, Ramesh and Topi, *Modern Database Management*, chapters "Introduction to SQL" and "Advanced SQL".
