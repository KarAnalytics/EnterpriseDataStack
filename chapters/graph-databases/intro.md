# Graph Databases

When your data is fundamentally a **network** — customers and products, devices and events, people and relationships, concepts and citations — the relational model starts to fight you. Joins pile up, queries get ugly, and performance suffers. Graph databases are built for exactly this shape of data: a schema of **nodes** connected by **relationships**, each carrying properties, queried with languages designed to traverse.

## When to use a graph database

Good fits:
- **Social networks** — friends of friends, influence paths
- **Fraud detection** — ring detection, suspicious patterns across accounts and devices
- **Recommendations** — "customers who bought X also bought Y" expressed as a traversal
- **Supply chain & bill-of-materials** — multi-level dependency graphs
- **Knowledge graphs** — entities and typed relationships that power enterprise search and RAG
- **Identity and access** — who-can-access-what across complex role hierarchies
- **Network / infrastructure** — hosts, services, dependencies

Bad fits: tabular analytics, large aggregations over scalar columns, simple CRUD on independent records — stay with relational or columnar.

## The property graph model

Two primary constructs:

- **Node** — an entity with a label and properties. `(:Customer {name: "Ada", tier: "gold"})`
- **Relationship** — a *directed*, *typed* edge between nodes, with its own properties. `(:Customer)-[:PURCHASED {date: "2025-01-03"}]->(:Product)`

A few contrasts with relational:

- Relationships are first-class — they have types and properties, not just foreign keys
- Traversals are O(1) per hop — a graph DB stores relationship pointers at each node
- Schema is flexible — you can add properties to individual nodes without altering the whole label

## Cypher — Neo4j's query language

Cypher is declarative and pattern-based. You draw ASCII-art patterns and the database finds matches.

```cypher
// Create nodes and a relationship
CREATE (c:Customer {id: 1, name: "Ada", tier: "gold"})
CREATE (p:Product  {sku: "A100", name: "Coffee Grinder"})
CREATE (c)-[:PURCHASED {date: date("2025-01-03"), qty: 1}]->(p);

// Find products purchased by gold-tier customers in January 2025
MATCH (c:Customer {tier: "gold"})-[r:PURCHASED]->(p:Product)
WHERE r.date >= date("2025-01-01") AND r.date < date("2025-02-01")
RETURN c.name, p.name, r.date
ORDER BY r.date;
```

A few Cypher patterns to know:

```cypher
-- Variable-length paths (friends of friends, 1 to 3 hops)
MATCH (me:Person {name: "Ada"})-[:FRIEND*1..3]-(other)
RETURN DISTINCT other.name;

-- Shortest path
MATCH (a:Airport {code: "LAX"}), (b:Airport {code: "JFK"})
MATCH path = shortestPath((a)-[:FLIES_TO*]-(b))
RETURN path;

-- Pattern existence check
MATCH (c:Customer)
WHERE EXISTS { (c)-[:PURCHASED]->(:Product {sku: "A100"}) }
RETURN c.name;
```

Alternative query languages you may encounter:

- **Gremlin** (Apache TinkerPop) — imperative, works across many graph DBs
- **SPARQL** — for RDF triple stores (the semantic web lineage)
- **GQL** (ISO-standard Graph Query Language, ratified 2024) — emerging universal standard

## Leading products

- **Neo4j** — the market leader; Cypher; Desktop, Aura (managed cloud), and Enterprise editions
- **Amazon Neptune** — managed; supports both property graphs (Gremlin, openCypher) and RDF (SPARQL)
- **TigerGraph** — parallel graph DB, GSQL query language
- **ArangoDB** — multi-model (graph + document + key-value)
- **JanusGraph** — open-source, distributed, backed by Cassandra/HBase

## Modeling — getting it right

Good graph models share common traits:

- **Nodes for nouns, relationships for verbs.** A `Purchase` is a verb ("customer *purchased* product"), so it's a relationship — not a node — unless the act of purchasing has its own rich attributes (line items, taxes, discounts) in which case model it as an `Order` node.
- **Direction matters.** A `FOLLOWS` relationship from Ada to Linus is not the same as the reverse — but you can query both directions with `-[:FOLLOWS]-` (undirected).
- **Properties on the right thing.** `purchase_date` goes on the `:PURCHASED` relationship, not on the product.
- **Labels over types.** A node can have multiple labels (`:Person:Customer:Employee`) — useful for mixed populations.

## Performance and scale

- **Index on identity-like properties** (`:Customer(id)`, `:Product(sku)`) so lookups are fast
- **Constraints** enforce uniqueness: `CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.id IS UNIQUE;`
- **Avoid deep unbounded traversals** — `*1..5` is usually fine, `*1..100` is often not
- **Shard by community** when scaling — nodes that talk to each other should live on the same partition

## Graphs and generative AI

Graph databases have seen a second wind through **Graph RAG** — combining a knowledge graph with a vector store so LLMs can retrieve both the relevant facts *and* the relationships between them. Chapter 10 of the companion AI book builds exactly this kind of system.

## Learning outcomes

- Decide whether a use case is a good fit for a graph database
- Model entities and relationships with labels, types, and properties
- Write basic Cypher queries including pattern matches, variable-length paths, and shortest paths
- Pick the right product for a given constraint (cost, scale, query language, cloud)
- Recognize the role of graph DBs in modern AI retrieval pipelines

