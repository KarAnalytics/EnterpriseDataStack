# SQL: DDL, DCL and DML

**SQL** (Structured Query Language, often pronounced "sequel") is the language used to communicate with a relational database. With it you create, retrieve, update and delete data: the CRUD operations. This chapter covers the SQL that *builds and maintains* a database: defining tables and constraints (DDL), controlling who can use them (DCL), changing their contents (DML), and the transactions that keep changes safe. The running example implements the animal shelter design from [The Relational Model](../relational-model/intro.md) in Oracle.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Name the types of SQL commands and give examples of each.
2. Choose Oracle data types for columns.
3. Create, alter and drop tables with primary key, foreign key, NOT NULL, UNIQUE, CHECK and DEFAULT constraints, and choose a referential integrity rule.
4. Insert, update and delete rows safely.
5. Grant and revoke privileges, and explain transactions, COMMIT, ROLLBACK and the ACID properties.
:::

## Types of SQL commands

A database is described by a **catalog**: a set of **schemas**. A schema contains the objects created by one user (tables, views, constraints). In Oracle Cloud, the tables you create live in your `ADMIN` schema, and Oracle's sample tables live in schemas such as `SH`.

SQL commands fall into groups:

| Group | Purpose | Commands |
|---|---|---|
| **DDL**, data definition language | define the database: create, change and remove tables and constraints | `CREATE`, `ALTER`, `DROP` |
| **DML**, data manipulation language | maintain and query the data | `INSERT`, `UPDATE`, `DELETE`, `SELECT` |
| **DCL**, data control language | control access to the data | `GRANT`, `REVOKE` |
| **TCL**, transaction control language | make changes permanent or undo them | `COMMIT`, `ROLLBACK` |

TCL commands are often grouped with DCL. `SELECT` is part of DML, but it is so central that the next two chapters are devoted to it.

## Oracle data types

| Kind | Data type | Notes |
|---|---|---|
| Fixed-length text | `CHAR(n)` | always n characters, padded with spaces; good for codes like state abbreviations |
| Variable-length text | `VARCHAR2(n)` | up to n characters; the usual choice for text |
| Large text | `CLOB` | character large object, for long documents |
| Numbers | `NUMBER(p,s)` | p = **precision** (total digits), s = **scale** (digits after the decimal point); `NUMBER(8,2)` holds up to 999999.99 |
| Whole numbers | `INTEGER` | equivalent to `NUMBER(38)` |
| Date | `DATE` | a date *and* time to the second; write literals as `DATE '2026-03-14'` |
| Timestamp | `TIMESTAMP` | a date and time with fractions of a second |

Examples of choosing types:

| Value | Example | Type |
|---|---|---|
| Street number | 1654 | `INTEGER` (or `VARCHAR2` if numbers like "12B" occur) |
| Street name | Naismith Dr. | `VARCHAR2(250)` |
| City | Lawrence | `VARCHAR2(50)` |
| State | KS | `CHAR(2)` |
| Zip code | 66045 | `VARCHAR2(10)`: it is a code, not a quantity, and may have a leading zero or a +4 extension |
| Order total | 43.11 | `NUMBER(10,2)` |
| Order date and time | 2026-10-11 15:01:02 | `TIMESTAMP` (or `DATE`) |
| Birth date | 1997-04-22 | `DATE` |
| Preferred customer | true | `CHAR(1)` with a check for `'Y'`/`'N'` (the `BOOLEAN` type exists only in recent Oracle versions) |

The full list is in Oracle's [SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Data-Types.html).

### Case and comments

SQL **keywords and unquoted names are case-insensitive** in Oracle: `select * from customers` and `SELECT * FROM CUSTOMERS` are the same query. **Data values are case-sensitive**, however: `WHERE city = 'paris'` does not match `'Paris'`. Whether data should be case-sensitive is a business decision (standardize it during [data processing](../data-processing/intro.md)).

```sql
-- A double hyphen starts a comment that runs to the end of the line.
/* A slash-star comment
   can span several lines. */
```

## DDL: creating tables

```sql
CREATE TABLE table_name (
    column_name  data_type  [DEFAULT value]  [column_constraint],
    column_name  data_type  [DEFAULT value]  [column_constraint],
    ...
    [, table_constraint, ...]
);
```

Before writing a `CREATE TABLE`, decide for each column:

- the **data type**, with its length, precision and scale;
- whether it may be **NULL**;
- whether its values must be **unique**;
- whether it is part of the **primary key** or a **foreign key**;
- whether it has a **default value**;
- whether it needs a tighter rule than its type allows (a **check**).

### Worked example: the shelter database

The relational model of the shelter ([The Relational Model](../relational-model/intro.md)) becomes these tables. Parent tables are created before the tables that reference them.


:::{figure} ../relational-model/images/shelter_relational.png
:label: fig-ddl-shelter
:alt: Four shelter tables with their keys: ADOPTER, ADOPTION, ANIMAL and VET_VISIT, with arrows from foreign keys to the tables they reference.
Part of the shelter design being implemented: arrows point from each foreign key to the table it references.
:::

```sql
CREATE TABLE adopter (
    adopter_id   NUMBER(6)     CONSTRAINT adopter_pk PRIMARY KEY,
    name         VARCHAR2(60)  NOT NULL,
    street       VARCHAR2(100),
    city         VARCHAR2(50),
    state        CHAR(2),
    zip          VARCHAR2(10)
);

CREATE TABLE animal (
    animal_id     NUMBER(6)     CONSTRAINT animal_pk PRIMARY KEY,
    name          VARCHAR2(40),
    species       VARCHAR2(20)  NOT NULL,
    date_of_birth DATE,
    intake_date   DATE          DEFAULT SYSDATE NOT NULL,
    CONSTRAINT animal_species_ck CHECK (species IN ('Dog', 'Cat', 'Rabbit', 'Bird', 'Other'))
);

CREATE TABLE adoption (
    adoption_id   NUMBER(8)     CONSTRAINT adoption_pk PRIMARY KEY,
    adoption_date DATE          NOT NULL,
    fee           NUMBER(7,2)   CONSTRAINT adoption_fee_ck CHECK (fee >= 0),
    animal_id     NUMBER(6)     NOT NULL,
    adopter_id    NUMBER(6)     NOT NULL,
    CONSTRAINT adoption_animal_fk  FOREIGN KEY (animal_id)  REFERENCES animal (animal_id),
    CONSTRAINT adoption_adopter_fk FOREIGN KEY (adopter_id) REFERENCES adopter (adopter_id)
);

CREATE TABLE vet_visit (
    animal_id   NUMBER(6),
    visit_no    NUMBER(3),
    visit_date  DATE          NOT NULL,
    reason      VARCHAR2(200),
    cost        NUMBER(8,2),
    CONSTRAINT vet_visit_pk PRIMARY KEY (animal_id, visit_no),
    CONSTRAINT vet_visit_animal_fk FOREIGN KEY (animal_id)
        REFERENCES animal (animal_id) ON DELETE CASCADE
);
```

Things to notice:

- A primary key can be declared **on the column** (`adopter_pk`) or **as a table constraint** (`vet_visit_pk`); a composite key must be a table constraint.
- **Name every constraint.** Constraint names must be unique in the schema, and a name like `adoption_fee_ck` makes error messages readable.
- A **foreign key** column must have the same data type as the key it references.
- `DEFAULT SYSDATE` fills the intake date with today's date when none is given.
- `CHECK` constraints enforce business rules that the data type cannot.

### Referential integrity rules

What should happen to child rows when a parent row is deleted? Three common rules:

| Rule | On deleting a parent row | Oracle syntax |
|---|---|---|
| **Restrict** (Oracle's default) | refused if child rows exist | (no clause) |
| **Cascade** | the child rows are deleted too | `ON DELETE CASCADE` |
| **Set null** | the child rows' foreign key is set to NULL | `ON DELETE SET NULL` |

Vet visits are a weak entity: they mean nothing without their animal, so they **cascade**. Adoptions are business records and must not disappear silently, so they **restrict**: an animal with adoptions cannot be deleted. Set null fits optional relationships, such as a volunteer's mentor leaving the organization.

The same dependency governs dropping tables: Oracle will not drop a table that other tables reference. Drop the children first (`vet_visit`, `adoption`), then the parents.

### Changing and removing tables

```sql
DESCRIBE adopter;                                   -- show the current structure

ALTER TABLE adopter ADD (email VARCHAR2(100));      -- add a column
ALTER TABLE adopter MODIFY (email VARCHAR2(150));   -- change its type or size
ALTER TABLE adopter ADD CONSTRAINT adopter_email_uq UNIQUE (email);
ALTER TABLE adopter DROP (email);                   -- remove a column

DROP TABLE vet_visit;                               -- remove a table and its data
```

Some changes are refused when data exists: Oracle will not shrink a column below the size of the values already in it, or make a column `NOT NULL` while it contains NULLs.

## DML: changing data

### INSERT

```sql
-- Values for every column, in the order the table defines them
INSERT INTO adopter VALUES (501, 'Jordan Patel', '121 Helen St', 'Topeka', 'KS', '66603');

-- Values for named columns, in any order; the others get NULL or their default
INSERT INTO animal (animal_id, name, species, date_of_birth)
VALUES (1001, 'Biscuit', 'Dog', DATE '2023-05-10');

INSERT INTO adoption VALUES (70001, DATE '2026-03-14', 125.00, 1001, 501);
```

The second form is safer: it keeps working if columns are added to the table later.

### UPDATE

```sql
UPDATE adoption
SET    fee = 100.00
WHERE  adoption_id = 70001;
```

**An UPDATE without WHERE changes every row.** `UPDATE adoption SET fee = 100` sets the fee of every adoption ever recorded. Write the `WHERE` first, and test it with a `SELECT` using the same condition.

### DELETE

```sql
DELETE FROM vet_visit
WHERE  visit_date < DATE '2020-01-01';
```

The same warning applies: `DELETE FROM vet_visit` with no `WHERE` empties the table.

### Loading data in bulk

Inserting rows one at a time is fine for examples; real data is loaded in bulk. Create the table first, then load the rows:

- with the **Data Load** tool in Oracle Database Actions (upload a CSV or Excel file into a new or existing table);
- with **SQL\*Loader** or the `DBMS_CLOUD` package for large files;
- from a program in Python, R or Java that connects to the database and inserts the rows.

Always check the loaded data afterward: count the rows, and look for values that were truncated or mis-parsed (see [Data Profiling](../data-profiling/intro.md)).

## DCL: controlling access

Controlling who can see and change data is one of the main advantages of a database over files.

```sql
GRANT privilege ON table_name TO user_or_role;
REVOKE privilege ON table_name FROM user_or_role;
```

Table-level privileges include `SELECT`, `INSERT`, `UPDATE`, `DELETE` and `REFERENCES` (permission to create a foreign key to the table).

```sql
GRANT SELECT ON animal TO PUBLIC;              -- everyone may read the animal list
REVOKE SELECT ON animal FROM PUBLIC;           -- withdraw it

GRANT SELECT, INSERT ON vet_visit TO vet_clinic;      -- the clinic's account can read and add visits
GRANT UPDATE (fee) ON adoption TO finance_team;       -- finance may change only the fee column
```

Depending on your career path you may use DCL every day, or never: in many organizations a database administrator manages privileges, following the policies of [data governance](../data-governance/intro.md).

## Transactions

A **transaction** is a unit of work that must succeed or fail as a whole: recording an adoption might insert an adoption row *and* update the animal's status. A relational DBMS guarantees four properties, known as **ACID**:

| Property | Meaning | In a $1,000 bank transfer from account A to account B |
|---|---|---|
| **Atomicity** | all of the transaction happens, or none of it | the debit from A and the credit to B both happen, or neither does |
| **Consistency** | the database moves from one valid state to another, obeying every rule and constraint | if account B is frozen, or A lacks the funds, the transfer is rejected |
| **Isolation** | other users do not see a transaction's changes until it is complete | no report run during the transfer sees $1,000 missing from A but not yet in B |
| **Durability** | once committed, changes are permanent | after the bank confirms the transfer, a power failure does not undo it |

Two **transaction control** commands end a transaction:

- `COMMIT` makes the transaction's changes permanent and visible to others.
- `ROLLBACK` undoes every change since the last `COMMIT`.

DDL and DCL statements commit automatically: you cannot roll back a `CREATE TABLE` or a `GRANT`.

:::{note} In the course's Oracle Cloud worksheet
The Oracle Cloud SQL Worksheet used in this course commits each statement automatically, so every successful `INSERT`, `UPDATE` or `DELETE` is permanent as soon as it runs, and `ROLLBACK` cannot undo it. Check the `WHERE` clause of every `UPDATE` and `DELETE` (run it as a `SELECT` first) before you execute it. In a database tool with autocommit turned off, `ROLLBACK` works as described above.
:::

## SQL coding practices

- **Split statements over several lines**, one clause per line, and indent continuation lines.
- Write **SQL keywords in uppercase** (`CREATE`, `SELECT`) and names in lowercase or mixed case.
- Choose **meaningful, specific names**: `customer_id`, not `id` or `var1`; `purchase_tx_no`, not `sales_purchase_transaction_number`.
- **No spaces in names**: use underscores (`customer_name`) or capitals (`CustomerName`).
- **Comment** what each statement is for.
- Save your scripts as `.sql` files in a known folder, so you can rerun them.

## Common mistakes

- **UPDATE or DELETE without WHERE.**
- **Creating a child table before its parent**, or dropping a parent before its children.
- **Numbers that do not fit their type**: `NUMBER(4,2)` holds at most 99.99, so 143.11 is rejected.
- **Storing codes as numbers**: zip codes and phone numbers lose leading zeros.
- **Unnamed constraints**, which give error messages such as `SYS_C0012345` that nobody can interpret.
- **Comparing text in the wrong case**: `'paris'` is not `'Paris'`.

:::{warning} Check the AI
GenAI assistants write DDL quickly, but often in the wrong dialect. Watch for MySQL or PostgreSQL syntax in an Oracle script: `AUTO_INCREMENT` or `SERIAL` (Oracle uses `GENERATED AS IDENTITY` or a sequence), `VARCHAR` instead of `VARCHAR2`, `BOOLEAN` columns on older versions, `ON UPDATE CASCADE` (Oracle has no such clause), or `LIMIT`. Also check the logic: does every foreign key have the right delete rule for the business, and is every `UPDATE` and `DELETE` restricted by a `WHERE` clause? Run generated DDL on a test schema before your real one.
:::

## Practice

These exercises use your own schema in Oracle Cloud.

**1. Data types.** Choose an Oracle data type for: (a) a product SKU such as `KS-0042`; (b) a unit price up to $99,999.99; (c) a yes/no flag for gift wrap; (d) the time a support ticket was opened, to the second; (e) a product description of several paragraphs.

:::{note} Answer
:class: dropdown
(a) `VARCHAR2(10)`. (b) `NUMBER(7,2)`. (c) `CHAR(1)` with `CHECK (gift_wrap IN ('Y','N'))`. (d) `TIMESTAMP` (or `DATE`). (e) `CLOB` (or `VARCHAR2(4000)` if it is never longer).
:::

**2. Create and constrain.** Create the shelter's `volunteer` table: a numeric ID (primary key), a name (required), an email (unique), a start date (default today), and a `mentor_id` that refers to another volunteer. If a mentor's row is deleted, the mentees should remain, with no mentor.

:::{note} Answer
:class: dropdown
```sql
CREATE TABLE volunteer (
    volunteer_id NUMBER(6)     CONSTRAINT volunteer_pk PRIMARY KEY,
    name         VARCHAR2(60)  NOT NULL,
    email        VARCHAR2(100) CONSTRAINT volunteer_email_uq UNIQUE,
    start_date   DATE          DEFAULT SYSDATE,
    mentor_id    NUMBER(6),
    CONSTRAINT volunteer_mentor_fk FOREIGN KEY (mentor_id)
        REFERENCES volunteer (volunteer_id) ON DELETE SET NULL
);
```
:::

**3. Constraints at work.** Create the four shelter tables from the worked example and insert the sample rows. Then try each statement below and explain the result: (a) insert an adoption for animal 9999; (b) insert an animal with species `'Horse'`; (c) delete animal 1001; (d) drop table `animal`.

:::{note} Answer
:class: dropdown
(a) Rejected: the foreign key finds no parent (Oracle error ORA-02291). (b) Rejected by the check constraint `animal_species_ck` (ORA-02290). (c) Rejected while adoption 70001 refers to it: the restrict rule (ORA-02292); its vet visits would have cascaded. (d) Rejected because other tables' foreign keys reference it (ORA-02449); drop `vet_visit` and `adoption` first.
:::

**4. Safe updates.** Write an `UPDATE` that raises every adoption fee for dogs by 10%, and the `SELECT` you would run first to see which rows it will change.

:::{note} One possible answer
:class: dropdown
```sql
SELECT a.adoption_id, a.fee
FROM   adoption a
WHERE  a.animal_id IN (SELECT animal_id FROM animal WHERE species = 'Dog');

UPDATE adoption
SET    fee = fee * 1.10
WHERE  animal_id IN (SELECT animal_id FROM animal WHERE species = 'Dog');
```
:::

## Key terms

SQL, CRUD, catalog, schema, DDL, DML, DCL, TCL, data type, precision, scale, constraint, primary key, foreign key, NOT NULL, UNIQUE, CHECK, DEFAULT, referential integrity rule (restrict, cascade, set null), CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, bulk load, GRANT, REVOKE, privilege, transaction, COMMIT, ROLLBACK, autocommit, ACID (atomicity, consistency, isolation, durability)

## Further reading

- Oracle, [SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/).
- Hoffer, Ramesh and Topi, *Modern Database Management*, chapter "Introduction to SQL".
