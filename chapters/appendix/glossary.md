# Glossary

Short definitions of the key terms in this book, with the chapter where each is explained.

## A to C

**ACID**: the four guarantees a relational database gives every transaction: atomicity, consistency, isolation and durability. [SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md)

**Action (Spark)**: an operation that computes and returns a result, such as `count()` or `show()`, triggering the lazy transformations before it. [Apache Spark](../spark/intro.md)

**Additive fact**: a fact that can be summed across every dimension, such as sales amount. Compare semi-additive and non-additive facts. [Advanced DW Design](../dw-design-advanced/intro.md)

**Aggregate function**: a function that summarizes many rows into one value: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`. [SQL Queries](../sql-queries/intro.md)

**Anomaly (update, insertion, deletion)**: a problem caused by redundant data when rows are changed, added or removed. [Normalization](../normalization/intro.md)

**Associative entity**: an entity class that represents a many-to-many relationship with attributes of its own, drawn with rounded corners. [Databases and the ER Model](../er-model/intro.md)

**Attribute**: a named property of an entity class (simple, composite, multivalued, derived or identifier); it becomes a column. [Databases and the ER Model](../er-model/intro.md)

**Big data**: data whose volume, velocity, variety, veracity and value exceed what traditional systems can handle. [Introduction to Hadoop](../hadoop/intro.md)

**Business key** (natural key): the identifier used in the source system, kept as an attribute in a warehouse dimension. [Advanced DW Design](../dw-design-advanced/intro.md)

**Cardinality (relationships)**: the minimum and maximum number of instances of one entity class related to an instance of another. [Databases and the ER Model](../er-model/intro.md)

**Cardinality (profiling)**: the number of distinct actual values in a column. [Data Profiling](../data-profiling/intro.md)

**Cartesian product**: every row of one table paired with every row of another, the result of a join with no join condition. [SQL Queries](../sql-queries/intro.md)

**Changed data capture (CDC)**: tracking and extracting only the source data that changed since the last extraction. [ETL](../etl-procedure/intro.md)

**Clustering (OpenRefine)**: grouping values that are probably the same thing spelled differently, using key collision or nearest neighbor methods. [OpenRefine](../openrefine/intro.md)

**Common table expression (CTE)**: a named subquery defined with `WITH` at the start of a query. [Advanced SQL Queries](../sql-advanced/intro.md)

**Completeness**: actual values divided by total records. [Data Profiling](../data-profiling/intro.md)

**Conformed dimension**: a dimension shared, with the same meaning, by several fact tables or data marts. [Data Warehousing Concepts](../dw-concepts/intro.md)

**Correlated subquery**: a subquery that refers to a column of the outer query, so it is evaluated for each outer row. [Advanced SQL Queries](../sql-advanced/intro.md)

**Crow's foot notation**: the ER notation that draws entity classes as boxes of attributes and relationships as lines with cardinality symbols at the ends. [Databases and the ER Model](../er-model/intro.md)

**CRUD**: create, read (retrieve), update and delete, the core operations on any database. [Databases and the ER Model](../er-model/intro.md)

**Cypher**: Neo4j's pattern-based graph query language. [Graph Databases](../graph-databases/intro.md)

## D to F

**Data governance**: the policies, roles and controls that ensure data is handled securely, ethically and consistently. [Data Governance](../data-governance/intro.md)

**Data lake**: a central repository of data stored in its raw, unprocessed form. [Data Warehousing Concepts](../dw-concepts/intro.md)

**Data mart**: a data warehouse with a limited scope, for one function or department; dependent (fed from a warehouse) or independent. [Data Warehousing Concepts](../dw-concepts/intro.md)

**Data processing**: adjusting data so that it is organized, readable and ready for producing information. [Data Processing](../data-processing/intro.md)

**Data profiling**: examining a data source and summarizing what it actually contains. [Data Profiling](../data-profiling/intro.md)

**Data quality**: fitness for use: whether data is good enough for a particular decision. [Data Profiling](../data-profiling/intro.md)

**Data steward**: the person who looks after a data domain day to day: definitions, quality and issues. [Data Governance](../data-governance/intro.md)

**Data warehouse**: an integrated, enterprise-wide collection of high-quality information from different databases, used to support decision making. [Data Warehousing Concepts](../dw-concepts/intro.md)

**DataFrame**: a distributed table with named columns in Spark (or an in-memory table in pandas and R). [Apache Spark](../spark/intro.md)

**DBMS**: database management system, the software that creates, maintains and controls access to databases; an RDBMS is a relational one. [Databases and the ER Model](../er-model/intro.md)

**DCL, DDL, DML, TCL**: the groups of SQL commands that control access (`GRANT`, `REVOKE`), define structures (`CREATE`, `ALTER`, `DROP`), manipulate data (`INSERT`, `UPDATE`, `DELETE`, `SELECT`), and control transactions (`COMMIT`, `ROLLBACK`). [SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md)

**Degenerate dimension**: a dimension key, such as a transaction number, stored in the fact table with no dimension table of its own. [Advanced DW Design](../dw-design-advanced/intro.md)

**Denormalization**: deliberately adding redundancy, as in data warehouse dimensions, to simplify and speed up queries. [Normalization](../normalization/intro.md)

**DIKW**: data, information, knowledge, wisdom: the stages by which raw data becomes action. [EDM Foundations](../edm-foundations/intro.md)

**Dimension table**: a table of descriptive attributes (who, what, where, when) used to filter, group and label facts. [Data Warehouse Design](../dw-design/intro.md)

**Disjoint / overlapping**: whether an instance can belong to at most one subclass (disjoint, d) or to several (overlapping, o). [Databases and the ER Model](../er-model/intro.md)

**Distinctness**: cardinality divided by actual values. [Data Profiling](../data-profiling/intro.md)

**ELT**: extract, load, then transform inside the target system. [ETL](../etl-procedure/intro.md)

**Encoding (character)**: how characters are stored as bytes, such as UTF-8 or Windows-1252; reading with the wrong one garbles text. [Data Processing](../data-processing/intro.md)

**Enterprise data management (EDM)**: managing an organization's data through processes that turn raw data into knowledge: planning, extraction, integration, wrangling, databases, warehousing and governance. [EDM Foundations](../edm-foundations/intro.md)

**Entity, entity class, entity instance**: a thing the organization keeps data about; the collection of such things; and one occurrence. [Databases and the ER Model](../er-model/intro.md)

**ER model**: the entity-relationship model, a conceptual model of entities, attributes and relationships. EER adds superclasses and subclasses. [Databases and the ER Model](../er-model/intro.md)

**ETL**: extract, transform, load: moving data from sources into a data warehouse. [ETL](../etl-procedure/intro.md)

**Fact table**: the central table of a star schema, holding the measurements of a business process at a declared grain, with foreign keys to the dimensions. [Data Warehouse Design](../dw-design/intro.md)

**Factless fact table**: a fact table with only foreign keys, recording that events or combinations occurred. [Advanced DW Design](../dw-design-advanced/intro.md)

**Foreign key**: a column that refers to the primary key of another (or the same) table. [The Relational Model](../relational-model/intro.md)

**Functional dependency**: A → B means each value of A determines exactly one value of B; it can be full, partial or transitive. [Normalization](../normalization/intro.md)

## G to N

**Grain**: the level of detail of one row of a fact table. [Data Warehouse Design](../dw-design/intro.md)

**GREL**: General Refine Expression Language, OpenRefine's language for transforming values. [OpenRefine](../openrefine/intro.md)

**HDFS**: the Hadoop Distributed File System, which stores files as replicated blocks across a cluster. [Introduction to Hadoop](../hadoop/intro.md)

**Inmon / Kimball**: the top-down approach (one normalized enterprise warehouse feeding data marts) and the bottom-up approach (the warehouse as conformed dimensional data marts). [Data Warehousing Concepts](../dw-concepts/intro.md)

**Join (inner, outer)**: combining tables on common columns; an inner join keeps matching rows only, an outer join also keeps unmatched rows from one or both sides. [SQL Queries](../sql-queries/intro.md)

**Junk dimension**: a dimension listing the combinations of several low-cardinality flags. [Advanced DW Design](../dw-design-advanced/intro.md)

**Labeled property graph**: a graph of nodes and relationships, each with labels or types and properties. [Graph Databases](../graph-databases/intro.md)

**Lazy evaluation**: Spark's practice of recording transformations and running them only when an action needs a result. [Apache Spark](../spark/intro.md)

**MapReduce**: a model for processing data in parallel in map, shuffle and reduce phases. [Introduction to Hadoop](../hadoop/intro.md)

**Materialized view**: a view whose result is stored and refreshed periodically. [Advanced SQL Queries](../sql-advanced/intro.md)

**Metacharacter**: a character with special meaning in a regular expression, such as `^`, `$`, `*` or `[`. [Regular Expressions](../regular-expressions/intro.md)

**Mini-dimension**: a small dimension holding rapidly changing attributes, split off from a large dimension. [Advanced DW Design](../dw-design-advanced/intro.md)

**Name node / data node**: the HDFS master that holds the metadata, and the workers that store blocks. [Introduction to Hadoop](../hadoop/intro.md)

**Normal forms (1NF, 2NF, 3NF)**: a valid relation with single values and a key; plus no partial dependencies; plus no transitive dependencies. [Normalization](../normalization/intro.md)

**NULL**: the absence of a value. Test it with `IS NULL`, never `= NULL`. [SQL Queries](../sql-queries/intro.md)

## O to S

**OLTP / OLAP**: online transaction processing (running the business with many small transactions) and online analytical processing (analyzing it with aggregate queries). [Data Warehousing Concepts](../dw-concepts/intro.md)

**Operational data store (ODS)**: an integrated, frequently refreshed copy of operational data for near-real-time queries. [Data Warehousing Concepts](../dw-concepts/intro.md)

**Partial identifier**: an attribute that, together with borrowed keys, identifies instances of a weak or associative entity. [Databases and the ER Model](../er-model/intro.md)

**Primary key**: the column or columns that uniquely identify each row of a table; never NULL. [The Relational Model](../relational-model/intro.md)

**RDD**: resilient distributed dataset, Spark's fault-tolerant collection of records partitioned across a cluster. [Apache Spark](../spark/intro.md)

**Referential integrity**: the rule that every foreign key value matches an existing primary key (or is NULL). [The Relational Model](../relational-model/intro.md)

**Regular expression**: a pattern that describes combinations of characters, for searching, extracting and validating text. [Regular Expressions](../regular-expressions/intro.md)

**Relational model**: data organized as relations (tables) with keys, written in this book in text form with underlined primary keys. [The Relational Model](../relational-model/intro.md)

**Reverse engineering**: generating a data model diagram from an existing database. [Advanced SQL Queries](../sql-advanced/intro.md)

**Role-playing dimension**: one dimension used several times by a fact table in different roles, such as order date and ship date. [Advanced DW Design](../dw-design-advanced/intro.md)

**Schema on read / schema on write**: applying structure when data is read (Spark, Hadoop) or before it is loaded (relational databases). [Apache Spark](../spark/intro.md)

**Semi-additive fact**: a fact that can be summed across some dimensions but not all, such as an inventory level over time. [Advanced DW Design](../dw-design-advanced/intro.md)

**Slowly changing dimension (SCD)**: techniques for dimension attributes that change: Type 1 overwrite, Type 2 new row, Type 3 new column, Type 4 history table. [Advanced DW Design](../dw-design-advanced/intro.md)

**Snowflake schema**: a star schema whose dimensions are normalized into several tables. [Data Warehouse Design](../dw-design/intro.md)

**Staging area**: where extracted data is held while it is transformed. [ETL](../etl-procedure/intro.md)

**Star schema**: a fact table surrounded by denormalized dimension tables. [Data Warehouse Design](../dw-design/intro.md)

**Subquery**: a query inside another query; scalar (one value) or vector (a list), correlated or uncorrelated. [Advanced SQL Queries](../sql-advanced/intro.md)

**Superclass / subclass**: an entity class and its specialized subtypes, which inherit its attributes and relationships. [Databases and the ER Model](../er-model/intro.md)

**Surrogate key**: a meaningless integer key assigned by the warehouse to a dimension row. [Advanced DW Design](../dw-design-advanced/intro.md)

## T to Z

**Total / partial specialization**: whether every superclass instance must belong to a subclass (total, double line) or not (partial, single line). [Databases and the ER Model](../er-model/intro.md)

**Transaction**: a unit of work that succeeds or fails as a whole, ended by `COMMIT` or `ROLLBACK`. [SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md)

**Transformation (Spark)**: an operation that defines a new RDD or DataFrame from an existing one, such as `filter` or `groupBy`; lazily evaluated. [Apache Spark](../spark/intro.md)

**Uniqueness**: cardinality divided by total records. [Data Profiling](../data-profiling/intro.md)

**View**: a named query that behaves like a table; dynamic (recomputed on use) or materialized (stored). [Advanced SQL Queries](../sql-advanced/intro.md)

**Weak entity**: an entity that cannot exist without its owner and is identified by the owner's key plus a partial identifier. [Databases and the ER Model](../er-model/intro.md)

**Window function**: a function such as `RANK()` or `SUM() OVER (...)` that computes a value for each row from a set of related rows. [Advanced SQL Queries](../sql-advanced/intro.md)

**YARN**: Yet Another Resource Negotiator, Hadoop's cluster resource manager and job scheduler. [Introduction to Hadoop](../hadoop/intro.md)
