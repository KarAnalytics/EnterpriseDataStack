# ETL — DB to DW Procedure

The warehouse schemas in the last three chapters don't populate themselves. The process that moves data from source systems into the warehouse, shaped and cleaned along the way, is **ETL** — Extract, Transform, Load. This chapter walks through the procedure end-to-end before we apply it to a concrete example (Northwind) in Chapter 17.

## E, T, L — and ELT

- **Extract** — pull data from sources (OLTP databases, files, APIs, logs). Full loads, incremental loads, or change-data-capture (CDC) depending on volume.
- **Transform** — cleanse, standardize, enrich, conform, and reshape data into the warehouse's star-schema form.
- **Load** — insert into the warehouse — typically into a staging area first, then into the production star schema.

**ELT** reorders the last two: load raw data into the warehouse first, then transform inside the warehouse using SQL. ELT has become the modern default because cloud warehouses are fast enough to handle transforms cheaply and it preserves the raw data for re-processing. Tools like **dbt** are built around this pattern.

## Reference architecture

```
Source systems  →  Staging  →  Transform  →  Warehouse (star schema)
   (OLTP)          (landing)    (scrub,       (fact + dim tables)
                                 conform,
                                 SCD apply)
                                                   ↓
                                             BI / reporting
```

## Extraction patterns

- **Full load** — re-extract the entire source table every run. Simple; works for small reference tables.
- **Incremental by watermark** — pull rows where `updated_at > last_run_watermark`. Fast; requires a reliable timestamp.
- **Change Data Capture (CDC)** — stream inserts/updates/deletes from the database's transaction log (Debezium, Oracle GoldenGate, AWS DMS). Lowest latency; most operationally complex.
- **File drop** — upstream system writes files to a shared location (SFTP, S3, GCS); ETL picks them up on a schedule.

## Transform step — what actually happens

The transform layer is where most of the logic lives. Typical operations, in roughly the order they run:

1. **Cleansing** — trim whitespace, standardize casing, fix encodings, parse dates
2. **Data-type coercion** — `"1,234.50"` → `DECIMAL(10,2)`
3. **Validation** — reject or quarantine rows that violate business rules
4. **Deduplication** — by natural key (last-wins, first-wins, or manual rules)
5. **Enrichment / lookup** — join to reference data to add categories, regions, geocoordinates
6. **Conforming** — map source codes to warehouse codes (e.g., `"M"` and `"Male"` both → `"M"`)
7. **Surrogate key lookup / generation** — map natural keys to warehouse surrogate keys; generate new surrogates for unseen values
8. **Slowly changing dimension logic** — detect attribute changes and apply Type 1 / 2 / 3 rules
9. **Fact build** — join staging facts to dimension surrogate keys; compute derived measures

## Load step

- Use **set-based SQL** (bulk `INSERT`, `MERGE`, `COPY`) — never row-by-row
- **Staging-then-swap** — load into a staging partition, validate row counts, then atomically swap into the target
- **Idempotency** — re-running yesterday's load should produce the same result, not duplicates. Achieve this with deterministic keys and `MERGE` or `INSERT ... ON CONFLICT` semantics.

## Load order

Dimensions first, then facts. Within dimensions, load parents before children if the hierarchy matters. Within facts, children depend on parents only through surrogate keys — so as long as all needed dimensions are up to date first, facts load independently.

## Orchestration

A production ETL pipeline needs a scheduler to run jobs in the right order, handle retries, alert on failure, and backfill. The common options:

- **Airflow** — open-source, Python-based DAGs, dominant in the open-source world
- **Prefect**, **Dagster** — modern alternatives with better local dev
- **dbt Cloud** — if you're all-in on ELT in a cloud warehouse
- **Cloud-native** — GCP Cloud Composer, AWS Step Functions, Azure Data Factory, Databricks Workflows
- **Vendor ETL suites** — Informatica, Talend, IBM DataStage, SSIS (SQL Server Integration Services), Oracle Data Integrator

## Quality gates and observability

Every production pipeline has automated checks at each stage:

- **Row count bounds** — did we load a plausible number of rows?
- **Schema drift** — did any column change type or disappear?
- **Freshness** — is the source up to date?
- **Referential integrity** — are there fact rows with missing dimension keys?
- **Null rate** — did null rates change significantly?
- **Business rule checks** — do totals reconcile against the source system?

Tools like **Great Expectations**, **dbt tests**, **Monte Carlo**, and **Soda** codify these as tests that run every load.

## Learning outcomes

- Distinguish ETL from ELT and say when each fits
- Choose an extraction pattern (full / incremental / CDC / file drop) for a given source
- Sequence transform operations correctly
- Implement dimension and fact loads in the right order, with SCD logic applied
- Identify the quality gates a production pipeline needs

