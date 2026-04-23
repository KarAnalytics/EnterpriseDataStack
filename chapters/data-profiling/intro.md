# Data Profiling

**Data profiling** is the discipline of examining a dataset to produce informative summaries of what it actually contains — *before* you trust it for any business decision. It's the diagnostic step that tells you what needs processing, whether a source is fit for purpose, and where hidden quality problems are waiting to bite downstream systems.

## What a profile reveals

A good profile answers these questions for every column:

- **Completeness** — how many rows are missing, null, or blank?
- **Distinct count** — how many unique values does this column take?
- **Value distribution** — what are the most common values, and how long is the tail?
- **Range and statistics** — for numeric columns: min, max, mean, median, standard deviation, percentiles
- **Pattern** — for string columns: common patterns (e.g., `999-99-9999` for SSNs), length distribution
- **Type conformance** — do the values actually match the declared type? (A `VARCHAR` column full of numbers is a smell)
- **Constraint conformance** — do values satisfy business rules (age > 0, email matches a regex, state in a known set)?
- **Referential integrity** — do foreign keys actually resolve to rows in the parent table?

## Levels of profiling

| Level | What it looks at | Example finding |
|---|---|---|
| **Column profiling** | One column at a time | `email` has 12% nulls and 3% invalid formats |
| **Cross-column profiling** | Pairs of columns in the same table | `zip_code` and `state` disagree in 1.4% of rows |
| **Cross-table profiling** | Relationships between tables | `orders.customer_id` has 42 values with no match in `customers` |

## Tools

- **SQL itself** — aggregate queries (`COUNT`, `COUNT DISTINCT`, `MIN`, `MAX`, `AVG`, `STDDEV`) are the most portable profiler
- **Python** — `pandas.DataFrame.describe()`, `pandas-profiling` / `ydata-profiling`, `great-expectations`
- **OpenRefine** — faceting is profiling with a GUI
- **Commercial** — Informatica Data Quality, Talend, Collibra, AWS Glue DataBrew, Oracle Enterprise Data Quality
- **Cloud** — BigQuery's `INFORMATION_SCHEMA`, Snowflake's data profiling functions

## A minimal SQL profile

Run this on every new source you touch:

```sql
SELECT
    COUNT(*)                                    AS total_rows,
    COUNT(email)                                AS non_null_emails,
    COUNT(DISTINCT email)                       AS distinct_emails,
    COUNT(*) FILTER (WHERE email IS NULL)       AS null_emails,
    AVG(total_amount)                           AS avg_amount,
    MIN(order_date)                             AS earliest_order,
    MAX(order_date)                             AS latest_order
FROM orders;
```

## From profile to action

A profile is not the deliverable — a *decision* is. Typical decisions after profiling:

- Reject a source as unfit for purpose
- Accept it but constrain how it can be used (e.g., exclude a bad column)
- Add processing rules to fix the issues (Chapter 6)
- Add validation rules to future loads so the same issues are caught on ingestion
- Escalate to the data steward / governance team (Chapter 9)

## Learning outcomes

- Define data profiling and contrast it with processing
- Produce a column-level profile in SQL or with a profiling library
- Identify cross-column and cross-table issues
- Translate profiling findings into concrete remediation or validation rules

