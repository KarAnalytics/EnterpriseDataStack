# Data Warehousing Concepts

Operational databases (OLTP) are built to serve one question fast, millions of times a day: *"give me this customer's current balance"*. Data warehouses (OLAP) are built to answer a different kind of question: *"what were our top-10 products by revenue across the last 36 months, broken out by region and season?"* Those two workloads pull schema design in opposite directions — and that tension is what data warehousing is about.

## OLTP vs. OLAP at a glance

| Dimension | OLTP | OLAP / Data Warehouse |
|---|---|---|
| Primary operations | `INSERT`, `UPDATE`, `DELETE`, point `SELECT` | complex `SELECT` / aggregation |
| Optimized for | Transaction throughput | Query throughput over large scans |
| Schema style | Highly normalized (3NF/BCNF) | Denormalized (star / snowflake) |
| Data age | Current state | Historical, typically months to years |
| Row count | Millions | Billions |
| Users | Many, concurrent, short transactions | Fewer, analytical, long queries |
| Typical example | A banking transaction system | A bank's risk / reporting warehouse |

## Inmon vs. Kimball

The two foundational architectures, both still in widespread use:

- **Inmon (Corporate Information Factory)** — build an enterprise-wide, normalized **Enterprise Data Warehouse (EDW)** first; feed subject-oriented **data marts** from it. Top-down. Slower to deliver; cleaner long-term.
- **Kimball (Dimensional modeling)** — build **data marts** directly using star schemas, then connect them via **conformed dimensions**. Bottom-up. Faster to deliver; relies on disciplined dimension management.

Most modern warehouses are Kimball-style at the presentation layer, Inmon-ish underneath.

## Core warehouse properties (Inmon's definition)

A data warehouse is:

- **Subject-oriented** — organized around business subjects (customer, product) rather than applications
- **Integrated** — consistent naming, units, and encodings across sources
- **Time-variant** — every fact is timestamped; history is preserved
- **Non-volatile** — data is loaded and queried, not updated in place

## Lake, warehouse, and lakehouse

The modern landscape has three patterns:

- **Data lake** — raw files (CSV, JSON, Parquet) in cheap object storage (S3, GCS, ADLS). Schema-on-read. Flexible, cheap, easy to dump into; hard to query without governance.
- **Data warehouse** — structured, columnar analytical store (Snowflake, BigQuery, Redshift, Synapse). Schema-on-write. Fast queries; more expensive per TB.
- **Lakehouse** — combines the two: tables live on lake storage but are accessed through a transactional layer (Delta Lake, Apache Iceberg, Apache Hudi). Databricks and modern BigQuery/Snowflake external tables are the reference implementations.

## Metadata and lineage

A warehouse is only as trustworthy as its metadata. Expect any serious platform to track:

- **Technical metadata** — schemas, data types, sizes, partitions
- **Business metadata** — definitions, owners, SLAs
- **Operational metadata** — load times, row counts, freshness, quality scores
- **Lineage** — the exact path from source system to final report

## OLAP operations

When you look at a warehouse through a BI tool (Power BI, Tableau, Looker), you're doing OLAP:

- **Slice** — fix one dimension (e.g., `region = 'West'`)
- **Dice** — fix multiple dimensions (e.g., `region = 'West' AND quarter = 'Q3'`)
- **Drill-down** — move from summary to detail (year → quarter → month)
- **Roll-up** — aggregate upward (day → month → year)
- **Pivot** — swap rows and columns to see the data from a different angle

These operations map directly to the **star schema** we'll build in the next chapter.

## Learning outcomes

- Contrast OLTP and OLAP workloads and explain why they need different schema designs
- Describe the Inmon and Kimball architectures and when each fits
- Define Inmon's four warehouse properties
- Distinguish a data lake, a data warehouse, and a lakehouse
- Name the five OLAP operations

