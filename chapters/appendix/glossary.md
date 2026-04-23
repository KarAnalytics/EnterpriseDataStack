# Glossary

Short definitions of the terms used across this book.

**ACID** — Atomicity, Consistency, Isolation, Durability. The four guarantees a relational database makes for a transaction.

**Aggregate fact table** — A fact table pre-aggregated at a coarser grain than the base fact table for faster queries.

**ANN (Approximate Nearest Neighbor)** — Indexing techniques (HNSW, IVF, ScaNN) that trade small recall loss for fast vector similarity search.

**Bridge table** — A table that resolves a many-to-many relationship in a dimensional model.

**Candidate key** — Any minimal set of columns that could serve as a primary key.

**CDC (Change Data Capture)** — A pattern for streaming row-level changes from a source database's transaction log.

**Conformed dimension** — A dimension that is used identically across multiple fact tables, enabling cross-process analytics.

**CTE (Common Table Expression)** — A named, temporary result set inside a query, introduced with `WITH`.

**Cypher** — Neo4j's declarative, pattern-based graph query language.

**DCL (Data Control Language)** — `GRANT` / `REVOKE` — the sublanguage of SQL that controls access.

**DDL (Data Definition Language)** — `CREATE`, `ALTER`, `DROP` — the sublanguage of SQL that defines schema.

**Degenerate dimension** — A dimension attribute (typically a transaction id) stored on the fact row without its own dimension table.

**DIKW pyramid** — Data → Information → Knowledge → Wisdom; the progression from raw signals to acted-on insight.

**Dimension table** — A descriptive context table in a star schema.

**DML (Data Manipulation Language)** — `INSERT`, `UPDATE`, `DELETE`, `MERGE` — the sublanguage that changes rows.

**DSS (Decision Support System)** — A system (typically a data warehouse + BI layer) built for tactical and strategic decisions.

**Embedding** — A dense numeric vector representation of content (text, image, audio) produced by an ML model.

**ELT** — Extract, Load, Transform — variant of ETL where transformation happens inside the warehouse.

**ER diagram** — Entity-Relationship diagram; the standard notation for logical data models.

**ETL** — Extract, Transform, Load — the process of moving and reshaping data from sources into the warehouse.

**Factless fact table** — A fact table that records occurrence of events with no numeric measures.

**Fact table** — The central measurement table in a star schema.

**Foreign key** — A column that references the primary key of another table, enforced by the database.

**Galaxy schema** — Multiple fact tables sharing conformed dimensions; also called a fact constellation.

**GREL** — General Refine Expression Language; OpenRefine's expression language.

**HDFS** — Hadoop Distributed File System; the block-replicated storage layer of Hadoop.

**HNSW** — Hierarchical Navigable Small World; a popular ANN index structure.

**Idempotency** — A property of an operation that can be re-run safely without changing the outcome beyond the first run.

**Index** — A data structure (B-tree, hash, bitmap) that speeds up reads on specific columns at the cost of writes.

**Inmon architecture** — Top-down DW architecture: enterprise DW first, data marts derived from it.

**Junk dimension** — A dimension that bundles several low-cardinality flag columns into one.

**Kimball architecture** — Bottom-up DW architecture: build data marts directly with conformed dimensions.

**Lakehouse** — An architecture combining a data lake's cheap storage with warehouse-like transactional tables (Delta Lake, Iceberg, Hudi).

**MapReduce** — Hadoop's original compute model: map, shuffle, reduce.

**Materialized view** — A stored snapshot of a query's result, refreshed on a schedule or trigger.

**MERGE** — SQL statement that inserts, updates, or deletes in one pass based on a match condition.

**Mini-dimension** — A small dimension broken out from a larger one for attributes that change often.

**MVCC** — Multi-Version Concurrency Control; an implementation strategy that provides isolation without locking.

**Natural key** — A key drawn from real-world attributes (email, SSN, ISBN).

**NoSQL** — Any non-relational database: document, key-value, wide-column, graph, time-series.

**Normalization** — Reorganizing tables to reduce redundancy and anomalies. Forms: 1NF, 2NF, 3NF, BCNF, 4NF, 5NF.

**OLAP (Online Analytical Processing)** — Analytical query workload; optimized for large scans and aggregation.

**OLTP (Online Transaction Processing)** — Transactional workload; optimized for many small writes.

**OpenRefine** — Free desktop tool for interactive data cleanup, profiling, and clustering.

**Partition** — A subset of a table stored separately for pruning and parallelism; also used in Spark for distributed processing.

**Primary key** — The chosen unique identifier for each row in a table.

**Profiling** — Examining a dataset to produce summaries of its structure, values, and quality.

**RAG (Retrieval-Augmented Generation)** — Architecture pattern that retrieves relevant context from a vector (and/or keyword / graph) store and adds it to an LLM prompt.

**RDD** — Resilient Distributed Dataset; Spark's original low-level distributed collection abstraction.

**RDBMS** — Relational Database Management System.

**Referential integrity** — The guarantee that foreign keys resolve to actual rows in the parent table.

**Role-playing dimension** — A dimension used in multiple logical roles (e.g., `dim_date` as order date vs. ship date).

**SCD (Slowly Changing Dimension)** — Patterns for handling changes to dimension attribute values over time. Types 0–6.

**Shuffle** — Redistributing data across cluster nodes by key, needed for joins and group-bys in Spark.

**Snowflake schema** — A star schema whose dimensions are normalized into multiple tables.

**Spark** — The dominant in-memory distributed compute engine; unified API for batch, streaming, SQL, and ML.

**Staging** — A landing area where extracted data lives before transformation and loading into the warehouse.

**Star schema** — A dimensional model with one fact table and denormalized dimension tables radiating out.

**Surrogate key** — A system-generated, business-meaningless key (usually an auto-increment integer).

**Tableau / Power BI / Looker** — The most common BI tools that sit on top of a data warehouse.

**Transaction** — An atomic unit of work in a database; guaranteed ACID.

**TCL (Transaction Control Language)** — `COMMIT`, `ROLLBACK`, `SAVEPOINT`.

**Type 2 SCD** — The most common SCD pattern: add a new row for each attribute change with `effective_from` / `effective_to` / `is_current`.

**View** — A saved SQL query exposed as a virtual table.

**Watermark** — A timestamp (or similar counter) used to bound incremental loads.

**Window function** — A SQL function that computes across a set of rows related to the current row without collapsing the result.

**YARN** — Yet Another Resource Negotiator; Hadoop's cluster resource manager.
