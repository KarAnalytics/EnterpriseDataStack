# ETL: From Database to Data Warehouse

A data warehouse design ([Part IV](../dw-design/intro.md)) is only a plan until data flows into it. **Extract, transform, load (ETL)** is the process that moves data from operational databases and other sources into the warehouse: extracting it, transforming it into the warehouse's format and quality, and loading it into the fact and dimension tables. This chapter explains the three stages, the newer ELT alternative, and a five-step procedure for building a warehouse from a database with SQL alone. The [next chapter](../etl-northwind/intro.md) applies the procedure to the complete Northwind example.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Describe the extract, transform and load stages, and the role of the staging area.
2. Distinguish full and incremental extraction and loading, and online and offline extraction.
3. List record-level, field-level and data-quality transformations.
4. Contrast ETL with ELT.
5. Follow the five-step procedure to build a star schema from a database, including surrogate keys and a temporary fact table.
:::

## What ETL does

**Extract, transform and load** is the process of extracting data from different sources, transforming it into a meaningful form (with calculations, concatenations, cleaning and so on), and loading it into the data warehouse. ETL automates the movement of data from the OLTP systems that run the business into the OLAP system that analyzes it ([Data Warehousing Concepts](../dw-concepts/intro.md)).

ETL is where most of the effort in a data warehouse project goes, often estimated at around **80%** of the total. The work is usually done in a **staging area**: a separate storage area (database tables, files, or cloud storage) where extracted data is held while it is transformed, so that neither the source systems nor the warehouse are disturbed.


:::{figure} images/etl_flow.png
:label: fig-etl-flow
:alt: A flow from operational databases, files and external data, through a staging area where data is transformed, into a data warehouse of dimension and fact tables, and on to reports and analytics.
Extract, transform, load. Source: {download}`etl_flow.drawio <images/etl_flow.drawio>`.
:::

Throughout, the ETL process should be **documented**, and its **metadata** recorded: where each warehouse column comes from, which rules transformed it, and when it was loaded.

## Extraction

Extraction retrieves the required data from the source systems with as few resources as possible, since the sources are busy running the business.

- **Full extraction** takes all the data. It is used the first time a warehouse is loaded, and reflects everything currently in the source.
- **Incremental extraction** takes only what has changed since the last successful extraction, tracked by timestamps, change logs or triggers. This is called **changed data capture (CDC)**.
- **Online extraction** reads directly from the source system, without an external staging area.
- **Offline extraction** has the source system dump data to flat files, which are then extracted into the warehouse.

## Transformation

Transformation converts data from the format of the source systems to the format of the warehouse. It includes:

- **Record-level functions**: selecting and partitioning rows, joining tables, normalizing or denormalizing, and aggregating.
- **Field-level functions**: one-to-one transformations (converting a code to a description, a date to a standard format) and many-to-one transformations (combining first and last name into one column, or `quantity * unit_price` into a line amount).
- **Complex, programmatic transformations**, written in a programming language or procedural SQL.

### Transformation for data quality

Warehouse data should be **correct, unambiguous, consistent and complete**. The data processing and profiling of [Part II](../data-processing/intro.md) happen here:

- **Anomaly detection**.
- **Enforcing column properties**: NULLs in required columns, numeric values outside expected highs and lows, values that are unusually short or long, and values outside a valid set (a state code of `XX`).
- **Other processes**: decoding, reformatting, time stamping, unit conversion, **surrogate key generation**, merging records from several sources, **error detection and logging**, and locating missing data.

## Loading

Loading writes the transformed data into the warehouse's fact and dimension tables, correctly and with as few resources as possible.

- A **full load** writes the entire dataset, usually the very first time.
- An **incremental load** adds only the delta, the difference between source and target, at regular intervals (nightly, hourly): for example, only rows changed after the last extract date. Dimension changes are applied with the slowly changing dimension rules of [Advanced Data Warehouse Design](../dw-design-advanced/intro.md).

## ETL versus ELT

**Extract, load, transform (ELT)** reverses the last two steps: raw data is loaded first, often into a data lake on the same cloud platform as the warehouse, and transformed *inside* the target system when it is needed. ELT became practical because cloud storage made keeping raw data cheap, and cloud warehouses have the processing power to transform data where it sits.

| | ETL | ELT |
|---|---|---|
| Where data is transformed | in a staging area, before loading | in the target warehouse or lake, after loading |
| What is stored | transformed data | raw data, plus transformed versions |
| Flexibility | transformations fixed up front | new transformations can be applied to raw data later |
| Typical platform | traditional warehouse with an ETL server | cloud warehouse or lakehouse |

## A five-step procedure: database to data warehouse

Commercial ETL tools move data automatically or semi-automatically. This book builds the ETL **by hand, in SQL**. It is slower, but it shows exactly what every drag-and-drop tool is doing, and the skills carry over directly.

### Step 1: design the data warehouse

Follow the design steps of [Data Warehouse Design](../dw-design/intro.md): select the business process, determine the sources, choose the schema, list the processing needs, declare the grain, choose the dimensions, identify the facts, and define the tables and constraints.

### Step 2: extract the raw data into a staging area

Download or copy the source data into the staging area: your own schema, local files, or cloud storage. In Oracle, Database Actions can export query results to files, and source tables can be copied with `CREATE TABLE ... AS SELECT`.

### Step 3: process the raw data

Correct values, replace NULLs, transform values (for example, taking logarithms), aggregate, and concatenate, as the design requires. In ELT, this step happens later, inside the warehouse.

### Step 4: create the warehouse tables

Create a table for each dimension, with an automatically generated **surrogate key**, and keep the source system's key as a **business key**. In Oracle, an identity column generates the surrogate key:

```sql
CREATE TABLE store_dim (
    store_sk    NUMBER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    store_id    CHAR(4)       NOT NULL,           -- business key from the source
    store_name  VARCHAR2(50)  NOT NULL,
    city        VARCHAR2(40),
    state       CHAR(2),
    CONSTRAINT store_dim_pk PRIMARY KEY (store_sk),
    CONSTRAINT store_dim_uq UNIQUE (store_id)
);
```

### Step 5: load the dimensions and the facts

Load each dimension from the staging tables with `INSERT ... SELECT`, letting the identity column assign the keys:

```sql
INSERT INTO store_dim (store_id, store_name, city, state)
SELECT store_id, store_name, city, state
FROM   stg_stores;
```

The fact table is harder: the source transactions carry *business* keys (store ID, product SKU, date), but the fact table needs *surrogate* keys. The standard technique has two parts:

1. **Create a temporary, denormalized fact table** by joining the source transaction tables, so that each row holds the business keys and the measures at the declared grain:

```sql
CREATE TABLE temp_sales AS
SELECT t.store_id, t.sku, t.sale_date, t.quantity, t.quantity * t.unit_price AS sales_amount
FROM   stg_transactions t
JOIN   stg_stores s ON s.store_id = t.store_id;
```

2. **Create the fact table by looking up the surrogate keys**: join the temporary table to each dimension on its business key, and keep only the surrogate keys and the measures:

```sql
CREATE TABLE sales_fact AS
SELECT sd.store_sk, pd.product_sk, dd.date_sk, ts.quantity, ts.sales_amount
FROM   temp_sales ts
JOIN   store_dim   sd ON ts.store_id  = sd.store_id
JOIN   product_dim pd ON ts.sku       = pd.sku
JOIN   date_dim    dd ON ts.sale_date = dd.full_date;
```

Finally, **add the foreign keys** from the fact table to each dimension, so the database enforces the star schema's referential integrity:

```sql
ALTER TABLE sales_fact ADD (
    CONSTRAINT sales_store_fk   FOREIGN KEY (store_sk)   REFERENCES store_dim,
    CONSTRAINT sales_product_fk FOREIGN KEY (product_sk) REFERENCES product_dim,
    CONSTRAINT sales_date_fk    FOREIGN KEY (date_sk)    REFERENCES date_dim
);
```

Then **reconcile**: the fact table's row count and totals must match the source. If an inner join to a dimension finds no match (a product missing from the product dimension, a NULL date), that row silently disappears from the facts. Compare `COUNT(*)` and `SUM(sales_amount)` of `temp_sales` and `sales_fact`, and use outer joins (with an "unknown" dimension row) where a key may legitimately be missing.

## Common mistakes

- **Loading facts before dimensions**, or dimensions without their business keys, so surrogate keys cannot be looked up.
- **Losing rows in inner joins** during the key lookup, and not reconciling counts and totals.
- **Transforming in the source system**, which slows down the business.
- **No incremental strategy**: reloading everything every night stops scaling as data grows.
- **No log of what failed.** Rejected rows should be written to an error table, not silently dropped.

:::{warning} Check the AI
GenAI assistants can generate an entire ETL script, and the result is easy to accept because it runs. Check the calculations line by line: a classic error is computing a discounted amount as `quantity * price - discount` when the discount is a *rate* (it should be `quantity * price * (1 - discount)`), which inflates revenue on every discounted line without any error message. Then reconcile: the warehouse's row counts and totals must match the source system's, by year and by category. A script that runs is not evidence that it is right.
:::

## Practice

**1. Full or incremental?** (a) The first load of five years of sales history. (b) A nightly load of yesterday's orders. (c) After a bug fix, reloading one month of corrected data. (d) A load of only customers whose address changed since the last run.

:::{note} Answer
:class: dropdown
(a) Full extraction and load. (b) Incremental. (c) A targeted reload of one partition (a full reload of that month, incremental for the warehouse as a whole). (d) Incremental extraction with changed data capture; the dimension changes are applied as slowly changing dimensions.
:::

**2. Classify the transformations.** (a) Combining `first_name` and `last_name` into `customer_name`. (b) Keeping only orders from 2020 onward. (c) Converting `'KS'` to `'Kansas'`. (d) Summing order lines to one row per order. (e) Assigning a surrogate key to each new product.

:::{note} Answer
:class: dropdown
(a) Field-level, many-to-one. (b) Record-level selection. (c) Field-level, one-to-one (decoding). (d) Record-level aggregation. (e) Key generation (a data quality process).
:::

**3. ETL or ELT?** A startup collects website clicks, app events and support tickets in a cloud data lake, and does not yet know which analyses it will need. Which approach fits, and why?

:::{note} Answer
:class: dropdown
ELT: load the raw data cheaply into the lake now, and transform it inside the cloud platform when the analyses are defined. Nothing is lost by transforming too early.
:::

**4. Reconcile.** After loading `sales_fact`, write two queries that check it against `temp_sales`: one comparing row counts, and one comparing total sales amount.

:::{note} One possible answer
:class: dropdown
```sql
SELECT (SELECT COUNT(*) FROM temp_sales) AS staged_rows,
       (SELECT COUNT(*) FROM sales_fact) AS fact_rows
FROM   dual;

SELECT (SELECT SUM(sales_amount) FROM temp_sales) AS staged_total,
       (SELECT SUM(sales_amount) FROM sales_fact) AS fact_total
FROM   dual;
```
Any difference means rows were lost (or duplicated) in the key lookup.
:::

## Key terms

ETL, extract, transform, load, staging area, metadata, full extraction, incremental extraction, changed data capture, online and offline extraction, record-level and field-level transformations, data quality transformations, full load, incremental load, ELT, data lake, surrogate key, identity column, business key, temporary fact table, key lookup, reconciliation

## Further reading

- Kimball, R., & Caserta, J. (2004). *The Data Warehouse ETL Toolkit*. Wiley.
