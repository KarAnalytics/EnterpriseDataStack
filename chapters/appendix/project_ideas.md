# Project Ideas

Projects that combine several chapters, using the free tools of this book. Each is scoped for a team of three or four people over four to six weeks, or for one person working on their own at a slower pace. Every project should end with a short report that states your assumptions, shows your design diagrams, and explains how you checked your results.

## 1. From open data to a data warehouse

**What to build:** Take a public dataset from a city open-data portal (for example, 311 service requests, building permits or crime reports) and build a dimensional data mart for it in Oracle Cloud.

**Steps:** Profile the raw data ([Data Profiling](../data-profiling/intro.md)) and clean it in OpenRefine or Python ([Data Processing](../data-processing/intro.md)). Design a star schema with a declared grain ([Data Warehouse Design](../dw-design/intro.md)), load it with SQL ([ETL](../etl-procedure/intro.md)), and answer five business questions with star-join queries.

**Deliverables:** profiling summary, cleaning recipe (OpenRefine operation history or code), star schema diagram in draw.io, ETL script, reconciliation of row counts and totals, and the five queries with their results.

## 2. Designing a database for a small organization

**What to build:** Interview someone who runs a small organization (a club, a clinic, a shop, a nonprofit) and design a database for one of its processes.

**Steps:** Write the business rules, draw an EER diagram ([Databases and the ER Model](../er-model/intro.md)), map it to a relational model ([The Relational Model](../relational-model/intro.md)), check it with normalization ([Normalization](../normalization/intro.md)), and implement it with DDL and sample data ([SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md)).

**Deliverables:** business rules, EER diagram with assumptions, relational model, DDL script with constraints, sample data, ten SQL queries the organization would actually ask, and a reverse-engineered diagram from Oracle's Data Modeler compared with your design.

## 3. A second data mart with conformed dimensions

**What to build:** Extend the Northwind warehouse of [ETL Example: Northwind](../etl-northwind/intro.md) with a second fact table, for example freight at the order grain, or an inventory snapshot of products.

**Steps:** Declare the new grain, reuse (conform) the existing dimensions, load the new facts, and write queries that combine both fact tables.

**Deliverables:** constellation diagram, ETL script, reconciliation, and five cross-process queries (for example, freight cost as a share of sales by shipper and year).

## 4. A slowly changing dimension in practice

**What to build:** Simulate a year of changes to a customer or product dimension (moves, segment changes, price changes) and implement Type 1, Type 2 and Type 3 handling for different attributes.

**Steps:** Design which attributes get which treatment and why ([Advanced DW Design](../dw-design-advanced/intro.md)), write the SQL that applies each month's changes, and show how a report on last year's sales changes (or does not) under each type.

**Deliverables:** design rationale, SQL for each change type, before-and-after tables, and the comparison of reports.

## 5. A network analysis with Neo4j

**What to build:** Model a connected dataset as a property graph: the trade network of [Graph Databases](../graph-databases/intro.md), a public transport network, a citation network, or co-authorship data.

**Steps:** Design the nodes, relationships and properties; load them with `LOAD CSV`; profile the graph (counts, zero-valued relationships, missing coverage); and answer questions that need several hops.

**Deliverables:** graph model diagram, load script, verification queries, five multi-hop Cypher queries, and a comparison with how the same questions would be written in SQL.

## 6. A data governance review

**What to build:** Choose a dataset used in a real or realistic organization (for example, a student success model or a customer loyalty program) and write a governance plan for it.

**Steps:** Identify the applicable laws and policies, the data owners and stewards, the minimum data needed, the protection measures, the ethical risks, and the retention period ([Data Governance](../data-governance/intro.md)). Profile a sample to find quality issues that would affect decisions.

**Deliverables:** a governance plan of three to five pages, a data inventory with classifications, and a short risk register.

## 7. Big data with Spark

**What to build:** Analyze a dataset too large to open comfortably in Excel (several million rows, such as a year of taxi trips or bike-share rides) with PySpark ([Apache Spark](../spark/intro.md)).

**Steps:** Read the data with an explicit schema, profile it, clean it, and answer analytical questions with both the DataFrame API and Spark SQL. Optionally, run the same job on a cloud Hadoop cluster and compare.

**Deliverables:** notebook with commentary, a profile of the data, five analytical results, and a note on what you would change to run the job on a real cluster.

## Evaluation (suggested)

| Criterion | Weight |
|---|---|
| Correctness: designs follow the rules, queries return verified results | 30% |
| Data quality: profiling, cleaning and reconciliation are shown | 20% |
| Design rationale: assumptions and choices are stated and justified | 20% |
| Communication: clear diagrams and a readable report | 20% |
| Governance and ethics: privacy, access and fairness are addressed | 10% |
