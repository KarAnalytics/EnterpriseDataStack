# Data Modeling with draw.io

Before you type your first `CREATE TABLE`, you design. **Data modeling** is the practice of diagramming the entities in a business, the attributes that describe them, and the relationships between them — so that the database you build is a faithful, queryable reflection of how the business actually works. [draw.io](https://app.diagrams.net) (a.k.a. diagrams.net) is a free, browser-based tool that makes this straightforward.

## Three levels of data model

| Level | Audience | Detail |
|---|---|---|
| **Conceptual** | Business stakeholders | Entities and high-level relationships only |
| **Logical** | Analysts, architects | Full attribute lists, keys, relationship cardinalities — DBMS-independent |
| **Physical** | DBAs, developers | Tables, columns with exact types, indexes, partitions — specific to a DBMS |

You typically iterate: conceptual with the business, logical with the team, physical at implementation time.

## Entity-Relationship (ER) diagrams

An ER diagram is the standard notation for logical models. Three building blocks:

- **Entity** — a thing the business cares about (Customer, Order, Product)
- **Attribute** — a property of an entity (Customer.email, Order.total_amount)
- **Relationship** — how entities are connected (Customer *places* Order)

Attributes can be:

- **Simple** — single-valued, indivisible (birth_date)
- **Composite** — broken into sub-parts (address → street, city, state, zip)
- **Derived** — calculable from others (age from birth_date)
- **Multi-valued** — can hold multiple values per entity (a customer's phone numbers)

## Cardinality and participation

Relationships have a cardinality — how many of each entity can be on each side:

- **One-to-one (1:1)** — a person has one passport
- **One-to-many (1:N)** — a customer has many orders
- **Many-to-many (M:N)** — an order contains many products; a product appears in many orders

Participation tells you whether the relationship is mandatory or optional:

- **Total (mandatory)** — every order *must* have a customer
- **Partial (optional)** — a customer *may* have placed no orders yet

Most ER tools (draw.io included) use **crow's-foot notation** to express both at once:

```
Customer ──┤├────< Order
   (1, mandatory)    (0..N)
```

## Resolving many-to-many

A many-to-many relationship can't be implemented directly in a relational schema; you split it into an **associative (junction) table**:

```
Order ──< OrderLine >── Product
```

`OrderLine` typically carries its own attributes (quantity, unit_price) and a composite key of `(order_id, product_id)`.

## Weak entities and identifying relationships

A **weak entity** can't exist without a related strong entity — think `OrderLine` without `Order`, or `Dependent` without `Employee`. Its primary key includes the owner's key.

## Supertypes and subtypes

When entities share attributes, model them as a supertype with subtypes (generalization / specialization). A `Person` supertype with `Employee` and `Customer` subtypes is the canonical example. Implementation options:

- **Single table** (all columns, nullable) — simple, wastes space
- **Table per subtype** — faithful, more joins
- **Table per concrete class** — fewer joins, duplicated supertype columns

## From ER to schema

A clean mechanical procedure:

1. Each entity becomes a table
2. Each simple attribute becomes a column
3. Each 1:N relationship adds a foreign key on the "many" side
4. Each M:N relationship becomes a junction table
5. Weak entities include the strong entity's key in their PK
6. Subtypes map to one of the three patterns above

## Working in draw.io

Practical tips:

- Start with the **ER shape library** — it has entity, attribute, and relationship shapes ready
- Use the **Mockup / Entity-Relation** templates to start faster
- Export to PNG or SVG for documentation; export to XML to version-control your diagrams alongside the code
- Use colors to separate transactional entities from reference entities at a glance

## Learning outcomes

- Describe the three levels of data model and when each is used
- Build an ER diagram with entities, attributes, and relationships in draw.io
- Express cardinality and participation correctly with crow's-foot notation
- Resolve many-to-many relationships and model weak entities
- Translate an ER model into a normalized relational schema

