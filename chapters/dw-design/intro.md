# Data Warehouse Design

Designing a data warehouse means designing for **query speed, ease of understanding, and historical accuracy** — priorities that deliberately break the 3NF rules you followed for OLTP. The dominant design pattern is the **star schema**, pioneered by Ralph Kimball.

## Star schema

A star schema has two kinds of tables:

- **Fact table** — the central table holding the business events or measurements. Every row is a *measurement* (a sale, a web click, a temperature reading). Columns are **foreign keys to dimensions** plus numeric **measures**.
- **Dimension tables** — the context around each fact (who, what, where, when, how). Columns are descriptive attributes used for filtering, grouping, and labeling.

Visually:

```
            ┌────────────┐
            │  dim_date  │
            └─────┬──────┘
                  │
┌────────────┐   ┌────────────┐   ┌─────────────┐
│ dim_product├───┤ fact_sales ├───┤ dim_customer│
└────────────┘   └─────┬──────┘   └─────────────┘
                       │
                 ┌─────┴──────┐
                 │  dim_store │
                 └────────────┘
```

The star comes from drawing the fact table in the middle with dimensions radiating out.

## Designing the fact table

Kimball's four-step process:

1. **Pick the business process** (e.g., "retail sales")
2. **Declare the grain** — the meaning of one row ("one product line on one sales transaction at one store on one date")
3. **Identify the dimensions** — what context do we want to analyze by?
4. **Identify the facts** — what numeric measures are at the declared grain?

Grain is the single most important decision. Every measure in the fact table must be true at the stated grain. Mixing grains in one fact table causes double-counting and wrong answers.

## Types of measures

- **Additive** — can be summed across any dimension (quantity, revenue). Most common.
- **Semi-additive** — sum across some dimensions but not others (balances sum across customers but not across time — use average or end-of-period instead)
- **Non-additive** — can't be summed at all (ratios, percentages — store the numerator and denominator and compute the ratio in the BI tool)

## Types of fact tables

- **Transaction fact** — one row per event (one sale, one click). Most common.
- **Periodic snapshot fact** — one row per entity per period (end-of-month inventory per product per store)
- **Accumulating snapshot fact** — one row per entity tracking a pipeline (order received → picked → shipped → delivered); the row is updated as milestones complete

## Designing dimension tables

Dimensions are **wide, denormalized, and richly attributed**. A `dim_product` table might have 40 columns — product name, brand, category, sub-category, supplier, color, size, weight, launch_date, status. That denormalization is deliberate: it keeps joins to a single level.

Dimension keys are always **surrogate keys** (auto-generated integers), never natural keys. Why:

- Natural keys change (products get reclassified, customers get new email addresses)
- Surrogate keys give you a stable hook for slowly changing dimensions
- Joins on 4-byte integers are faster than joins on strings

## Slowly Changing Dimensions (SCDs)

When a dimension attribute changes (a customer moves cities), you need to decide how to handle history. The standard types:

| Type | Policy | Trade-off |
|---|---|---|
| **Type 0** | Keep original, ignore changes | Simplest; loses history |
| **Type 1** | Overwrite in place | Loses history; easy |
| **Type 2** | Add a new row with `effective_from` / `effective_to` / `is_current` | Full history; fact rows link to the correct historical version |
| **Type 3** | Add a "previous value" column | Keeps one prior value only |
| **Type 4** | Separate "history" table | Keeps main table slim |
| **Type 6 (1+2+3)** | Hybrid | Used when you need multiple views of history |

Type 2 is by far the most common.

## Conformed dimensions

When two fact tables (say `fact_sales` and `fact_returns`) share a `dim_product`, that dimension is **conformed** — it means the same thing in both contexts. Conformed dimensions are the glue that lets you combine facts across business processes ("sales minus returns = net revenue").

## Naming conventions

Cosmetic but important for usability:

- Tables: `fact_`, `dim_`, `bridge_` prefixes
- Surrogate keys: `<table>_sk`
- Natural keys: `<table>_nk` or `<table>_id_src`
- Date columns in facts: `<role>_date_sk` (e.g., `order_date_sk`, `ship_date_sk`)

## Learning outcomes

- Apply Kimball's four-step fact table design process
- Declare the grain of a fact table and check that measures are consistent with it
- Distinguish additive, semi-additive, and non-additive measures
- Choose a fact table type (transaction / periodic / accumulating) for a given process
- Implement SCD Type 1 and Type 2 dimensions
- Explain what makes a dimension "conformed"

