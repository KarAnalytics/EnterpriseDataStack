# Advanced SQL Queries

Some questions need more than one `SELECT`: "which suppliers shipped more than supplier S4?" first needs S4's total, then a comparison with every other supplier. This chapter covers **subqueries** (queries within queries) and the operators that work with them, then a set of other SQL tools every analyst meets: set operators, common table expressions, views, window functions, procedural SQL, and reverse engineering a data model from an existing database.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Write subqueries in the `SELECT`, `FROM`, `WHERE` and `HAVING` clauses.
2. Distinguish scalar from vector subqueries, and correlated from uncorrelated subqueries.
3. Use `IN`, `ANY`, `ALL`, `EXISTS` and `NOT EXISTS`, and avoid the `NOT IN` NULL trap.
4. Combine queries with `UNION`, `INTERSECT` and `MINUS`, and simplify them with `WITH`.
5. Create views, and explain dynamic versus materialized views.
6. Use window functions for rankings and running totals.
7. Reverse-engineer a data model with Oracle's Data Modeler.
:::

The examples use the suppliers and parts tables from [SQL Queries](../sql-queries/intro.md) and the SH sample schema.

## Subqueries

A **subquery** is a query inside another query. The **inner query** answers a sub-question; the **outer query** uses its answer.

*Which suppliers shipped more parts in total than supplier S4?* The sub-question is "how many parts did S4 ship?":

```sql
SELECT sp1.sid, s.sname, SUM(sp1.quantity) AS total
FROM   suppliers s
JOIN   shipments sp1 ON s.sid = sp1.sid
GROUP  BY sp1.sid, s.sname
HAVING SUM(sp1.quantity) >
       (SELECT SUM(sp2.quantity)
        FROM   shipments sp2
        WHERE  sp2.sid = 'S4');
```

S4 shipped 1,300, so only S2 (Jones, 1,500) qualifies.

*Which parts weigh less than the average part?*

```sql
SELECT p1.pid, p1.pname
FROM   parts p1
WHERE  p1.weight < (SELECT AVG(p2.weight) FROM parts p2);
```

The average is 15.75, so P1, P4, P5 and P8 qualify. You cannot write `WHERE weight < AVG(weight)` directly: aggregates are not allowed in `WHERE`, but a subquery that computes one is.

### Where a subquery can go

**In the WHERE or HAVING clause** (the most common): *suppliers who shipped part P2.*

```sql
SELECT *
FROM   suppliers
WHERE  sid IN (SELECT sp.sid FROM shipments sp WHERE sp.pid = 'P2');     -- S1, S2, S3, S4
```

**In the FROM clause**, where the subquery acts as a temporary table (an **inline view**): *shipments by suppliers in Paris.*

```sql
SELECT sp.sid, sp.pid, sp.quantity
FROM   shipments sp
JOIN   (SELECT s.sid FROM suppliers s WHERE s.scity = 'Paris') ps
       ON sp.sid = ps.sid;
```

**In the SELECT clause**, computing a value for each row: *each supplier, with the highest status of any supplier in the same city.*

```sql
SELECT s1.sid, s1.sname, s1.status, s1.scity,
       (SELECT MAX(s2.status)
        FROM   suppliers s2
        WHERE  s2.scity = s1.scity) AS "Max status in city"
FROM   suppliers s1;
```

Jones (status 10) is shown with 30, the status of Blake, the other Paris supplier.

### Scalar and vector subqueries

- A **scalar subquery** returns a single value (one row, one column), such as the average weight. It can be used wherever a value can: with `=`, `<`, `>`, or in `SELECT`.
- A **vector subquery** returns several rows (or columns), such as a list of supplier IDs. It must be used with an operator that accepts a list: `IN`, `ANY`, `ALL`, `EXISTS`, or in `FROM`.

Using `=` with a subquery that returns several rows fails with "single-row subquery returns more than one row".

### Uncorrelated and correlated subqueries

- An **uncorrelated** subquery does not depend on the outer query. It runs once, and the outer query uses the result. *Names of parts shipped by S4:*

```sql
SELECT p.pid, p.pname
FROM   parts p
WHERE  p.pid IN (SELECT sp.pid FROM shipments sp WHERE sp.sid = 'S4');    -- P2, P4, P5
```

- A **correlated** subquery refers to a column of the outer query, so conceptually it runs once for each outer row. *Parts heavier than the average part of the same color:*

```sql
SELECT p.pid, p.pname, p.color, p.weight
FROM   parts p
WHERE  p.weight > (SELECT AVG(p2.weight)
                   FROM   parts p2
                   WHERE  p2.color = p.color);       -- P3 (blue) and P6 (red)
```

The inner query uses `p.color` from the outer row, so each part is compared with the average of its own color.

A more involved correlated example: *parts whose total shipped quantity is less than the average total of all the other parts.*

```sql
SELECT p1.pid, p1.pname, SUM(sp1.quantity) AS total
FROM   parts p1
JOIN   shipments sp1 ON p1.pid = sp1.pid
GROUP  BY p1.pid, p1.pname
HAVING SUM(sp1.quantity) <
       (SELECT AVG(SUM(sp2.quantity))         -- Oracle allows nested aggregates
        FROM   shipments sp2
        WHERE  sp2.pid <> p1.pid
        GROUP  BY sp2.pid);
```

P3, P4, P5 and P6 qualify. For P3, for example, the other five parts average 980.

## IN, ANY, ALL and EXISTS

| Operator | True when |
|---|---|
| `x IN (list)` | x equals some value in the list |
| `x > ANY (list)` | x is greater than **at least one** value in the list |
| `x > ALL (list)` | x is greater than **every** value in the list |
| `EXISTS (subquery)` | the subquery returns at least one row |
| `NOT EXISTS (subquery)` | the subquery returns no rows |

*Suppliers whose status is greater than that of at least one supplier of P4 or P5:*

```sql
SELECT s1.sid, s1.sname
FROM   suppliers s1
WHERE  s1.status > ANY
       (SELECT s2.status
        FROM   suppliers s2
        JOIN   shipments sp2 ON s2.sid = sp2.sid
        WHERE  sp2.pid IN ('P4', 'P5'));
```

P4 and P5 were shipped by S1 and S4, both with status 20, so the result is S3, S5 and S6. Replacing `ANY` with `ALL` ("greater than every such supplier") gives the same result here only because both statuses are equal; with different statuses the two would differ.

*Suppliers who do not ship part P1* (`NOT IN`):

```sql
SELECT s.sid, s.sname
FROM   suppliers s
WHERE  s.sid NOT IN (SELECT sp.sid FROM shipments sp WHERE sp.pid = 'P1');    -- S3, S4, S5, S6
```

*Suppliers who have shipped at least one part* (`EXISTS`), and *suppliers who have shipped nothing* (`NOT EXISTS`):

```sql
SELECT *
FROM   suppliers s
WHERE  EXISTS (SELECT 1 FROM shipments sp WHERE sp.sid = s.sid);        -- S1 to S4

SELECT *
FROM   suppliers s
WHERE  NOT EXISTS (SELECT 1 FROM shipments sp WHERE sp.sid = s.sid);    -- S5, S6
```

`EXISTS` subqueries are correlated, and they only check whether a row exists, so what they select does not matter (`SELECT 1` is a common convention). The same questions can be answered with joins: `SELECT DISTINCT s.* ... JOIN shipments` for the first, and a `LEFT JOIN ... WHERE sp.sid IS NULL` for the second.

### The NOT IN trap

If the subquery of a `NOT IN` returns even one NULL, the whole condition is never true, and the query returns **no rows**. Suppose a shipment of P1 was recorded with a missing supplier ID. Then `NOT IN` above returns nothing at all, because `'S3' NOT IN ('S1', 'S2', NULL)` is not true but *unknown*. `NOT EXISTS` is unaffected and still returns S3, S4, S5 and S6. Prefer `NOT EXISTS`, or add `WHERE sp.sid IS NOT NULL` inside the subquery.

## Set operators

`UNION`, `INTERSECT` and `MINUS` combine the results of two queries that return the same columns (same number, compatible types):

| Operator | Returns |
|---|---|
| `UNION` | rows in either result (duplicates removed; `UNION ALL` keeps them) |
| `INTERSECT` | rows in both results |
| `MINUS` | rows in the first result but not the second (`EXCEPT` in standard SQL and most other databases) |

```sql
(SELECT * FROM suppliers WHERE status < 20 OR scity = 'Paris')     -- S2, S3
UNION
(SELECT * FROM suppliers WHERE sid IN ('S2', 'S4'));               -- S2, S4
```

`UNION` gives S2, S3 and S4; `INTERSECT` gives S2; and `MINUS` gives S3. *Which cities have both suppliers and parts?*

```sql
SELECT scity FROM suppliers
INTERSECT
SELECT pcity FROM parts;          -- London, Paris
```

## Common table expressions

A `WITH` clause names a subquery so the main query can use it like a table. It makes multi-step queries readable, and a named result can be used more than once. *Suppliers whose total shipped quantity is above the average supplier's total:*

```sql
WITH supplier_totals AS (
    SELECT sid, SUM(quantity) AS total
    FROM   shipments
    GROUP  BY sid
)
SELECT sid, total
FROM   supplier_totals
WHERE  total > (SELECT AVG(total) FROM supplier_totals);       -- S2 (1500; the average is 1325)
```

## Views

A **view** is a named query, stored in the database, that behaves like a table. Its content is generated each time it is used, so it is a virtual table with no rows of its own.

```sql
CREATE VIEW red_parts AS
SELECT pid, pname, weight
FROM   parts
WHERE  color = 'Red';

SELECT * FROM red_parts WHERE weight > 12;      -- P4, P6
```

- A **base table** contains the raw data.
- A **dynamic view** stores only its query; the data comes from the base tables when the view is used, so it is always current and takes almost no space.
- A **materialized view** stores a copy of the query's result. It is fast to query, but must be refreshed periodically to stay in step with the base tables. Data warehouses use materialized views for summaries ([Data Warehousing Concepts](../dw-concepts/intro.md)).

Views simplify complex queries (users query a view instead of rewriting a five-table join), provide each user a customized picture of the data, keep programs working when base tables change (physical data independence), and can hide columns or rows from users who should not see them. `WITH CHECK OPTION` stops inserts and updates through the view that would create rows the view cannot see. Views help with security, but should not be the only security measure: use privileges ([SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md)) as well.

## Window functions

A **window function** computes a value for each row from a set of related rows (its *window*) without collapsing the rows into groups the way `GROUP BY` does. Windows are defined with `OVER (PARTITION BY ... ORDER BY ...)`. They are used for rankings, running totals, moving averages and comparisons with a group's total.

*Rank the parts by total quantity shipped:*

```sql
SELECT pid,
       SUM(quantity) AS total,
       RANK() OVER (ORDER BY SUM(quantity) DESC) AS rnk
FROM   shipments
GROUP  BY pid;
```

P2 ranks 1 and P1 ranks 2; P4 and P5 tie at 500, so both rank 3, and P3 ranks 5. (`DENSE_RANK` would rank P3 fourth; `ROW_NUMBER` never ties.)

*The top-selling product in each SH category in 2001:*

```sql
WITH product_sales AS (
    SELECT p.prod_category, p.prod_name, SUM(s.amount_sold) AS sales
    FROM   sh.sales s
    JOIN   sh.products p ON s.prod_id = p.prod_id
    JOIN   sh.times t    ON s.time_id = t.time_id
    WHERE  t.calendar_year = 2001
    GROUP  BY p.prod_category, p.prod_name
)
SELECT prod_category, prod_name, ROUND(sales) AS sales
FROM  (SELECT ps.*,
              RANK() OVER (PARTITION BY prod_category ORDER BY sales DESC) AS rnk
       FROM   product_sales ps)
WHERE  rnk = 1
ORDER  BY sales DESC;
```

In Hardware, for example, the leader is the Envoy Ambassador, at about $3.45 million.

*A running total of monthly sales in 2001:*

```sql
SELECT t.calendar_month_desc AS month,
       SUM(s.amount_sold) AS sales,
       SUM(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_month_desc) AS running_total
FROM   sh.sales s
JOIN   sh.times t ON s.time_id = t.time_id
WHERE  t.calendar_year = 2001
GROUP  BY t.calendar_month_desc
ORDER  BY month;
```

## SQL inside programs

- **Embedded SQL**: SQL statements written into a program in another language, such as Python, R, Java or C, which sends them to the database and processes the results.
- **Dynamic SQL**: a program builds the SQL statement as it runs, for example from a user's search choices. Build dynamic SQL with **bind variables** for user input, never by pasting the input into the SQL text, or the application is open to **SQL injection**.

## Routines and triggers (PL/SQL)

Databases can also store program modules. In Oracle they are written in **PL/SQL**, Oracle's procedural extension of SQL:

- **Functions** take input parameters and return a value.
- **Procedures** take input or output parameters and perform actions, without returning a value.
- **Triggers** run automatically in response to a database event (`INSERT`, `UPDATE` or `DELETE`) on a table.

A trigger that records every change to an adoption fee in an audit table:

```sql
CREATE OR REPLACE TRIGGER adoption_fee_audit
AFTER UPDATE OF fee ON adoption
FOR EACH ROW
BEGIN
    INSERT INTO fee_audit (adoption_id, old_fee, new_fee, changed_at)
    VALUES (:OLD.adoption_id, :OLD.fee, :NEW.fee, SYSTIMESTAMP);
END;
/
```

## Reverse engineering a data model

When you inherit a database, a diagram of it is the fastest way to understand it. Oracle SQL Developer (web and desktop) includes a **Data Modeler** that can draw ER models from scratch or **reverse-engineer** the data model of existing tables. In Oracle Cloud:

1. In Database Actions, open the menu at the top left and choose **Data Modeler**.
2. In the navigator, right-click a table and choose **Add Object with dependencies to Diagram**. Oracle adds the table and every table it references.
3. Read the result: primary key columns are marked **P**, foreign key constraints are listed under each table, and the lines show which table references which.

Compare the diagram with the `CREATE TABLE` statements: every `FOREIGN KEY ... REFERENCES` clause should appear as a line. A reverse-engineered diagram shows the *physical* design (data types, constraint names), not the business rules: it cannot tell you whether a relationship is optional or why it exists. That is what the ER model of [Part III](../er-model/intro.md) is for.

## Common mistakes

- **`=` with a subquery that returns several rows.** Use `IN`.
- **`NOT IN` with a subquery that can return NULL.** Use `NOT EXISTS`.
- **Correlated subqueries where a join would do.** On large tables, a join or a `WITH` query is often much faster.
- **Set operators with mismatched columns.** Both queries must return the same number of columns, in the same order, with compatible types.
- **Expecting a view to store data.** A dynamic view recomputes its query every time.
- **Window functions in `WHERE`.** They are computed after `WHERE`; filter on them in an outer query, as in the top-product example.

:::{warning} Check the AI
GenAI assistants often answer "find the top X in each group" with a correlated subquery or a `GROUP BY` that silently drops ties, and they reach for `NOT IN` where `NOT EXISTS` is safe. They also mix dialects: `LIMIT`, `EXCEPT` and `TOP` work in other databases but `MINUS` and `FETCH FIRST` are Oracle's. Ask the assistant how its query handles ties and NULLs, then test it: add a row that creates a tie or a NULL, and check that the result changes the way it should.
:::

## Practice

Questions 1 to 4 use the suppliers and parts tables; question 5 uses SH.

**1. Scalar subquery.** List the shipments whose quantity is above the average quantity of all shipments.

:::{note} Answer
:class: dropdown
```sql
SELECT spid, sid, pid, quantity
FROM   shipments
WHERE  quantity > (SELECT AVG(quantity) FROM shipments);
```
The average is about 353.3, so five shipments qualify: the three of 400 (S1's P3, S2's P2 and S4's P5), and those of 800 and 1200.
:::

**2. Correlated subquery.** List the shipments whose quantity is above the average quantity shipped *of the same part*.

:::{note} Answer
:class: dropdown
```sql
SELECT sp.sid, sp.pid, sp.quantity
FROM   shipments sp
WHERE  sp.quantity > (SELECT AVG(sp2.quantity)
                      FROM   shipments sp2
                      WHERE  sp2.pid = sp.pid);
```
S2's 800 of P1, S3's 1200 of P2, S4's 300 of P4 and S4's 400 of P5.
:::

**3. NOT EXISTS.** List the parts that no supplier in London has ever shipped.

:::{note} Answer
:class: dropdown
```sql
SELECT p.pid, p.pname
FROM   parts p
WHERE  NOT EXISTS (SELECT 1
                   FROM   shipments sp
                   JOIN   suppliers s ON s.sid = sp.sid
                   WHERE  sp.pid = p.pid AND s.scity = 'London');
```
P7 and P8. London suppliers S1 and S4 together shipped P1 to P6.
:::

**4. Window function.** For each supplier, show each of its shipments with its share of that supplier's total quantity, as a percentage.

:::{note} One possible answer
:class: dropdown
```sql
SELECT sid, pid, quantity,
       ROUND(100 * quantity / SUM(quantity) OVER (PARTITION BY sid), 1) AS pct_of_supplier
FROM   shipments
ORDER  BY sid, pct_of_supplier DESC;
```
For S3 the single shipment is 100%; S1's largest share is P3, with 400 of 1,300 (30.8%).
:::

**5. A view on SH.** Create a view `yearly_category_sales` with the total `AMOUNT_SOLD` by calendar year and product category. Then query it with a window function to show each category's sales and its rank within each year.

:::{note} One possible answer
:class: dropdown
```sql
CREATE VIEW yearly_category_sales AS
SELECT t.calendar_year, p.prod_category, SUM(s.amount_sold) AS sales
FROM   sh.sales s
JOIN   sh.times t    ON s.time_id = t.time_id
JOIN   sh.products p ON s.prod_id = p.prod_id
GROUP  BY t.calendar_year, p.prod_category;

SELECT calendar_year, prod_category, ROUND(sales) AS sales,
       RANK() OVER (PARTITION BY calendar_year ORDER BY sales DESC) AS rnk
FROM   yearly_category_sales
ORDER  BY calendar_year, rnk;
```
:::

## Key terms

subquery, inner and outer query, inline view, scalar subquery, vector subquery, uncorrelated subquery, correlated subquery, IN, ANY, ALL, EXISTS, NOT EXISTS, NOT IN NULL trap, set operators (UNION, UNION ALL, INTERSECT, MINUS), common table expression (WITH), view, base table, dynamic view, materialized view, WITH CHECK OPTION, window function, OVER, PARTITION BY, RANK, DENSE_RANK, ROW_NUMBER, embedded SQL, dynamic SQL, bind variable, SQL injection, PL/SQL, function, procedure, trigger, reverse engineering, Data Modeler

## Further reading

- Oracle, [SQL Language Reference: Analytic Functions](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Analytic-Functions.html).
- Hoffer, Ramesh and Topi, *Modern Database Management*, chapter "Advanced SQL".
