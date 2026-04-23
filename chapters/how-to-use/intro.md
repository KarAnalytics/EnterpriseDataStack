# How to Use This Book

This book is the companion reading for **BSAN 726: Enterprise Data Management**. It is organized to follow the semester's lecture sequence — 22 chapters grouped into eight parts — but each chapter is self-contained enough to use as a standalone reference later in your career.

## Reading paths

**As a student in BSAN 726:** Read the chapter before class alongside the lecture deck on Canvas. After class, revisit the chapter and work through the practice prompts at the end of each section. Most chapters take 20–40 minutes to read.

**As a practitioner brushing up:** Jump to the chapter you need. Part II (SQL) and Parts IV–V (Databases, Warehousing, ETL) are the most commonly revisited by alumni.

**As an instructor adopting the material:** The chapter-to-topic mapping tracks a typical semester in enterprise data management. Reuse and adapt as you see fit under the project's MIT license.

## Each chapter contains

- **Why this matters** — the business problem the topic solves
- **Core concepts** — definitions and key ideas
- **Worked examples** — concrete illustrations where appropriate
- **Learning outcomes** — what you should be able to do after finishing

## Software you'll need

Different chapters assume different tools. You don't need all of these on day one — install them as chapters introduce them:

- A SQL client — **DB Browser for SQLite** (free, cross-platform) is the easiest starting point; **MySQL Workbench** or **Oracle SQL Developer** also work
- **OpenRefine** — free, runs locally in a browser (Chapter 7)
- **draw.io** (a.k.a. diagrams.net) — free, browser-based (Chapter 11)
- **Neo4j Desktop** or **Neo4j Aura** — free tier for graph databases (Chapter 18)
- A **Google Cloud Platform** account with billing enabled — Dataproc and Spark labs (Chapters 20–21)
- A terminal / command line — macOS and Linux have one built in; on Windows use **Windows Terminal** or **Git Bash**

## Conventions used in this book

- Inline code like `SELECT * FROM customers` refers to code you'd type into a SQL client
- Blocks like this:
  ```sql
  SELECT customer_id, SUM(amount) AS total
  FROM orders
  GROUP BY customer_id;
  ```
  are complete, runnable snippets
- File paths, table names, and column names are shown in `monospace`
- Definitions in **bold** at first use

## Feedback

Corrections and suggestions are welcome. Open an issue on [GitHub](https://github.com/KarAnalytics/EnterpriseDataStack/issues) or email the author.
