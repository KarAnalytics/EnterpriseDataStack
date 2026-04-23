# SQL — DDL, DCL, and DML

SQL is the language of relational data. Before we can ask questions of a database (the job of `SELECT`), we need to create the structures that hold the data, control who can touch them, and change rows as the business evolves. Those three responsibilities are split across three sub-languages within SQL.

## The three sublanguages

| Sublanguage | Stands for | What it does | Representative commands |
|---|---|---|---|
| **DDL** | Data Definition Language | Defines and changes schema | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` |
| **DCL** | Data Control Language | Controls access to data | `GRANT`, `REVOKE` |
| **DML** | Data Manipulation Language | Changes data within tables | `INSERT`, `UPDATE`, `DELETE`, `MERGE` |

A fourth sublanguage, **TCL** (Transaction Control Language — `COMMIT`, `ROLLBACK`, `SAVEPOINT`), manages the atomicity of DML operations.

## DDL — defining schema

DDL statements describe the *shape* of the database: tables, columns, data types, primary and foreign keys, indexes, constraints, views.

```sql
CREATE TABLE customers (
    customer_id   INT PRIMARY KEY,
    first_name    VARCHAR(50) NOT NULL,
    last_name     VARCHAR(50) NOT NULL,
    email         VARCHAR(100) UNIQUE,
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Key ideas:

- **Data types** — `INT`, `VARCHAR(n)`, `DECIMAL(p, s)`, `DATE`, `TIMESTAMP`, `BOOLEAN`, `TEXT`. Choose the narrowest type that fits the value.
- **Constraints** — `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`. Enforced by the database, not the application.
- **`ALTER TABLE`** — add columns, change types, add constraints, rename. Changes to large tables can be expensive.
- **`DROP` vs `TRUNCATE` vs `DELETE`** — `DROP` removes the table definition entirely, `TRUNCATE` empties it but keeps the structure and is not logged row-by-row, `DELETE` is DML and removes matching rows transactionally.

## DCL — controlling access

DCL lets a database owner hand out or revoke permissions:

```sql
GRANT SELECT, INSERT ON customers TO analyst_role;
REVOKE INSERT ON customers FROM analyst_role;
```

In enterprise systems, permissions are almost always granted to **roles** rather than individual users. This is the day-to-day front line of the governance topic we covered in Chapter 2.

## DML — changing data

DML is how rows enter, change, and leave a table:

```sql
INSERT INTO customers (customer_id, first_name, last_name, email)
VALUES (1, 'Ada', 'Lovelace', 'ada@example.com');

UPDATE customers
SET    email = 'ada.lovelace@example.com'
WHERE  customer_id = 1;

DELETE FROM customers
WHERE  customer_id = 1;
```

Every DML statement runs inside a transaction. If your client is in autocommit mode, each statement commits immediately; if not, you need to `COMMIT` (or `ROLLBACK`) to finalize changes.

## Transactions — the ACID promise

Relational databases guarantee four properties for every transaction:

- **Atomicity** — all or nothing
- **Consistency** — constraints hold before and after
- **Isolation** — concurrent transactions don't see each other's partial work
- **Durability** — once committed, survives crashes

Understanding ACID is what lets you reason about what happens when two people update the same row at the same time.

## Learning outcomes

- Distinguish DDL, DCL, and DML statements and know when each is appropriate
- Write `CREATE TABLE` statements with appropriate types, keys, and constraints
- Modify schemas safely with `ALTER`
- Grant and revoke table-level permissions
- Insert, update, and delete rows; explain what a transaction guarantees

