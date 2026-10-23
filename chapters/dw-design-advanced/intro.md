# Advanced Data Warehouse Design

A basic star schema is enough to start. Real warehouses soon meet harder questions: what key should a dimension use, what level of detail should the fact table keep, which facts can be added up, what to do with yes/no flags and transaction numbers, how to use one dimension in two roles, and how to remember the past when a customer's details change. This chapter covers the techniques that answer them.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Explain why dimensions use surrogate keys, and the arguments on both sides.
2. Choose the granularity of a fact table and explain the trade-off.
3. Classify facts as additive, semi-additive or non-additive.
4. Use factless fact tables, junk dimensions, degenerate dimensions and role-playing dimensions.
5. Handle slowly changing dimensions (types 1 to 4) and rapidly changing dimensions (mini-dimensions).
:::

The examples continue the Prairie Pages bookstore star schema from [Data Warehouse Design](../dw-design/intro.md).

## Surrogate keys

A **surrogate key** is a meaningless integer (1, 2, 3, ...) assigned by the warehouse as a dimension's primary key: `StoreKey`, `BookKey`, `CustomerKey`. The key from the source system, such as a store number or ISBN, is kept as an ordinary attribute, the **business key** (or natural key).

Why not just use the business key?

- **Business keys change, and are reused.** A store is renumbered after a merger; a retired product code is assigned to a new product years later.
- **Surrogate keys allow history.** One store number can have several dimension rows, each describing the store during a different period (see slowly changing dimensions, below).
- **Surrogate keys buffer the warehouse** from changes in the operational systems, and allow data from several sources with clashing keys to be combined.
- **Surrogate keys are short, simple and uniform**: the same integer format for every dimension, which makes fact tables smaller and joins faster.

There are two views in practice. Traditional data warehousing strongly recommends surrogate keys for every dimension, for the reasons above. Many practitioners now consider a surrogate key on every dimension (and on every fact table) to be overkill, and use the source system's primary keys when they are stable and history is not needed. A sensible rule: use surrogate keys for dimensions whose history you must track, or whose business keys are unreliable.

## Granularity of the fact table

The **granularity** (grain) is the level of detail of one fact row. **Higher granularity** means more detail and more rows; lower granularity means more summarized data and fewer rows. For purchase orders, one fact row could be:

- one **line** on a purchase order (finest, **transactional grain**);
- one **purchase order**;
- all purchase orders of a customer in a month (**aggregated grain**).

A finer grain supports more analysis, such as **market basket analysis** (which items are bought together), at the cost of more rows and storage. In web commerce the finest grain may be a single click. Choose the finest grain the sources provide unless storage or performance rules it out; coarser summaries can always be computed from fine facts, never the reverse.

## Additive, semi-additive and non-additive facts

| Type | Can be summed across | Example in a daily sales fact table |
|---|---|---|
| **Additive** | all dimensions | Dollar sales amount: sum across stores, books and days. |
| **Semi-additive** | some dimensions, but not all | Quantity on hand (inventory): sum across stores and books, but **not** across days. |
| **Non-additive** | no dimension | Profit margin (a ratio or percentage). |

Summing a semi-additive fact over time gives nonsense: 40 copies on hand on Monday and 40 on Tuesday is not 80 copies. Over time, use the average or the last value instead. For non-additive facts, store the additive components (profit and sales) and compute the ratio after summing: the margin of a region is total profit divided by total sales, not the average of each store's margin.

## Factless fact tables

A **factless fact table** has no numeric facts at all: only foreign keys. It records that an event happened, or that a combination of dimension members existed. Examples:

- **Promotion coverage**: which books were on promotion in which stores on which days, whether or not they sold. Comparing it with the sales facts answers "which promoted books did *not* sell?"
- **Attendance**: which student attended which class session on which date.

Analysis uses `COUNT(*)` instead of `SUM`.

## Junk dimensions

Transactions often carry several **low-cardinality flags and indicators**: was the purchase online, how was it paid, was a discount applied? Giving each flag its own dimension clutters the schema; putting them in the fact table bloats it. A **junk dimension** instead lists every feasible combination of the flags once, and the fact table carries a single key:

| JunkKey | OnlinePurchase | Payment | Discount |
|---|---|---|---|
| 1 | Yes | Credit card | Yes |
| 2 | Yes | Credit card | No |
| 3 | No | Credit card | Yes |
| 4 | No | Credit card | No |
| 5 | No | Cash | Yes |
| 6 | No | Cash | No |

(Online purchases cannot be paid in cash, so that combination never occurs and is not listed.)

## Degenerate dimensions

The receipt number in the Prairie Pages fact table is a **degenerate dimension (DD)**: a dimension key stored in the fact table with no dimension table of its own, because everything else about the receipt (date, store) is already in other dimensions. It groups all the line items of one transaction, which is what makes market basket analysis possible: "find receipts that contain both a cookbook and a gardening book". Order numbers, invoice numbers and ticket numbers are typical degenerate dimensions.

## Role-playing dimensions

A **role-playing dimension** is one dimension table referenced several times by the same fact table, each time in a different role. Online orders have an **order date** and a **ship date**: both are dates, so both refer to the same Date dimension, through two foreign keys:


:::{figure} images/role_playing.png
:label: fig-role-playing
:alt: DATE_DIM connected twice to ORDER_FACT, once labeled order date to OrderDateKey and once labeled ship date to ShipDateKey.
A role-playing date dimension: one table, two roles. Source: {download}`role_playing.drawio <images/role_playing.drawio>`.
:::

In SQL, join the dimension twice with different aliases (often through two views, `ORDER_DATE` and `SHIP_DATE`, so users see two clearly named dimensions):

```sql
SELECT od.month_name AS order_month,
       AVG(sd.full_date - od.full_date) AS avg_days_to_ship
FROM   order_fact f
JOIN   date_dim od ON f.order_date_key = od.date_key
JOIN   date_dim sd ON f.ship_date_key  = sd.date_key
GROUP  BY od.month_name;
```

## Slowly changing dimensions

Dimension attributes change slowly and irregularly over time: a customer moves, a book changes publisher, a store is remodeled. A **slowly changing dimension (SCD)** technique decides whether and how the warehouse remembers the old values. Consider a Prairie Pages loyalty member whose segment is reassessed every year, from **Silver** to **Gold** on 15 March 2026.

**Type 1: overwrite.** Replace the old value. Simple, but history is lost: all past sales now appear as Gold sales.

| CustomerKey | CustomerNo | Name | Segment |
|---|---|---|---|
| 1001 | C-48213 | Riley Chen | Gold (Silver is overwritten) |

**Type 2: add a new row.** Create a new dimension row (with a new surrogate key) each time the member changes, with effective dates and a current flag. Old sales stay linked to the Silver row; new sales link to the Gold row. This is the most common approach, because it keeps history accurately.

| CustomerKey | CustomerNo | Name | Segment | StartDate | EndDate | Current |
|---|---|---|---|---|---|---|
| 1001 | C-48213 | Riley Chen | Silver | 2024-06-02 | 2026-03-14 | N |
| 1874 | C-48213 | Riley Chen | Gold | 2026-03-15 | 9999-12-31 | Y |

**Type 3: add a column.** Keep the previous value in an extra column. Only a limited history (usually one previous value) is kept, but it makes "before and after" comparisons easy.

| CustomerKey | CustomerNo | Name | PreviousSegment | CurrentSegment |
|---|---|---|---|---|
| 1001 | C-48213 | Riley Chen | Silver | Gold |

**Type 4: history table.** Keep only the current row in the dimension, and move the full history to a separate history table.

| CustomerKey | CustomerNo | Name | Segment |
|---|---|---|---|
| 1001 | C-48213 | Riley Chen | Gold |

| CustomerNo | Segment | StartDate | EndDate |
|---|---|---|---|
| C-48213 | Silver | 2024-06-02 | 2026-03-14 |
| C-48213 | Gold | 2026-03-15 | 9999-12-31 |

The end date `9999-12-31` is a common convention for "still current", which keeps date-range queries simple. Oracle's SH schema is ready for Type 2: `SH.PRODUCTS` and `SH.CUSTOMERS` have effective-from, effective-to and valid-flag columns (`PROD_EFF_FROM`, `PROD_EFF_TO`, `PROD_VALID`), although the sample data contains only one version of each row.

## Rapidly changing dimensions

Some attributes change often and regularly: a customer segment recalculated **daily**, a credit score, an age band. Type 2 would then add a new customer row every day, and the dimension would explode. The solution is to split the dimension:

- keep the **stable** attributes (name, gender, date of birth, address) in the main Customer dimension;
- move the **rapidly changing** attributes into a small **mini-dimension**, such as `CUSTOMER_PROFILE` with one row per combination of banded values (segment, income band, credit band);
- give the fact table a foreign key to both, so each sale records the customer's profile *at the time of the sale*.

To answer "what is this customer's current profile?" without scanning facts, the validity can also be recorded explicitly, with a table that links each customer to profile rows with start and end dates.

## Common mistakes

- **Using business keys everywhere, then needing history.** Retrofitting surrogate keys is painful.
- **Summing semi-additive facts over time**, such as adding up month-end balances for a year.
- **Averaging ratios.** Store the components and compute the ratio after aggregating.
- **A dimension table for a receipt number.** It would have one row per receipt and no attributes: keep it as a degenerate dimension.
- **Type 1 by default.** Overwriting is easy, but it silently rewrites history in every past report.

:::{warning} Check the AI
These techniques are where GenAI answers most often go subtly wrong. Typical errors: calling an account balance "additive" (it is semi-additive across time); proposing Type 2 for an attribute that changes daily (a mini-dimension is needed); or describing Type 3 and Type 4 the wrong way round (the numbering of types beyond 3 varies between sources, so check which definition your team uses). Ask the assistant to show a few example rows before and after a change, and check that a report on last year's sales would still give last year's answer.
:::

## Practice

**1. Additivity.** Classify each fact: (a) number of hotel nights sold; (b) account balance at month end; (c) average room rate; (d) number of employees on payroll at month end; (e) revenue.

:::{note} Answer
:class: dropdown
(a) Additive. (b) Semi-additive (not across time). (c) Non-additive (store revenue and nights, and divide). (d) Semi-additive (not across time). (e) Additive.
:::

**2. Which technique?** (a) A warehouse must report sales by each customer's state *at the time of the sale*, even after customers move. (b) Analysts only care about each customer's current email address. (c) A retailer wants to compare each store's sales under its current and its previous regional manager. (d) A bank recomputes each customer's risk band every night.

:::{note} Answer
:class: dropdown
(a) SCD Type 2. (b) Type 1. (c) Type 3 (current and previous manager columns). (d) A mini-dimension for the risk band (a rapidly changing dimension).
:::

**3. Dimension design.** An airline's ticket sales have: the ticket number, the booking date, the flight date, the fare class, whether the ticket is refundable, whether seat selection was purchased, and the fare amount. Identify a degenerate dimension, a role-playing dimension, a junk dimension and the facts.

:::{note} One possible answer
:class: dropdown
Degenerate dimension: the ticket number. Role-playing dimension: Date, used as booking date and flight date. Junk dimension: the combinations of refundable and seat-selection flags (fare class could join them, or be its own dimension). Fact: fare amount (and a ticket count of 1).
:::

**4. A Type 2 change.** Using the Riley Chen Type 2 table, write the rows after a third change, to **Platinum** on 1 April 2027. Which rows change, and which are added?

:::{note} Answer
:class: dropdown
The Gold row (key 1874) is updated: EndDate becomes 2027-03-31 and Current becomes N. A new row is added, for example key 2390, C-48213, Riley Chen, Platinum, 2027-04-01, 9999-12-31, Y. The Silver row is unchanged.
:::

## Key terms

surrogate key, business (natural) key, granularity, transactional grain, aggregated grain, market basket analysis, additive, semi-additive and non-additive facts, factless fact table, junk dimension, degenerate dimension, role-playing dimension, slowly changing dimension (types 1, 2, 3 and 4), effective date, current flag, rapidly changing dimension, mini-dimension

## Further reading

- Kimball, R., & Ross, M. (2013). *The Data Warehouse Toolkit* (3rd ed.), chapter 5 on slowly changing dimension techniques.
- Kimball Group, [Dimensional Modeling Techniques](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/).
