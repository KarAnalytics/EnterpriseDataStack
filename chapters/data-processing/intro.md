# Data Processing

Real data is never clean. Spreadsheets arrive with inconsistent casing, dates in five formats, trailing whitespace, merged cells, and columns that mean different things on different rows. **Data processing** is the set of techniques that reshape messy inputs into a form a database — or a downstream ML model — can actually use.

## Where processing sits in the EDM pipeline

Back in Chapter 2 we placed data wrangling between integration and storage. Processing is the "do the work" half of wrangling; profiling (Chapter 8) is the "figure out what's wrong" half.

```
Sources → Ingestion → Integration → PROFILING → PROCESSING → Storage → Warehousing
```

You almost always profile first, process second, and profile again to confirm the fix.

## Core processing operations

- **Standardization** — consistent casing, trimmed whitespace, unified date formats, canonical addresses and phone numbers
- **Parsing and splitting** — breaking `"Doe, Jane"` into `last_name` and `first_name`; splitting a full address into street / city / state / zip
- **Type coercion** — `"1,234.50"` → `1234.50`; `"2025-01-03"` → `DATE`
- **Deduplication** — exact duplicates (same row twice) vs. fuzzy duplicates (same customer, different spellings)
- **Imputation** — replacing missing values with a default, a calculated value (mean/median/mode), a forward-fill, or a model prediction
- **Outlier handling** — detect with statistical rules (IQR, z-score) or business rules; cap, drop, or flag
- **Normalization and scaling** — min-max scaling, z-score standardization (mostly for ML inputs)
- **Encoding** — turning categorical labels into codes (`'M'/'F'` → `1/0`; one-hot vectors)
- **Joining reference data** — enriching rows with lookups (country code → country name, product id → category)

## Batch vs. stream

Most enterprise processing is still **batch**: a job runs nightly, processes a day of data, writes to the warehouse. But increasingly data is processed as it arrives (**stream**) using Kafka, Kinesis, Pub/Sub, or Spark Streaming. The operations above apply in both — only the cadence changes.

## Tools you'll see in this course

- **SQL** — for anything that fits in a table, SQL is usually the fastest path (`REPLACE`, `TRIM`, `CAST`, `COALESCE`, `CASE WHEN`)
- **Python + pandas** — more flexible for ad-hoc transforms
- **OpenRefine** (Chapter 7) — GUI-driven, excellent for interactive exploration and clustering of near-duplicates
- **Spark** (Chapter 21) — for data too big for one machine

## Data quality dimensions

A clean dataset satisfies all six dimensions:

| Dimension | Question it answers |
|---|---|
| Accuracy | Do values match reality? |
| Completeness | Are required fields filled? |
| Consistency | Do the same facts agree across tables? |
| Timeliness | Is the data fresh enough to act on? |
| Validity | Do values conform to format / domain rules? |
| Uniqueness | Is each real-world entity represented once? |

Most processing rules exist to move the data up the scale on one of these dimensions.

## Learning outcomes

- Identify common data quality issues in a raw dataset
- Choose the right operation (standardize / parse / impute / dedupe / encode) for a given problem
- Explain the six data-quality dimensions and give an example of a failure in each
- Decide when to use SQL, pandas, or OpenRefine for a processing task

