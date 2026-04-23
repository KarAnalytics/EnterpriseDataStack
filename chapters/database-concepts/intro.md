# Database Concepts

A **database management system (DBMS)** is software that stores, retrieves, and controls access to structured data. This chapter covers the conceptual foundation underneath everything you've written in SQL so far: the relational model, keys, integrity constraints, normalization, transactions, and the physical structures that make queries fast.

## Why a database instead of a spreadsheet?

Spreadsheets break at scale for predictable reasons:

- **Concurrency** — two people can't safely edit the same rows at once
- **Integrity** — nothing stops you from typing `"Texsa"` instead of `"Texas"`
- **Relationships** — a spreadsheet of orders can't cleanly reference a spreadsheet of customers
- **Security** — access is file-level, not row- or column-level
- **Performance** — Excel chokes around a million rows; a database is just warming up

A DBMS replaces all of those with explicit mechanisms: transactions, constraints, foreign keys, roles, and indexes.

## The relational model

Proposed by E.F. Codd in 1970 and still the dominant storage paradigm:

- Data is stored in **relations** (tables)
- Each row is a **tuple**; each column is an **attribute** with a declared type (domain)
- Row order is not semantically meaningful
- Relationships between tables are expressed with keys, not by embedding one table inside another

## Keys

- **Primary key** — uniquely identifies each row. One per table. Cannot be null.
- **Candidate key** — any minimal set of columns that could serve as a primary key
- **Alternate key** — candidate keys not chosen as the primary
- **Foreign key** — a column (or set) that references the primary key of another table; the database enforces that referenced values actually exist
- **Natural vs. surrogate** — natural keys come from the domain (email, SSN); surrogate keys are system-generated (auto-increment IDs, UUIDs). Surrogate keys are usually safer long-term.

## Integrity constraints

- **Entity integrity** — primary keys are unique and non-null
- **Referential integrity** — foreign keys resolve or are null
- **Domain integrity** — values conform to the column's type and any `CHECK` constraints
- **User-defined** — business rules enforced via constraints or triggers (e.g., `order_total >= 0`)

## Normalization

Normalization is a process of reorganizing tables to reduce redundancy and update anomalies. The standard forms build on each other:

| Form | Rule (informal) |
|---|---|
| **1NF** | Atomic values, no repeating groups, a primary key |
| **2NF** | 1NF plus: every non-key column depends on the *whole* primary key |
| **3NF** | 2NF plus: no transitive dependencies (non-key depends only on the key) |
| **BCNF** | Stricter 3NF: every determinant is a candidate key |
| **4NF / 5NF** | Eliminate multi-valued and join dependencies |

In practice, most OLTP schemas are designed to 3NF or BCNF. Data warehouses (Chapters 12–14) deliberately **denormalize** for query performance.

## Transactions and concurrency

A **transaction** is a unit of work that is ACID — **A**tomic, **C**onsistent, **I**solated, **D**urable (we met this in Chapter 3).

Isolation levels trade off correctness vs. throughput:

| Level | Prevents |
|---|---|
| Read Uncommitted | — |
| Read Committed | Dirty reads |
| Repeatable Read | + Non-repeatable reads |
| Serializable | + Phantom reads |

Databases implement isolation with **locks** (pessimistic) or **MVCC** (optimistic, multi-version concurrency control — used by PostgreSQL, Oracle, SQL Server in RCSI mode).

## Physical storage and indexing

- **Heap** — unordered table of rows
- **B-tree index** — balanced tree for range and equality lookups (the default)
- **Hash index** — exact-equality lookups only
- **Clustered index** — physically orders the table rows; one per table
- **Covering index** — includes all columns needed for a query; no table lookup required

The **query optimizer** uses statistics about index and column distributions to pick an execution plan. `EXPLAIN` (or `EXPLAIN ANALYZE`) shows you the plan — this is the single most valuable debugging tool in database work.

## DBMS categories

- **Relational (RDBMS)** — Oracle, PostgreSQL, MySQL, SQL Server, SQLite
- **Document** — MongoDB, Couchbase
- **Key-value** — Redis, DynamoDB
- **Wide-column** — Cassandra, HBase
- **Graph** — Neo4j, Amazon Neptune (Chapter 18)
- **Vector** — Pinecone, Chroma, Weaviate (Chapter 22)
- **Time-series** — InfluxDB, TimescaleDB

This course focuses on relational, but the concepts here (schemas, keys, transactions, indexes) show up in every category with small variations.

## Learning outcomes

- Explain why a DBMS replaces a spreadsheet for enterprise data
- Identify primary, candidate, alternate, foreign, natural, and surrogate keys
- Apply 1NF / 2NF / 3NF to normalize a denormalized table
- Reason about transactions, ACID, and isolation levels
- Read a basic `EXPLAIN` plan and name the kind of index each step uses

