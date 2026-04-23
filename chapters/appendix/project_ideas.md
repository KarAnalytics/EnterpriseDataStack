# Project Ideas

End-of-semester project prompts, designed to exercise multiple chapters at once. Each one is scoped for a team of 3–4 students working over 4–6 weeks.

## 1. Retail sales warehouse

**What to build:** End-to-end pipeline from a public retail dataset (e.g., Online Retail II, Brazilian E-Commerce) into a Kimball star schema, with SCD Type 2 on customers.

**Chapters exercised:** SQL (Part II), profiling & processing (Part III), databases (Part IV), warehousing + ETL (Part V).

**Deliverables:** ER diagram of the source, star schema diagram of the warehouse, SQL scripts for DDL and the ETL, and five business-question queries answered against the warehouse.

---

## 2. Airline operational dashboard

**What to build:** Ingest a public flight dataset (BTS On-Time Performance, OpenSky), load into a warehouse with role-playing date dimensions, build a Power BI / Tableau / Looker Studio dashboard.

**Chapters exercised:** DW concepts, advanced DW design (role-playing, conformed dims), ETL, SQL querying.

**Deliverables:** Dashboard with filters by carrier, route, and month; on-time % by route; delay breakdown by cause; aircraft utilization.

---

## 3. Fraud detection with a graph database

**What to build:** Load a synthetic or public payments dataset (PaySim, Elliptic++) into Neo4j; detect suspicious rings with Cypher queries and centrality algorithms.

**Chapters exercised:** Graph databases, data processing, governance.

**Deliverables:** Graph schema, loading scripts, three Cypher queries finding ring-fraud, mule accounts, and anomalous high-degree nodes; a short governance memo on what controls a bank would need before deploying this.

---

## 4. University knowledge graph

**What to build:** Scrape public course catalog + faculty pages; model as a knowledge graph (faculty, courses, departments, research areas); query it with Cypher.

**Chapters exercised:** Graph databases, data wrangling, data governance (PII considerations).

**Deliverables:** Graph model, loader, five Cypher queries ("which faculty could co-teach a course on X?", "shortest collaboration path between two researchers").

---

## 5. Hadoop / Spark log analytics on GCP

**What to build:** Process a large log dataset (NASA access logs, Common Crawl subset) on Dataproc; produce hourly / daily aggregations; write results to BigQuery or back to GCS as Parquet.

**Chapters exercised:** Big Data trio (Hadoop, GCP Dataproc, Spark).

**Deliverables:** Spark job source, cluster create/delete scripts, cost log for the run, output dashboard.

---

## 6. RAG over lecture slides

**What to build:** Chunk every lecture PDF from this course, embed with an open-source model, store in a vector DB (Chroma or pgvector), expose a chat interface that answers questions with citations.

**Chapters exercised:** Vector databases — plus the companion AI book if you want to go deeper.

**Deliverables:** Ingestion script, vector DB with metadata filters (by chapter), chat UI, 10 test questions with actual vs. expected answers, discussion of hallucination risks.

---

## 7. Data governance audit

**What to build:** Pick a public dataset (e.g., Chicago taxi trips, city-level crime data) and produce a full governance audit: catalog entry, PII classification, lineage diagram of a realistic downstream pipeline, retention policy, DPIA-style risk assessment.

**Chapters exercised:** Data governance, plus touches on every data-handling chapter.

**Deliverables:** Catalog spec, classification table, lineage diagram, retention policy, risk assessment memo.

---

## 8. Lakehouse migration plan

**What to build:** Take a mid-sized public dataset and implement the same pipeline three ways — (a) PostgreSQL warehouse, (b) BigQuery / Snowflake, (c) Spark on Dataproc writing Delta or Iceberg. Compare cost, latency, and developer experience.

**Chapters exercised:** Warehousing concepts, Spark, big data architecture.

**Deliverables:** Pipeline code for all three variants, a benchmarking notebook, and a 3-page recommendation memo.

---

## 9. OpenRefine + dbt cleanup pipeline

**What to build:** Take a genuinely messy public dataset (e.g., WHO country indicators with inconsistent names, USGS earthquakes with mixed date formats) and run it through OpenRefine → CSV → a dbt project in DuckDB or PostgreSQL that produces a clean star schema.

**Chapters exercised:** Data processing, OpenRefine, profiling, warehousing.

**Deliverables:** OpenRefine operation recipe, dbt project, documented quality tests.

---

## 10. Healthcare warehouse with governance constraints

**What to build:** Using the MIMIC-III / MIMIC-IV demo dataset (public, de-identified ICU data), design a star schema for visits with de-identification and access-control considerations baked in.

**Chapters exercised:** Data warehousing design, governance, SQL.

**Deliverables:** Data-use agreement adherence memo, schema diagram, DDL, three clinical analytic queries, and an access-control plan (who can see what).

---

## Evaluation rubric (suggested)

Rate each project on five dimensions, 1–5 scale:

1. **Correctness** — does it produce the right answers?
2. **Design quality** — is the schema / pipeline / model well-chosen for the problem?
3. **Code quality** — readable, organized, version-controlled?
4. **Governance awareness** — appropriate controls, privacy, and documentation?
5. **Communication** — can a business stakeholder understand the deliverables?
