# Advanced Data Warehouse Design

With star schemas under your belt, this chapter covers the patterns you'll need once real-world complexity shows up: snowflake schemas, galaxies, bridge tables, role-playing dimensions, junk and degenerate dimensions, and aggregate navigators.

## Snowflake schema

A **snowflake schema** normalizes dimensions into multiple tables — e.g., splitting `dim_product` into `dim_product`, `dim_subcategory`, `dim_category`, `dim_department`. The result looks like a snowflake rather than a star.

Pros:
- Saves storage on repeated attributes (though this matters much less in columnar warehouses)
- Cleaner enforcement of hierarchies

Cons:
- More joins per query
- Harder for business users to understand
- BI tools often work less smoothly

**Rule of thumb:** start with star; snowflake only when a specific dimension is genuinely huge and slowly changing.

## Galaxy (fact constellation) schema

Most real warehouses have many fact tables sharing conformed dimensions — `fact_sales`, `fact_inventory`, `fact_shipments` all pointing at `dim_product`, `dim_date`, `dim_store`. This arrangement is called a **galaxy schema** or **fact constellation**. It's the natural endpoint of adding business processes incrementally.

## Role-playing dimensions

A single dimension can play multiple roles in the same fact table. The classic case is `dim_date`:

```
fact_orders
  ├─ order_date_sk     →  dim_date (viewed as dim_order_date)
  ├─ ship_date_sk      →  dim_date (viewed as dim_ship_date)
  └─ delivery_date_sk  →  dim_date (viewed as dim_delivery_date)
```

Rather than duplicate the table, expose **views** over `dim_date` with renamed columns (`order_date`, `ship_date`, `delivery_date`) so BI tools can present them as separate dimensions.

## Degenerate dimensions

A **degenerate dimension** is a "dimension" that lives entirely in the fact table because it has no descriptive attributes — typically a transaction or document identifier (invoice number, order number, ticket id). Keep it on the fact row for grouping (e.g., line items in the same invoice) but don't build a separate dimension table for it.

## Junk dimensions

Low-cardinality flags scattered across the fact table (is_gift, shipping_method_type, payment_flag) can bloat the schema. Combine them into a single **junk dimension** that holds every realistic combination:

```
dim_order_flags_sk | is_gift | payment_method | shipping_type | channel
------------------ | ------- | -------------- | ------------- | -------
                 1 |    Y    |      card      |     ground    |  web
                 2 |    N    |      card      |     ground    |  web
                 3 |    N    |      cash      |     pickup    | store
                ...
```

One `dim_order_flags_sk` on the fact row replaces several flag columns.

## Bridge tables — multi-valued dimensions

Some relationships are genuinely many-to-many between a fact and a dimension — a hospital visit with multiple diagnoses, a sale attributed to multiple salespeople. Model with a **bridge table**:

```
fact_visit → visit_diagnosis_group_sk
             bridge_diagnosis_group (group_sk, diagnosis_sk, weight)
             dim_diagnosis
```

The **weight factor** lets you allocate fact measures across members (e.g., 0.4 of the revenue attributed to salesperson A, 0.6 to salesperson B) — but use it only when the business genuinely wants allocated totals.

## Mini-dimensions

When a dimension has a handful of rapidly changing attributes (customer demographics, say), splitting them into a **mini-dimension** keeps the main dimension stable and Type 2 history manageable:

```
dim_customer (demographics_sk →)  dim_customer_demographics
```

## Aggregate fact tables

For very large fact tables, pre-aggregated summary tables (`fact_sales_monthly_by_region`) can dramatically speed up common queries. Modern MPP warehouses (BigQuery, Snowflake, Redshift) often compute these on the fly using materialized views or result caching, but traditional warehouses explicitly build and maintain an **aggregate navigator**.

## Factless fact tables

A **factless fact table** records the *occurrence* of events with no numeric measures — student attendance, product-promotion eligibility, website page views. The "fact" is the existence of the row; you aggregate with `COUNT(*)`.

## Performance design

Design choices that often matter more than schema shape:

- **Partitioning** — range-partition fact tables by date so queries scan only recent partitions
- **Clustering / sort keys** — physically order rows by common filter columns (customer, product)
- **Columnar storage** — reads only the columns the query uses (all modern cloud warehouses)
- **Result caching** — repeated identical queries return instantly
- **Workload management** — separate clusters/warehouses for ETL vs. BI vs. ad-hoc

## Learning outcomes

- Explain when to snowflake a dimension and when to leave it flat
- Model role-playing, degenerate, junk, and mini-dimensions correctly
- Use a bridge table for a multi-valued relationship
- Decide whether a factless fact table fits a requirement
- Choose partitioning and clustering strategies for a large fact table

