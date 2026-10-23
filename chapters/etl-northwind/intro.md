# ETL Example: Northwind

This chapter applies the five-step procedure of [ETL: From Database to Data Warehouse](../etl-procedure/intro.md) to a complete example: building a sales data warehouse from the **Northwind** database, entirely in SQL. Every statement runs in Oracle Cloud, and every count and total below was checked against the data.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Design a star schema for a business process in an operational database.
2. Create and load dimension tables with surrogate keys, including a junk dimension, a role-playing date dimension and a Type 3 attribute.
3. Build a fact table by looking up surrogate keys through a temporary fact table.
4. Add foreign keys, reconcile the warehouse with its source, and query the result.
:::

## The source database

Northwind is a sample database created by Microsoft for an imaginary food trading company, and it is used in many database courses ([Databases and the ER Model](../er-model/intro.md) reads its ER model). The course's Oracle version prefixes every table with `NW_`. The tables used here:

| Table | Rows | Contents |
|---|---|---|
| `NW_ORDERS` | 830 | one row per order: customer, employee, shipper (`ShipVia`), order and shipped dates |
| `NW_ORDERDETAILS` | 2,155 | one row per product on an order: unit price, quantity, discount |
| `NW_CUSTOMERS` | 91 | customer companies and their cities |
| `NW_PRODUCTS` | 77 | products, package sizes and list prices |
| `NW_EMPLOYEES` | 9 | sales staff: names, titles and cities |
| `NW_SHIPPERS` | 3 | shipping companies |

The orders run from 4 July 1996 to 6 May 1998. Twenty-one orders have not shipped yet, so their `ShippedDate` is NULL. `Discount` in `NW_ORDERDETAILS` is a **rate** between 0 and 1 (0.15 means 15%), and 838 of the 2,155 order lines have a discount.

## Step 1: design the warehouse

- **Business process**: sales, specifically the quantity and amount of each product sold, over time.
- **Data sources**: the Northwind tables above.
- **Schema**: a simple star schema.
- **Processing needs**: remove duplicates and NULLs where they would break keys; compute each line's billed amount; derive day, month and year for dates.
- **Grain**: **one product on one order**, that is, each product purchased in an order by a customer, handled by an employee and shipped by a shipper.
- **Dimensions**: Product, Customer, Employee, Shipper, Date (in two roles: order date and ship date), and a junk dimension for the discount flag. The order ID stays in the fact table as a degenerate dimension.
- **Facts**: quantity sold and billed amount.
- **Constraints**: a surrogate primary key on each dimension, and a foreign key from the fact table to each one.


:::{figure} images/northwind_star.png
:label: fig-northwind-star
:alt: The Northwind star schema: PRODUCT_SALES_FACT in the center, connected to PRODUCTS_DIM, CUSTOMERS_DIM, EMPLOYEES_DIM, SHIPPERS_DIM, JUNK_DIM, and twice to DATE_DIM for the order date and the ship date.
The Northwind sales star schema built in this chapter. Source: {download}`northwind_star.drawio <images/northwind_star.drawio>`.
:::

## Steps 2 and 3: staging and processing

Here the source tables are already in the same Oracle database, so they serve as the staging area, and the processing happens inside the warehouse with SQL as the tables are loaded. Strictly, that makes this an **ELT** process: load first, transform inside the target.

## Steps 4 and 5: dimensions

Each dimension gets an identity column as its surrogate key, keeps the Northwind ID as its business key, and is loaded with `INSERT ... SELECT`.

**Products**

```sql
CREATE TABLE products_dim (
    product_sk        NUMBER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    product_id        NUMBER,
    product_name      VARCHAR2(40) NOT NULL,
    quantity_per_unit VARCHAR2(20),
    unit_price        NUMBER,
    CONSTRAINT products_dim_pk PRIMARY KEY (product_sk)
);

INSERT INTO products_dim (product_id, product_name, quantity_per_unit, unit_price)
SELECT productid, productname, quantityperunit, unitprice
FROM   nw_products;                                         -- 77 rows
```

**Customers, with a Type 3 attribute.** The dimension keeps both the current city and the previous city, a Type 3 slowly changing dimension ([Advanced Data Warehouse Design](../dw-design-advanced/intro.md)). On the first load the previous city is simply the current one; later loads would move the current value into `old_customer_city` before overwriting it.

```sql
CREATE TABLE customers_dim (
    customer_sk       NUMBER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    customer_id       CHAR(5),
    customer_name     VARCHAR2(100) NOT NULL,
    customer_city     VARCHAR2(60),
    old_customer_city VARCHAR2(60),
    CONSTRAINT customers_dim_pk PRIMARY KEY (customer_sk)
);

INSERT INTO customers_dim (customer_id, customer_name, customer_city)
SELECT customerid, companyname, city
FROM   nw_customers;                                        -- 91 rows

UPDATE customers_dim SET old_customer_city = customer_city;  -- after the insert, not before
```

**Shippers**

```sql
CREATE TABLE shippers_dim (
    shipper_sk   NUMBER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    shipper_id   NUMBER,
    company_name VARCHAR2(50) NOT NULL,
    phone        VARCHAR2(24),
    CONSTRAINT shippers_dim_pk PRIMARY KEY (shipper_sk)
);

INSERT INTO shippers_dim (shipper_id, company_name, phone)
SELECT shipperid, companyname, phone
FROM   nw_shippers;                                         -- 3 rows
```

**Employees, with a transformation.** The source stores first and last names separately; the dimension combines them into one column with the concatenation operator.

```sql
CREATE TABLE employees_dim (
    employee_sk    NUMBER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    employee_id    NUMBER,
    employee_name  VARCHAR2(100),
    employee_title VARCHAR2(30),
    employee_city  VARCHAR2(15),
    CONSTRAINT employees_dim_pk PRIMARY KEY (employee_sk)
);

INSERT INTO employees_dim (employee_id, employee_name, employee_title, employee_city)
SELECT employeeid, firstname || ' ' || lastname, title, city
FROM   nw_employees;                                        -- 9 rows
```

**A junk dimension.** The only low-cardinality flag here is whether a line was discounted. A junk dimension lists each possible combination of flags once (here, just two rows), and fact rows point to the right one. With more flags (say, rush order and gift wrap), the dimension would list every combination.

```sql
CREATE TABLE junk_dim (
    junk_sk      NUMBER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    discount_ind CHAR(1) NOT NULL,
    CONSTRAINT junk_dim_pk PRIMARY KEY (junk_sk),
    CONSTRAINT junk_dim_ck CHECK (discount_ind IN ('Y', 'N'))
);

INSERT INTO junk_dim (discount_ind) VALUES ('N');
INSERT INTO junk_dim (discount_ind) VALUES ('Y');
```

**A role-playing date dimension.** Orders have an order date and a ship date; both refer to the same Date dimension. The dimension needs one row for every date that occurs in either role, without duplicates, so the two lists are combined with `UNION` (which removes duplicates), skipping the NULL ship dates of unshipped orders. Day, month and year are derived with date functions.

```sql
CREATE TABLE date_dim (
    date_sk     NUMBER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    sdate       DATE NOT NULL,
    day_of_week VARCHAR2(10),
    month       NUMBER(2),
    year        NUMBER(4),
    CONSTRAINT date_dim_pk PRIMARY KEY (date_sk),
    CONSTRAINT date_dim_uq UNIQUE (sdate)
);

INSERT INTO date_dim (sdate, day_of_week, month, year)
SELECT d, TO_CHAR(d, 'Dy'), EXTRACT(MONTH FROM d), EXTRACT(YEAR FROM d)
FROM  (SELECT orderdate AS d FROM nw_orders
       UNION
       SELECT shippeddate FROM nw_orders WHERE shippeddate IS NOT NULL)
ORDER  BY d;                                                -- 480 dates
```

## Step 5: the fact table

**First, a temporary fact table** at the declared grain, with the business keys and the measures. The billed amount applies the discount *rate*: quantity × unit price × (1 − discount).

```sql
CREATE TABLE temp_sales_fact AS
SELECT od.productid,
       o.customerid,
       o.employeeid,
       o.shipvia                                             AS shipperid,
       od.orderid,
       od.quantity,
       od.discount,
       ROUND(od.quantity * od.unitprice * (1 - od.discount), 2) AS billing_amount,
       o.orderdate,
       o.shippeddate
FROM   nw_orderdetails od
JOIN   nw_orders o ON od.orderid = o.orderid;              -- 2,155 rows
```

**Then the fact table**, by looking up each surrogate key through its business key. Two details matter:

- The **date dimension is joined twice**, once per role, with different aliases.
- The **ship date join is an outer join**, because unshipped orders have no ship date. An inner join would silently drop their 73 order lines.

```sql
CREATE TABLE product_sales_fact AS
SELECT pd.product_sk,
       cd.customer_sk,
       ed.employee_sk,
       sd.shipper_sk,
       odt.date_sk  AS order_date_sk,
       sdt.date_sk  AS ship_date_sk,
       jd.junk_sk,
       t.orderid    AS order_dd,                            -- degenerate dimension
       t.quantity,
       t.billing_amount
FROM   temp_sales_fact t
JOIN   products_dim  pd  ON t.productid  = pd.product_id
JOIN   customers_dim cd  ON t.customerid = cd.customer_id
JOIN   employees_dim ed  ON t.employeeid = ed.employee_id
JOIN   shippers_dim  sd  ON t.shipperid  = sd.shipper_id
JOIN   junk_dim      jd  ON jd.discount_ind = CASE WHEN t.discount <> 0 THEN 'Y' ELSE 'N' END
JOIN   date_dim      odt ON t.orderdate  = odt.sdate
LEFT   JOIN date_dim sdt ON t.shippeddate = sdt.sdate;      -- 2,155 rows
```

**Finally, the foreign keys.** The date dimension is referenced twice, once for each role.

```sql
ALTER TABLE product_sales_fact ADD (
    CONSTRAINT psf_product_fk    FOREIGN KEY (product_sk)    REFERENCES products_dim,
    CONSTRAINT psf_customer_fk   FOREIGN KEY (customer_sk)   REFERENCES customers_dim,
    CONSTRAINT psf_employee_fk   FOREIGN KEY (employee_sk)   REFERENCES employees_dim,
    CONSTRAINT psf_shipper_fk    FOREIGN KEY (shipper_sk)    REFERENCES shippers_dim,
    CONSTRAINT psf_junk_fk       FOREIGN KEY (junk_sk)       REFERENCES junk_dim,
    CONSTRAINT psf_order_date_fk FOREIGN KEY (order_date_sk) REFERENCES date_dim,
    CONSTRAINT psf_ship_date_fk  FOREIGN KEY (ship_date_sk)  REFERENCES date_dim
);
```

## Reconcile and use the warehouse

**Reconcile with the source.** The warehouse must account for every order line and every dollar:

```sql
SELECT COUNT(*), SUM(billing_amount) FROM temp_sales_fact;      -- 2155, 1265793.15
SELECT COUNT(*), SUM(billing_amount) FROM product_sales_fact;   -- 2155, 1265793.15
SELECT COUNT(*) FROM product_sales_fact WHERE ship_date_sk IS NULL;   -- 73 lines of unshipped orders
```

**View the star schema.** Reverse-engineer the result with the Data Modeler ([Advanced SQL Queries](../sql-advanced/intro.md)): add `PRODUCT_SALES_FACT` with dependencies, and the diagram shows the fact table with lines to all six dimension tables (two of them to `DATE_DIM`).

**Query it.** Star joins now answer business questions directly. Sales by shipper and year:

```sql
SELECT s.company_name, d.year, SUM(f.billing_amount) AS sales
FROM   product_sales_fact f
JOIN   shippers_dim s ON f.shipper_sk = s.shipper_sk
JOIN   date_dim d     ON f.order_date_sk = d.date_sk
GROUP  BY s.company_name, d.year
ORDER  BY d.year, sales DESC;
```

In 1997, the only complete year, United Package carried the most sales ($237,145.78), ahead of Federal Shipping and Speedy Express. And the role-playing date dimension makes "how long do orders take to ship?" a simple query:

```sql
SELECT ROUND(AVG(sdt.sdate - odt.sdate), 2) AS avg_days_to_ship
FROM  (SELECT DISTINCT order_dd, order_date_sk, ship_date_sk FROM product_sales_fact) f
JOIN   date_dim odt ON f.order_date_sk = odt.date_sk
JOIN   date_dim sdt ON f.ship_date_sk  = sdt.date_sk;       -- 8.49 days
```

(`DISTINCT` reduces the order lines to one row per order, so each order counts once.)

## Using data integration tools

This chapter built the ETL by hand in SQL. Commercial and open-source **data integration tools** do the same work through drag-and-drop interfaces and schedulers: Pentaho Data Integration (Kettle), Microsoft SQL Server Integration Services (SSIS), Oracle Data Integrator (ODI), Informatica, and cloud services such as AWS Glue or Azure Data Factory. Which one you use depends on your employer. Having built the steps by hand (dimensions, surrogate keys, key lookup, reconciliation), you will recognize them in any tool.

## Common mistakes

- **Subtracting the discount rate** (`quantity * price - discount`) instead of applying it. On Northwind this overstates total sales by about $88,500, or 7%, with no error message.
- **Inner-joining an optional key**, such as the ship date, and losing rows.
- **Updating before inserting**: an `UPDATE` on an empty table changes nothing.
- **Duplicate dates in the date dimension**, which multiply fact rows when joined. A `UNIQUE` constraint on `sdate` catches this.
- **Skipping reconciliation.** Always compare counts and totals with the source.

:::{warning} Check the AI
Ask a GenAI assistant to "write the ETL from Northwind to a star schema" and you will get a long script that runs. Read it with three questions. Does each measure match the business meaning of the source columns (is `Discount` a rate or an amount)? Does any inner join drop rows that should stay (unshipped orders, products without a category)? And do the warehouse's counts and totals reconcile with the source? An assistant cannot see your data, so it cannot check any of these for you.
:::

## Practice

**1. Grain check.** Why is the grain "one product on one order" rather than "one order"? Name one question each grain can answer that the other cannot.

:::{note} Answer
:class: dropdown
At the product-on-order grain you can analyze sales by product and category, and which products are bought together (using the order number). At the order grain you lose the products. The order grain is smaller and suits order-level questions such as freight cost per order, which belongs to the order rather than any product. Freight would therefore need its own fact table (or an allocation to lines).
:::

**2. Add a dimension.** Add a Category attribute to the product dimension, from `NW_CATEGORIES`, and reload it. Which product dimension attributes now form a hierarchy?

:::{note} One possible answer
:class: dropdown
```sql
ALTER TABLE products_dim ADD (category_name VARCHAR2(30));

UPDATE products_dim pd
SET    category_name = (SELECT c.categoryname
                        FROM   nw_products p
                        JOIN   nw_categories c ON p.categoryid = c.categoryid
                        WHERE  p.productid = pd.product_id);
```
Category, then product, form a hierarchy for drill-down. Profile the new column before using it: check for unexpected category names.
:::

**3. Query the warehouse.** Using only the warehouse tables, find the three employees with the highest total billed amount.

:::{note} Answer
:class: dropdown
```sql
SELECT e.employee_name, SUM(f.billing_amount) AS sales
FROM   product_sales_fact f
JOIN   employees_dim e ON f.employee_sk = e.employee_sk
GROUP  BY e.employee_name
ORDER  BY sales DESC
FETCH  FIRST 3 ROWS ONLY;
```
Margaret Peacock ($232,890.87), Janet Leverling ($202,812.87) and Nancy Davolio ($192,107.62).
:::

**4. Discount analysis.** Using the junk dimension, compare the number of order lines and total billed amount for discounted and undiscounted lines.

:::{note} Answer
:class: dropdown
```sql
SELECT j.discount_ind, COUNT(*) AS lines, SUM(f.billing_amount) AS sales
FROM   product_sales_fact f
JOIN   junk_dim j ON f.junk_sk = j.junk_sk
GROUP  BY j.discount_ind;
```
Undiscounted: 1,317 lines, $750,698.61. Discounted: 838 lines, $515,094.54.
:::

## Key terms

Northwind, staging area, ELT, surrogate key, identity column, business key, Type 3 attribute, junk dimension, role-playing dimension, UNION, temporary fact table, key lookup, degenerate dimension, outer join, reconciliation, data integration tools

## Further reading

- Kimball, R., & Caserta, J. (2004). *The Data Warehouse ETL Toolkit*. Wiley.
