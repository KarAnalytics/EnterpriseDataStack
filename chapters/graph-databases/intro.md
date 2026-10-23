# Graph Databases

Some data is mostly about connections: who knows whom, which country trades with which, which customers bought the same products, which accounts sent money to which. Relational databases can store connections, but every hop across them costs a join. A **graph database** stores the connections themselves, so following them is fast however large the database grows. This chapter introduces property graphs, when to use them, the Neo4j database and its query language **Cypher**, with a worked example on a network of international trade.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Describe the labeled property graph model: nodes, relationships, labels and properties.
2. Explain when a graph database outperforms a relational one, and when it does not.
3. Read and write Cypher queries with `MATCH`, `WHERE`, `RETURN`, aggregation, `WITH` and `COLLECT`.
4. Load nodes and relationships from CSV files into Neo4j and verify the load.
:::

## What a graph database is

A **graph database** is a database with an explicit graph structure. Data is stored as:

- **Nodes**: the things (a customer, a country, a product). A node can have one or more **labels** saying what kind of thing it is (`:Customer`, `:Supplier`), and a company can be both an employer and a vendor.
- **Relationships** (edges): the connections between nodes, each with a **type** (`:PURCHASED`, `:TRADES_WITH`) and a direction.
- **Properties**: key-value attributes on nodes *and* on relationships (a customer's name; a purchase's date and amount).


:::{figure} images/property_graph.png
:label: fig-property-graph
:alt: A small graph: a Customer node with name and city placed an Order node, which contains two Product nodes with quantities; a Supplier node supplies one product.
A labeled property graph: properties live on nodes and on relationships. Source: {download}`property_graph.drawio <images/property_graph.drawio>`.
:::

This is the **labeled property graph** model. Its defining feature is that each node knows its adjacent nodes directly. As the number of nodes grows, the cost of one local step (a **hop**) from a node to its neighbors stays the same, because no index lookup or table scan is needed to find them.

### Relationships in a relational database

In a relational database, connections are stored as foreign keys, often with a lookup (associative) table in between. To answer *which customers bought product XYZ?* from CUSTOMERS, ORDERS, ORDER_LINES and PRODUCTS takes three joins:

```sql
SELECT DISTINCT c.customer_name
FROM   customers c
JOIN   orders o       ON o.customer_id = c.customer_id
JOIN   order_lines ol ON ol.order_id   = o.order_id
JOIN   products p     ON p.product_id  = ol.product_id
WHERE  p.product_name = 'XYZ';
```

In a graph database the same question is a pattern:

```cypher
MATCH (c:Customer)-[:PLACED]->(:Order)-[:CONTAINS]->(p:Product {name: 'XYZ'})
RETURN DISTINCT c.name;
```

For one or two hops the difference is modest. For questions that chain many hops ("friends of friends of friends", "suppliers of suppliers", "money that moved through four accounts") joins multiply and slow down, while graph traversal stays fast.

### When to use a graph database, and when not

**Use a graph database** for rapid traversal: queries across many nodes and relationships, such as recommendations, fraud rings, supply chain dependencies, network analysis and knowledge graphs. Unlike many other NoSQL databases (such as the document stores MongoDB and Couchbase), Neo4j supports ACID transactions like a relational database, so it can also handle transactional workloads on connected data.

**Do not use a graph database** when queries span the entire database: aggregate reporting and OLAP across all records is what relational warehouses do best.

A sensible architecture uses each for what it does best: a relational DBMS for OLTP, a data warehouse for OLAP, and a graph database alongside them for highly connected queries and graph analytics.

## Neo4j

**Neo4j** is the most widely used graph database. It is available as:

- **Neo4j Sandbox**: a temporary database in the cloud, blank or pre-loaded with an example dataset, used through the browser. Nothing to install; a sandbox lives for a few days, and you can save your Cypher scripts for reuse.
- **Neo4j AuraDB**: the managed cloud service, with a free tier.
- **Neo4j Desktop**: a free application for Windows, macOS and Linux to create and manage local databases and projects.

Neo4j also publishes **GraphGists**, worked examples of graph models for many business problems, and built-in guides, including a **Northwind** tutorial that loads the database of [ETL Example: Northwind](../etl-northwind/intro.md) as a graph. Comparing its Cypher with the SQL you already know is an excellent exercise.

## Cypher basics

**Cypher** is Neo4j's query language. Where SQL describes tables to join, Cypher describes **patterns** to find, drawn with ASCII art: nodes in parentheses, relationships in square brackets with arrows.

```
MATCH (first node)-[relationship]->(second node)
WHERE <optional filter conditions>
RETURN <what to show>
```

`MATCH` plays the role of SQL's `SELECT ... FROM`. The examples in this section use Neo4j's built-in **Movies** example graph (`:Person` and `:Movie` nodes, connected by `:ACTED_IN` and `:DIRECTED`).

### Matching nodes

```cypher
MATCH (n) RETURN n;                              // every node in the graph
MATCH (p:Person) RETURN p;                       // every node labeled Person

MATCH (m:Movie {released: 2003}) RETURN m;       // filter on a property inside the pattern

MATCH (p:Person) WHERE p.born = 1965             // the same kind of filter, SQL style
RETURN p.name, p.born;                           // return properties instead of whole nodes
```

`//` starts a comment.

### Relationships

```cypher
()            // a node
()--()        // two nodes with a relationship of any type, in either direction
()-->()       // the first node has a relationship to the second
()<--()       // the second node has a relationship to the first
```

```cypher
// All people who acted in The Matrix, with the nodes and relationships
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie {title: 'The Matrix'})
RETURN p, rel, m;
```

### Patterns across several hops

The real strength of Cypher is chaining patterns:

```cypher
// Actors, the movies they acted in, and the directors of those movies
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
RETURN a.name, m.title, d.name;

// Actor-director pairs, how often they worked together, and on which movies
MATCH (actor:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(director:Person)
RETURN actor.name, director.name,
       count(m) AS collaborations, collect(m.title) AS movies
ORDER BY collaborations DESC;
```

Aggregation works like SQL's, but without `GROUP BY`: Cypher groups by every returned value that is not an aggregate. `collect()` gathers values into a list.

### WITH

`WITH` passes intermediate results from one part of a query to the next, like a subquery or a `WITH` clause in SQL. It is how Cypher expresses "not in": collect a set first, then filter against it (see the worked example).

## Worked example: the international trade network

This network describes trade between countries in 2017, built from the MIT Media Lab's Observatory of Economic Complexity. **Nodes** are countries; **relationships** are trade between them. Each relationship from country A to country B carries two properties: `TotalExportValue`, what A exported to B, and `TotalImportValue`, what A imported from B (in US dollars). The data is on GitHub at `KarAnalytics/InternationalTrade2017`.

### Loading the data

**Nodes**: one per row of the country file. `MERGE` creates the node only if it does not exist, and `SET` adds properties, converting text to numbers where needed.

```cypher
LOAD CSV WITH HEADERS FROM
  'https://raw.githubusercontent.com/KarAnalytics/InternationalTrade2017/main/country_names_with_info_and_latlong.csv' AS row
WITH row WHERE row.Id IS NOT NULL
MERGE (c:Country {ID: row.Id})
SET c.Country = row.Country,
    c.Continent = row.Continent,
    c.Population = toInteger(row.Population);
```

**Relationships**: one per row of the edge file. Each row finds its two country nodes, then creates a `:TRADE` relationship between them.

```cypher
LOAD CSV WITH HEADERS FROM
  'https://raw.githubusercontent.com/KarAnalytics/InternationalTrade2017/main/edges_trade_2017_simple.csv' AS row
MATCH (c1:Country {ID: row.Source})
MATCH (c2:Country {ID: row.Target})
CREATE (c1)-[:TRADE {YearOfTrade: toInteger(row.Year),
                     TotalExportValue: toFloat(row.export_val),
                     TotalImportValue: toFloat(row.import_val)}]->(c2);
```

### Verifying the load

Always check that what was loaded matches the files:

```cypher
MATCH (c:Country) RETURN count(c);                       // 242 countries
MATCH ()-[r:TRADE]->() RETURN count(r);                  // 27,464 relationships, one per edge-file row
MATCH (c:Country) RETURN c LIMIT 5;                      // look at a few nodes
CALL db.propertyKeys();                                  // the property names stored
MATCH p = ()-[r:TRADE]->() RETURN p LIMIT 5;             // look at a few relationships
```

Graph visualizations of a whole network this dense are not very readable; query for the subset you need instead.

To start over, delete everything. A node cannot be deleted while it has relationships, so `DETACH DELETE` removes the relationships first:

```cypher
MATCH (n) DETACH DELETE n;
```


:::{figure} images/trade_top12.png
:label: fig-trade-top12
:alt: A circular network of the 12 largest exporting countries, colored by continent, with node size showing total exports and line width showing trade between each pair. China, Germany and the United States are the largest nodes.
A small part of the trade network: the 12 largest exporters and the trade among them, drawn with Python (networkx).
:::

### Profiling the graph

Two facts about this dataset matter for every query:

- **Every pair is stored in both directions.** For each country pair there is a relationship A → B *and* B → A, and A's exports to B equal B's imports from A. Count relationships in one direction only.
- **Many relationships have a value of zero**: 3,475 of the 27,464 have no exports at all. A relationship means "a pair exists in the data", not "they traded". To count real trading partners, filter on the value.
- **Coverage is incomplete.** 27 of the 242 countries, including Belgium, Luxembourg and Taiwan, have no relationships at all in this subset. That reflects how the source data was assembled, not real-world trade. Never conclude "Belgium does not trade" from it.

### Queries

**Total imports of China**, in billions of dollars:

```cypher
MATCH (c1:Country)-[r:TRADE]->(c2:Country)
WHERE c1.Country = 'China'
RETURN ROUND(sum(r.TotalImportValue) / 1000000000, 2) AS china_imports_bil;    // 1346.6
```

**Export statistics across all relationships:**

```cypher
MATCH ()-[r:TRADE]->()
RETURN round(min(r.TotalExportValue), 2),
       round(avg(r.TotalExportValue), 2),
       round(max(r.TotalExportValue), 2);
```

The average is about $536 million but the median is only about $1.4 million: a few enormous flows (the largest, China to the United States, is $444 billion) dominate.

**The largest exporters:**

```cypher
MATCH (c1:Country)-[r:TRADE]->()
RETURN c1.Country, ROUND(sum(r.TotalExportValue) / 1000000000, 2) AS exports_bil
ORDER BY exports_bil DESC
LIMIT 10;
```

China leads ($2,304.77 billion), followed by Germany ($1,278.04 billion).

**Germany's trading partners, counted correctly:**

```cypher
MATCH (c1:Country {Country: 'Germany'})-[r:TRADE]->(c2:Country)
RETURN count(r) AS relationships,
       count(CASE WHEN r.TotalExportValue > 0 THEN 1 END) AS export_partners;    // 212 and 209
```

**Countries that import from Germany but not from France**, using `WITH` and `COLLECT` to build the "France" set first:

```cypher
MATCH (:Country {Country: 'France'})-[r:TRADE]->(c:Country)
WHERE r.TotalExportValue > 0
WITH collect(DISTINCT c) AS france_importers
MATCH (:Country {Country: 'Germany'})-[r2:TRADE]->(c2:Country)
WHERE r2.TotalExportValue > 0 AND NOT c2 IN france_importers
RETURN c2.Country;
```

The answer is France itself (France does not import from France) and Norfolk Island. Without the `> 0` filters, the answer would be different, and wrong.

## Common mistakes

- **Counting relationships as if they were trades, friendships or purchases**, without checking for zero-valued or placeholder relationships.
- **Ignoring direction**, or double counting in data stored in both directions.
- **`CREATE` where `MERGE` was needed**, creating duplicate nodes when a load is run twice.
- **Forgetting to convert types**: `LOAD CSV` reads every value as text, so `toInteger` and `toFloat` are needed before comparing or summing.
- **Using a graph database for whole-database aggregate reporting**, which a warehouse does better.

:::{warning} Check the AI
GenAI assistants write Cypher fluently, but they cannot see your graph's labels, relationship types and property names, so they guess, for example `:TRADES_WITH` instead of `:TRADE`, or `:Nation` instead of `:Country`. In Cypher, a name that does not exist does not cause an error; the pattern simply matches nothing, and the query returns 0 or an empty result that looks like a real answer. They also tend to count relationships without asking whether a relationship means what the question means (here, a relationship with zero value is not a trade). Before trusting a generated query, run `CALL db.propertyKeys()` and `CALL db.labels()`, check that every name in the query exists, and test it on a case whose answer you know.
:::

## Practice

These exercises use the trade network loaded in your own Neo4j Sandbox or AuraDB instance.

**1. Read a pattern.** In words, what does this return? `MATCH (a:Country)-[:TRADE]->(b:Country)-[:TRADE]->(c:Country) WHERE a.Country = 'Kenya' RETURN count(DISTINCT c)`

:::{note} Answer
:class: dropdown
The number of distinct countries two hops from Kenya: countries that have a trade relationship with some country that has a trade relationship with Kenya. (Because of the zero-valued relationships, it overstates real trade links unless you filter on value.)
:::

**2. A simple query.** Find Kenya's three largest import sources by value, in billions of dollars.

:::{note} Answer
:class: dropdown
```cypher
MATCH (k:Country {Country: 'Kenya'})-[r:TRADE]->(c:Country)
RETURN c.Country, ROUND(r.TotalImportValue / 1000000000, 2) AS imports_bil
ORDER BY imports_bil DESC
LIMIT 3;
```
China ($4.06 billion), India ($1.69 billion) and the United Arab Emirates ($1.27 billion).
:::

**3. Continents.** Count the countries on each continent, largest first.

:::{note} Answer
:class: dropdown
```cypher
MATCH (c:Country)
RETURN c.Continent, count(c) AS countries
ORDER BY countries DESC;
```
Africa (56), Asia (51) and Europe (49) lead. Profiling the result also shows inconsistent labels, such as "America" and "Australia" alongside "North America" and "Oceania".
:::

**4. SQL versus Cypher.** Load the Northwind tutorial graph in a new sandbox. Write the Cypher and the SQL for "which customers bought products from the Seafood category?", and count the joins the SQL needs.

## Key terms

graph database, labeled property graph, node, label, relationship, relationship type, property, hop, traversal, Neo4j (Sandbox, AuraDB, Desktop), GraphGist, Cypher, MATCH, WHERE, RETURN, pattern, direction, collect, WITH, LOAD CSV, MERGE, CREATE, DETACH DELETE, db.propertyKeys

## Further reading

- Neo4j, [Cypher Manual](https://neo4j.com/docs/cypher-manual/current/) and [Getting Started](https://neo4j.com/docs/getting-started/).
- Robinson, I., Webber, J., & Eifrem, E. (2015). *Graph Databases* (2nd ed.). O'Reilly Media.
