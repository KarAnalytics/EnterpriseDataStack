# The Enterprise Data Stack: A Practitioner's Handbook

Welcome to a practitioner's handbook covering the eight layers that move raw data through an enterprise and turn it into knowledge — from SQL at the bottom to vector databases at the top. This book accompanies **BSAN 726: Enterprise Data Management** at the University of Kansas School of Business and is organized as a sequence of short, concept-forward chapters — each mapped to a lecture deck you can download, read, and reference.

## What this book covers

```{image} https://img.shields.io/badge/Chapters-22-blue
```
```{image} https://img.shields.io/badge/📥_Download-PDF-red
:target: https://karanalytics.github.io/EnterpriseDataStack/enterprise-data-stack.pdf
```
```{image} https://img.shields.io/badge/License-MIT-green
```

Data management is the backbone of every analytics and AI system. Before you can model, forecast, or reason over data, someone has to design the schemas, write the SQL, wrangle the inputs, govern access, load the warehouse, and — increasingly — manage graph and vector stores for modern workloads. This book walks through those layers in eight parts:

**Part I — Foundations:** Where enterprise data management fits in the decision-making stack (operational, tactical, strategic), the data-information-knowledge-wisdom journey, and the command-line skills you'll use throughout.

**Part II — SQL:** The language of data. DDL to define schemas, DCL to control access, DML to change rows, and the full query vocabulary — from simple SELECTs to window functions, CTEs, and analytic queries.

**Part III — Data Wrangling & Governance:** Real-world data is messy. Techniques for data processing, profiling with tools like OpenRefine, and the governance policies (ethics, security, privacy, regulatory compliance) that keep enterprise data trustworthy.

**Part IV — Databases:** Relational database concepts — keys, constraints, normalization, transactions — and practical data modeling using draw.io to design entity-relationship diagrams.

**Part V — Data Warehousing & ETL:** Why operational databases aren't enough for analytics. Star and snowflake schemas, slowly changing dimensions, aggregate design, and ETL pipelines — with a full Northwind-to-DW worked example.

**Part VI — Graph Databases:** When your data is a network (customers, products, supply chains, knowledge), property graphs and Cypher queries are the right tool.

**Part VII — Big Data:** Scaling beyond a single machine with Hadoop (HDFS, MapReduce, YARN), running a managed cluster on GCP Dataproc, and in-memory analytics with Apache Spark.

**Part VIII — Vector Databases:** The storage layer behind retrieval-augmented generation and modern AI apps — embeddings, similarity search, and the leading vector DB options.

## How to use this book

Each chapter is a short narrative walkthrough of the concepts, closing with a set of learning outcomes. You can:

- **Read online** — browse the rendered HTML chapters on GitHub Pages
- **Download the full PDF** — a compiled PDF of the entire book is available at the top of the site

Companion software and accounts referenced throughout the course include DB Browser for SQLite, MySQL / PostgreSQL, Oracle Cloud, OpenRefine, draw.io, Neo4j, and Google Cloud Platform (for Dataproc and Spark).

## Companion book

This book has a sister: [**AI for Business: A Hands-On Guide**](https://karanalytics.github.io/AIHandsOnBook/), which picks up where this one ends — using the well-managed data from an EDM stack to build LLM-powered applications, RAG systems, and agents.

## How to cite this book

> Srinivasan, K. (2026). *The enterprise data stack: A practitioner's handbook*. https://karanalytics.github.io/EnterpriseDataStack/

BibTeX:

```bibtex
@book{srinivasan2026edstack,
  title  = {The Enterprise Data Stack: A Practitioner's Handbook},
  author = {Srinivasan, Karthik},
  year   = {2026},
  url    = {https://karanalytics.github.io/EnterpriseDataStack/}
}
```

## Author

[Karthik Srinivasan](https://business.ku.edu/people/karthik-srinivasan), University of Kansas School of Business.

## A Note on AI Collaboration

This book was assembled with help from Claude Code (Anthropic) as a writing and infrastructure partner — drafting chapter prose from lecture slides, structuring the MyST project, and wiring the GitHub Pages deployment. All content was reviewed, directed, and approved by the author, who takes full responsibility for the accuracy and pedagogical choices.

---

*Built with [MyST](https://mystmd.org). Source at [github.com/KarAnalytics/EnterpriseDataStack](https://github.com/KarAnalytics/EnterpriseDataStack).*
