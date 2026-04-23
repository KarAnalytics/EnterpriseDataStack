# EDM Foundations

Enterprise Data Management (EDM) is the discipline of moving raw data through an organization so it reliably produces knowledge that people can act on. Before we touch SQL or design a warehouse, we need a shared vocabulary for the levels of decision-making data supports, the journey from raw bytes to wisdom, and the components of the EDM pipeline itself.

## Decision-making levels

Every enterprise data system is ultimately built to support decisions, and decisions come in three flavors:

- **Operational** — day-to-day decisions on short time frames. Powered by **OLTP** (Online Transaction Processing) systems optimized for many small `INSERT`, `UPDATE`, `DELETE`, and simple `SELECT` statements. Throughput matters.
- **Tactical** — middle-management decisions on medium time horizons.
- **Strategic** — senior-management decisions with long-term focus.

Tactical and strategic decisions are served by **Decision Support Systems (DSS)** and data warehouses, which focus on complex ad-hoc `SELECT` queries, multidimensional representations of the data, and trend analysis. The architectural split between OLTP and OLAP/DW is a through-line for the rest of this book.

## The data journey: DIKW

Data doesn't become useful by itself. It moves up a ladder:

1. **Data** — raw signals (`Red, 192.234.235.245.678, v2.0`)
2. **Information** — data given meaning (*"the south-facing traffic light at Pitt and George has turned red"*)
3. **Knowledge** — information placed in context (*"the light I'm driving toward is red"*)
4. **Wisdom** — knowledge applied (*"I should stop the car"*)

Databases are the medium through which this transformation happens at enterprise scale.

## What EDM actually covers

Enterprise data management is a seven-component cycle:

1. **Planning** — define goals, scope, data sources, cost-benefit trade-offs for the EDM ecosystem
2. **Information Extraction (Ingestion)** — acquire raw data from internal systems, surveys, the web, IoT, logs, etc.
3. **Data Integration** — combine disparate sources into a unified view (ETL is the canonical pattern)
4. **Data Wrangling (Quality Control)** — profile, process, and validate data so it is accurate and reliable
5. **Database Management** — store transactional data using RDBMS / OLTP systems
6. **Data Warehousing** — structure data for efficient querying, reporting, and downstream ML (OLAP)
7. **Data Governance** — the policies, standards, and controls (ethics, security, privacy, fairness, regulatory compliance) that sit across every other step

Governance is the component students most often undervalue and professionals most often under-invest in. It is arguably the *most* important component of a modern EDM system and it touches every decision level — operational, tactical, and strategic.

## Beyond the foundations

Later in the book we extend the EDM pipeline with three newer categories:

- **Big data** — managing data at volumes, variety, velocity, value, and veracity (the 5 Vs) that exceed a single machine. Hadoop and Spark are the milestone technologies.
- **Graph data management** — property-graph databases for highly connected data (knowledge graphs, supply chains, social networks).
- **Vector databases** — storage and similarity search for high-dimensional embeddings of unstructured text, images, and audio; the storage layer behind generative AI and RAG.

## Command-line basics

Many EDM tools — Hadoop, Spark, GCP VMs, Linux servers — are operated through a terminal rather than a GUI. The metaphor: a GUI is like ordering from a menu with pictures (user-friendly but limited to the options shown); a CLI is like speaking directly to the chef (highly specific, composable, automatable).

Core commands every analyst should know:

- Navigation: `pwd`, `ls`, `cd`
- File & directory work: `mkdir`, `cp`, `mv`, `rm` (use with caution — `rm` has no undo), `rm -r` for directories
- File creation and viewing: `touch`, `vi` (enter insert mode with `i`, exit with `Esc`, save and quit with `:wq`), `cat`

Practice on a free sandbox like [linuxcontainers.org/incus/try-it](https://linuxcontainers.org/incus/try-it) (30-minute sessions) or work through the first two interactive modules on [LinuxSurvival.com](https://linuxsurvival.com).

## Learning outcomes

By the end of this chapter you should be able to:

- Distinguish OLTP from OLAP/DSS workloads and match them to decision levels
- Explain the DIKW pyramid with a concrete example
- Name the seven components of the EDM pipeline and describe what each one does
- Articulate why governance spans every other component
- Run basic terminal commands to navigate a filesystem, create files, and view their contents

