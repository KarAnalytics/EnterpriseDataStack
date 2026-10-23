# How to Use This Book

This book is designed to be practical, modular and useful long after a course ends, whether you read it cover to cover, look up one topic to solve a problem at work, or use it to review for an exam. Different readers will use it differently, and that is by design. This short chapter suggests approaches for different readers, explains what each chapter contains and what software you need, and lists resources for going deeper.

## For Students in BSAN 726 and 727

The book follows the course: its 22 chapters, in eight parts, follow the order of the course topics, and the practice exercises use the same tools you use in class. A good rhythm is:

1. **Before class or the module videos**, read the chapter to meet the vocabulary and the worked example.
2. **After class**, work through the practice questions. Try each one before you open its answer.
3. **When you prepare for an exam or a project checkpoint**, use the learning objectives at the top of each chapter as a checklist, and the key terms at the bottom as a self-test.
4. **After the course**, keep the book as your reference. Everything that was covered is here in one place.

The chapters are concise on purpose. They explain the concepts at the depth the course needs and point to textbooks for more (see [](#recommended-resources) below). The course slides and other materials remain on Canvas.

## For Readers Learning on Their Own

If you are exploring data management out of curiosity or for your career, read the parts in order. Each one builds on the ones before it: clean data first, then design databases, then warehouses, then query them with SQL, then move them with ETL, and finally look beyond relational databases at graphs and big data. Set up the free tools as each part begins, and do at least one practice question per chapter. Reading about a join is not the same as writing one.

## For Practitioners Solving Specific Problems

If you already work with data, use the book as a reference and jump straight to what you need:

- Inherited a messy spreadsheet? Start with [Data Processing](../data-processing/intro.md), [OpenRefine](../openrefine/intro.md) and [Data Profiling](../data-profiling/intro.md).
- Designing a new database? Read Part III, from [Databases and the ER Model](../er-model/intro.md) to [Normalization](../normalization/intro.md).
- Building reporting or analytics? Part IV covers dimensional modeling and star schemas, with [examples from many industries](../dw-examples/intro.md).
- Writing a tricky query? [SQL Queries](../sql-queries/intro.md) and [Advanced SQL Queries](../sql-advanced/intro.md).
- Loading a warehouse? Part VI walks through a complete ETL example.
- Working with networks, or with data too big for one machine? Parts VII and VIII.

## What Each Chapter Contains

- **Learning objectives**: what you should be able to do after the chapter.
- **Concepts**: definitions and key ideas.
- **A worked example**: one complete example, start to finish.
- **Common mistakes**: the errors people make most often.
- **Check the AI**: how a GenAI assistant typically goes wrong on the topic, and how to verify its answers.
- **Practice**: exercises in free tools, with answers you can reveal when you are ready.
- **Key terms** and **further reading**.

## Software You'll Need

You don't need everything on day one; set each tool up when its part begins. All of them are free.

- **Oracle Cloud** with an Always Free Autonomous Database, for all SQL. Everything runs in the browser-based SQL Worksheet, so there is nothing to install. See [Setting up Oracle Cloud](../appendix/oracle-cloud.md).
- **OpenRefine**, a desktop tool that runs in your browser, for data processing and profiling (Part II).
- **Python** or **R** (for example in Google Colab or RStudio), for data profiling and regular expressions (Part II).
- **draw.io** (diagrams.net), in the browser, for ER diagrams (Part III).
- **Neo4j** (AuraDB Free or Neo4j Desktop), for graph databases (Part VII).
- **Google Cloud Platform**, for Hadoop and Spark (Part VIII).
- A **terminal**: macOS and Linux have one built in; on Windows, use Windows Terminal or Git Bash (Part I).

## Conventions

- Inline code like `SELECT * FROM SH.CUSTOMERS` is something you type.
- Code blocks are complete snippets you can run:
  ```sql
  SELECT channel_id, SUM(amount_sold) AS total_sales
  FROM SH.SALES
  GROUP BY channel_id;
  ```
- Entity classes and tables appear in CAPITALS, and column names in `monospace`.
- Relational models are written in text form, with primary keys underlined, for example **ORDER** ({u}`OrderID`, OrderDate, CustomerID) FK (CustomerID) ref CUSTOMER.
- Terms are in **bold** where they are defined.

## A Word of Caution: Stay Grounded

GenAI assistants are now part of every data professional's toolkit. They write SQL, suggest table designs, clean data and explain error messages, and the course encourages you to use them. But they are statistical systems: they produce answers that *look* right, and they do not know your business rules unless you state them. A query that runs is not necessarily a query that is correct, and a confident normalization can quietly keep a redundant column.

That is why every chapter has a **Check the AI** box. The habit to build is simple: before you accept an answer, ask *does this make sense given what I know about the data and the business?* Reconcile totals with their source, read each relationship in both directions, and check a few rows by hand. The final line of defense is always you: the person who understands the problem, can tell whether an answer makes sense, and takes responsibility for the decisions made with it.

(recommended-resources)=
## Recommended Textbooks and Resources

These resources go deeper than this book. The first two are the course's reference texts.

**Textbooks:**

- Hoffer, J. A., Ramesh, V., & Topi, H. *Modern Database Management* (14th ed.). Pearson. The standard text for ER and EER modeling, logical design, normalization and SQL (Parts III and V).
- Kimball, R., & Ross, M. (2013). *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling* (3rd ed.). Wiley. The reference for dimensional design, with case studies across industries (Part IV).
- Robinson, I., Webber, J., & Eifrem, E. (2015). *Graph Databases* (2nd ed.). O'Reilly Media. A practical introduction to property graphs and Cypher (Part VII).
- White, T. (2015). *Hadoop: The Definitive Guide* (4th ed.). O'Reilly Media. HDFS, MapReduce and YARN in depth (Part VIII).
- Chambers, B., & Zaharia, M. (2018). *Spark: The Definitive Guide*. O'Reilly Media. Spark's structured APIs, Spark SQL and more (Part VIII).
- DAMA International. (2017). *DAMA-DMBOK: Data Management Body of Knowledge* (2nd ed.). Technics Publications. The professional reference for data governance and data management practice (Part II).

**Documentation and online resources:**

- Oracle. *Oracle Database SQL Language Reference*. https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/ The authoritative reference for every SQL statement used in this book.
- Kimball Group. *Dimensional Modeling Techniques*. https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/ Short, precise definitions of every dimensional modeling technique.
- OpenRefine. *User Manual*. https://openrefine.org/docs
- Neo4j. *Cypher Manual*. https://neo4j.com/docs/cypher-manual/current/
- Apache Spark. *Documentation*. https://spark.apache.org/docs/latest/

Each resource complements this book from a different angle: the textbooks give theoretical depth, the documentation gives precise syntax, and this book ties them together in the order a data project actually runs.

## Feedback

Corrections and suggestions are welcome. Open an issue on [GitHub](https://github.com/KarAnalytics/EnterpriseDataStack/issues) or email the author.
