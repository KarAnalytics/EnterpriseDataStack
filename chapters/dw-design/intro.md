# Data Warehouse Design

A data warehouse is designed for one thing above all: answering analytical questions quickly and simply. The design that does this best is the **star schema**, also called the **dimensional model**: a central table of measurements surrounded by tables that describe them. This chapter introduces facts, dimensions and grain, the star, snowflake and constellation schemas, and an eight-step design process, applied to a retail case and to Oracle's SH sample schema.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Explain fact tables, dimension tables and the grain of a fact table.
2. Distinguish star, snowflake and fact constellation schemas, and know when each fits.
3. Follow the steps of data warehouse design for a business process.
4. Recognize the characteristics of good facts and good dimensions.
5. Write a star-join query on a dimensional model.
:::

## The star schema

A **star schema** has two kinds of tables:

- The **fact table** holds the measurements, metrics or **facts** of a business process: one row per transaction or event (a sale, a shipment, a click), with its numeric measures (quantity, dollars) and a foreign key to each dimension.
- The **dimension tables** hold further information about each fact: when, where, what, who, how (date, store, product, customer). They provide the criteria for grouping and filtering the measures, and they are usually **denormalized**: one wide table per dimension, even if it repeats values.

The **grain** of the fact table is the finest level of detail it records: the meaning of one row. Declaring the grain is the most important design decision, because everything else follows from it.

Drawn on paper, the fact table sits in the middle and the dimensions around it, which gives the schema its name.

:::{figure} images/bookstore_star.png
:label: fig-star
:alt: A star schema with SALES_FACT in the center, connected to four dimension tables: DATE_DIM, STORE_DIM, BOOK_DIM and PROMOTION_DIM. The fact table lists foreign keys to each dimension, a receipt number marked DD, and the measures Quantity, SalesAmount, DiscountAmount and CostAmount.
A star schema for a bookstore chain's sales. Each dimension relates to many fact rows. DD marks a degenerate dimension (see [Advanced Data Warehouse Design](../dw-design-advanced/intro.md)). Source: {download}`bookstore_star.drawio <images/bookstore_star.drawio>`.
:::

### Snowflake schema

A **snowflake schema** normalizes one or more dimensions further, splitting a hierarchy into separate tables: books into subjects, subjects into genres. It is useful when a dimension has a large, well-defined hierarchy, and it removes some redundancy. The cost is more joins in every query. Practical tip: avoid excessive snowflaking, because it slows queries down and makes the schema harder for business users to understand.

:::{figure} images/bookstore_snowflake.png
:label: fig-snowflake
:alt: A snowflake schema. SALES_FACT connects to DATE_DIM, STORE_DIM and BOOK_DIM. BOOK_DIM connects to SUBJECT_DIM, which connects to GENRE_DIM.
The same sales facts with the book dimension snowflaked into subject and genre tables. Source: {download}`bookstore_snowflake.drawio <images/bookstore_snowflake.drawio>`.
:::

### Fact constellation

A **fact constellation** (or galaxy schema) has several fact tables that share dimension tables. It is useful when there are several clusters of facts, often at different grains, such as sales transactions and daily inventory levels. Real warehouses are usually hybrids: constellations with some snowflaked dimensions. Shared dimensions must mean the same thing for every fact table that uses them; these are the **conformed dimensions** of [Data Warehousing Concepts](../dw-concepts/intro.md).

:::{figure} images/bookstore_constellation.png
:label: fig-constellation
:alt: A fact constellation. SALES_FACT on the left and INVENTORY_FACT on the right both connect to the shared DATE_DIM, STORE_DIM and BOOK_DIM tables in the middle.
Sales and inventory facts sharing conformed dimensions. Source: {download}`bookstore_constellation.drawio <images/bookstore_constellation.drawio>`.
:::

## Steps in data warehouse design

1. **Select the business process** to model (sales, orders, inventory, claims, ...).
2. **Determine the data sources** that record it.
3. **Choose a suitable schema**: star, snowflake or constellation.
4. **List the data processing needs** (cleaning, standardizing, integrating; see [Part II](../data-processing/intro.md)).
5. **Identify the numeric facts** that measure the process.
6. **Declare the grain**: what one fact row means.
7. **Choose the dimensions** that apply to each fact row.
8. **Define the tables and constraints**: primary keys on the dimensions, and foreign keys from the fact table to each dimension.

Steps 5 to 7 are often done in the order grain, dimensions, facts: once the grain is declared, the dimensions and facts that are true at that grain follow naturally.

## Worked example: a bookstore chain

Prairie Pages runs 35 bookstores in four Midwestern states, with about 120,000 titles in its catalog. Management wants to analyze sales patterns: which titles, subjects and formats sell where and when, and how promotions affect them.

1. **Business process**: retail sales at the point of sale (the checkout).
2. **Data sources**: the point-of-sale system in each store, the product catalog, the store directory, and the marketing team's promotion calendar.
3. **Schema**: a simple star schema.
4. **Processing needs**: standardize author and publisher names across sources; give every receipt line a store, date and title; map each sale to the promotion active at the time.
5. **Grain**: one row per **line item on a receipt**, that is, one title on one sales transaction. This is the finest grain available, and the most flexible: any summary (by day, store, title or promotion) can be computed from it.
6. **Dimensions** true at that grain: **Date** (when), **Store** (where), **Book** (what), **Promotion** (under what conditions). The receipt number is kept in the fact table to group the lines of one purchase.
7. **Facts**: Quantity, SalesAmount, DiscountAmount and CostAmount (from which gross profit can be computed).
8. **Tables and constraints**: each dimension has a primary key; the fact table has a foreign key to each dimension, as in {numref}`fig-star`.

### The fact table

The fact table stores the performance measurements of the business. The facts are the business measures; the dimension keys define the grain. Good facts are:

- **numeric and additive**, so they can be summed across rows (more on additivity in [Advanced Data Warehouse Design](../dw-design-advanced/intro.md));
- **continuously valued**, like dollars and quantities, rather than a small set of codes;
- recorded only for **events that happened**: a fact table has no rows of zeros for books that did not sell;
- linked by **foreign keys** with referential integrity to every dimension.

Fact tables are **tall and thin**: few columns, but by far the most rows in the warehouse. Prairie Pages, selling a few thousand books per store per day, adds tens of millions of rows a year.

### The dimension tables

Dimension tables are **short and fat**: relatively few rows, many columns. Their attributes are:

- usually **textual**, verbose, descriptive and complete ("Hardcover", not "HC");
- **discretely valued**, with minimal use of codes;
- **denormalized**, capturing hierarchies in one table (book, subject, genre);
- the main source of **query constraints, groupings and report labels**: every `WHERE`, `GROUP BY` and column heading in a report comes from a dimension.

### The date dimension

A **date dimension** appears in almost every data warehouse. It has one row per day (365 rows per year) and many attributes that SQL date functions cannot easily provide: day of week, fiscal period, holiday flags, season, pay-day indicators. Analysts can then group or filter by any of them without calculating them in every query.

| DateKey | FullDate | DayOfWeek | MonthName | Quarter | Year | HolidayFlag |
|---|---|---|---|---|---|---|
| 20261124 | 2026-11-24 | Tuesday | November | Q4 | 2026 | N |
| 20261126 | 2026-11-26 | Thursday | November | Q4 | 2026 | Y |
| 20261127 | 2026-11-27 | Friday | November | Q4 | 2026 | N |

### Facts versus dimensions

| Facts | Dimensions |
|---|---|
| arise from business events | describe the business events |
| quantitative and continuous | descriptive and discrete |
| are summed, averaged, counted | are used to filter, group and label |

A quick test: if you would put it in a `SUM()`, it is a fact; if you would put it in a `GROUP BY`, it belongs in a dimension.

## A real star schema: Oracle SH

The SH (Sales History) schema in every Oracle Autonomous Database is a star schema ([The SH Sample Schema](../appendix/sh-schema.md)). `SH.SALES` is the fact table, with 918,843 rows at the grain of one product sold to one customer on one day through one channel under one promotion; its facts are `QUANTITY_SOLD` and `AMOUNT_SOLD`. The dimensions are `CUSTOMERS`, `PRODUCTS`, `TIMES`, `CHANNELS` and `PROMOTIONS` (and `COUNTRIES`, which snowflakes the customer geography). `SH.TIMES` is a classic date dimension: 1,826 rows, one per day from 1998 to 2002, with 38 calendar and fiscal attributes.

A **star join** joins the fact table to the dimensions it needs and groups by dimension attributes:

```sql
SELECT ch.channel_desc,
       t.calendar_year,
       SUM(s.amount_sold) AS sales
FROM   sh.sales s
JOIN   sh.channels ch ON s.channel_id = ch.channel_id
JOIN   sh.times t     ON s.time_id = t.time_id
GROUP  BY ch.channel_desc, t.calendar_year
ORDER  BY t.calendar_year, sales DESC;
```


:::{figure} images/sh_sales_by_channel.png
:label: fig-sh-sales
:alt: A stacked bar chart of SH sales by channel for 1998 to 2001. Direct Sales is the largest channel each year; Internet sales grow sharply in 2001; Tele Sales appear only in 1998.
The star join, charted: SH sales by channel and year, in millions of dollars.
:::

For 2001 the result shows Direct Sales at about $13.39 million, Partners at $8.04 million and Internet at $6.71 million. Tele Sales appear only in 1998, and the Catalog channel has no sales at all, a reminder that a dimension can contain members that never occur in the facts.

## Common mistakes

- **Not declaring the grain**, or mixing grains in one fact table (some rows per receipt line, others per receipt).
- **Putting descriptive text in the fact table.** Book titles belong in the Book dimension.
- **Over-normalizing dimensions** out of OLTP habit. A star schema is meant to be denormalized.
- **Summarizing too early.** A fact table at the daily-store level can never answer a question about individual receipts.
- **Codes instead of descriptions** in dimensions, which force every report writer to decode them.

:::{warning} Check the AI
Ask a GenAI assistant for "a star schema for a bookstore" and it will produce a reasonable diagram in seconds. Check three things before using it. First, is the grain stated, and is every fact true at that grain? Assistants often mix a receipt-level total into a line-item fact table, which double counts when summed. Second, are the dimensions denormalized, or has the assistant produced a normalized OLTP design with "dimension" in the table names? Third, is there a date dimension, rather than a bare date column in the fact table? The design must also fit your sources: an assistant will happily include a Customer dimension for a store that never records who bought what.
:::

## Practice

**1. Facts or dimensions?** For a car rental company, classify: rental days, pickup location, car model, total charge, customer's loyalty tier, mileage driven, rental date.

:::{note} Answer
:class: dropdown
Facts: rental days, total charge, mileage driven. Dimension attributes: pickup location (Location dimension), car model (Vehicle dimension), loyalty tier (Customer dimension), rental date (Date dimension).
:::

**2. Grain.** A coffee chain wants to analyze sales by drink, store and hour of the day, and to study which drinks are bought together. Which grain should its sales fact table have, and why not "one row per store per day"?

:::{note} Answer
:class: dropdown
One row per line item on a sales transaction (one drink on one order), with the order number kept as a degenerate dimension. A store-day grain cannot show the hour of the day or which drinks are on the same order; the line-item grain can be summarized to any coarser level.
:::

**3. Design a star.** A university library wants to analyze book checkouts by title, branch, patron type (undergraduate, graduate, faculty) and date. Follow the eight design steps and draw the star schema in draw.io.

:::{note} One possible answer
:class: dropdown
Process: checkouts. Grain: one checkout of one item. Dimensions: Date, Title (title, author, subject, format), Branch (name, campus), Patron (patron type, department; no names, for privacy). Facts: checkout count (always 1, useful for summing) and loan days (filled when the item is returned). CHECKOUT_FACT has foreign keys to the four dimensions.
:::

**4. Star join.** In your Oracle Cloud account, write a star-join query on SH that shows total `AMOUNT_SOLD` by product category for calendar year 2001, highest first.

:::{note} Answer
:class: dropdown
```sql
SELECT p.prod_category, SUM(s.amount_sold) AS sales
FROM   sh.sales s
JOIN   sh.products p ON s.prod_id = p.prod_id
JOIN   sh.times t    ON s.time_id = t.time_id
WHERE  t.calendar_year = 2001
GROUP  BY p.prod_category
ORDER  BY sales DESC;
```
Peripherals and Accessories is first, at about $7.86 million, followed by Photo ($6.33 million).
:::

## Key terms

star schema, dimensional model, fact table, fact, measure, dimension table, grain, denormalized, snowflake schema, fact constellation (galaxy schema), conformed dimension, date dimension, star join

## Further reading

- Kimball, R., & Ross, M. (2013). *The Data Warehouse Toolkit* (3rd ed.), chapters 2 and 3.
- Kimball Group, [Dimensional Modeling Techniques](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/).
