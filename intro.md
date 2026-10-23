# The Enterprise Data Stack: A Practitioner's Handbook

:::{image} cover.png
:alt: The Enterprise Data Stack cover emblem: a network of connected nodes inside a hexagon, in navy and bright blue.
:width: 260px
:align: center
:::

Welcome to a concept-first guide to how data moves through an organization and becomes something people can trust and use: from cleaning raw data, through designing databases and data warehouses and querying them with SQL, to graph databases and big data platforms. Every chapter explains the ideas, works through a complete example, and ends with practice you can do in free, professional tools.

## What this book covers

![Chapters](https://img.shields.io/badge/Chapters-22-blue)
[![Practice](https://img.shields.io/badge/Practice-Oracle_Cloud-red)](chapters/appendix/oracle-cloud.md)
[![Download PDF](https://img.shields.io/badge/📥_Download-PDF-red)](https://karanalytics.github.io/EnterpriseDataStack/enterprise-data-stack.pdf)
[![License](https://img.shields.io/badge/License-MIT-green)](https://github.com/KarAnalytics/EnterpriseDataStack)

Data management is the backbone of every analytics and AI system. Before anyone can model, forecast or reason over data, someone has to clean it, design the schemas, govern access, load the warehouse and write the queries. This book follows that journey in eight parts:

**Part I: Foundations.** Where enterprise data management fits in decision making (operational, tactical, strategic), the data journey from raw data to wisdom, the components of EDM, and the command-line skills used throughout.

**Part II: Data Wrangling and Governance.** Real data is messy. Data processing, cleaning with OpenRefine, data profiling with OpenRefine, Python and R, data governance (ethics, security, privacy and regulation), and regular expressions for pattern matching.

**Part III: Database Design.** The entity-relationship (ER) and enhanced ER models in crow's foot notation, drawing them in draw.io, mapping them to a relational model, and normalization.

**Part IV: Data Warehousing.** Why operational databases are not enough for analytics: warehouse concepts, star and snowflake schemas, advanced dimensional design, and warehouse examples from many industries.

**Part V: SQL.** The language of data: defining tables, controlling access and changing rows (DDL, DCL and DML), simple and aggregate queries, joins, subqueries and other advanced queries.

**Part VI: ETL.** Extracting, transforming and loading data from an operational database into a warehouse, with a complete Northwind example.

**Part VII: Graph Databases.** When your data is a network, property graphs in Neo4j and the Cypher query language are the right tool.

**Part VIII: Big Data.** Scaling beyond one machine with Hadoop (HDFS, MapReduce, YARN) and in-memory analytics with Apache Spark.

The appendices explain how to set up a free Oracle Cloud database and describe the sample data used in many SQL examples.

## How to use this book

The book is the companion text for **BSAN 726 (in person) and BSAN 727 (online): Enterprise Data Management** at the University of Kansas School of Business. Each chapter follows one topic of the course, so students can use it as a single place to review everything that was covered, during the course and long after it. It is also written for anyone who wants a practical introduction to enterprise data management. You can:

- **Read online**: browse the chapters right here.
- **Download the PDF**: the complete book as one file, from the button at the top of every page.
- **Practice**: every chapter ends with exercises in free tools. All SQL runs in a free Oracle Cloud database in your browser; see [Setting up Oracle Cloud](chapters/appendix/oracle-cloud.md). Diagrams are drawn in draw.io, data is cleaned in OpenRefine, and graphs are queried in Neo4j.

[How to Use This Book](chapters/how-to-use/intro.md) suggests reading paths for different readers and lists recommended textbooks.

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

This book was developed through a human-AI collaboration. The author used Claude Code (Anthropic) as a development partner: turning the author's lecture slides and course materials into chapter prose, creating the worked examples and diagrams, drafting practice questions, and assembling the MyST book and its deployment. All content was reviewed, directed and approved by the author, who takes full responsibility for the accuracy and pedagogical choices in the final work.

The same collaboration pattern runs through the book itself. Each chapter has a **Check the AI** box showing how a GenAI assistant typically goes wrong on that topic and how to verify its answers, because knowing the material well enough to catch those mistakes is exactly what the book aims to teach.

---

*Built with [MyST](https://mystmd.org). Source at [github.com/KarAnalytics/EnterpriseDataStack](https://github.com/KarAnalytics/EnterpriseDataStack).*
