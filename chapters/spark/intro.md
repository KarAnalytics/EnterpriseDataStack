# Apache Spark

MapReduce made big data processing possible, but every step writes its results to disk, which makes multi-step and interactive work slow. **Apache Spark** keeps data in memory across a cluster and offers a much richer and simpler programming model, including SQL. It has become the most widely used engine for large-scale data processing. This chapter explains Spark's architecture and core ideas, and works through a hands-on session with PySpark and Spark SQL.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Explain what Spark adds to the Hadoop model, and name its main components.
2. Describe resilient distributed datasets (RDDs), transformations, actions and lazy evaluation.
3. Explain DataFrames and Spark SQL, and how schema on read affects data types.
4. Read a file into a Spark DataFrame, inspect it, and query it with the DataFrame API and Spark SQL.
:::

## What Spark is

**Apache Spark** is an open-source alternative to MapReduce. It is built around a data structure called the **resilient distributed dataset (RDD)**: a collection of records partitioned across the machines of a cluster and maintained in a fault-tolerant way. Because RDDs can stay in memory, Spark handles **iterative** programs (which pass over the same data many times, as machine learning algorithms do) and **interactive, exploratory** work far better than MapReduce. For such workloads it can be orders of magnitude faster, and big data vendors adopted it rapidly.

Spark fits into the Hadoop ecosystem: it can read from HDFS and run on a cluster manager such as YARN ([Introduction to Hadoop](../hadoop/intro.md)), but it also runs on its own, on Kubernetes, or on a single laptop.

### Components

| Component | Purpose |
|---|---|
| **Spark Core** | the foundation: task scheduling, memory management, and the RDD API, usable from Java, Python, Scala and R |
| **Spark SQL** | DataFrames and a full SQL query engine |
| **MLlib** | machine learning: classification, regression, clustering and recommender systems |
| **Spark Streaming** (Structured Streaming) | processing data streams as they arrive |
| **GraphX** | graph processing |


:::{figure} images/spark_stack.png
:label: fig-spark-stack
:alt: A layered stack: Spark SQL and DataFrames, MLlib, Structured Streaming and GraphX on top of Spark Core, which runs on a cluster manager over storage.
The Spark stack. Source: {download}`spark_stack.drawio <images/spark_stack.drawio>`.
:::

## Spark Core: RDDs

Once data is loaded into an RDD, two kinds of operations can be performed:

- **Transformations** create a new RDD from an existing one (filter rows, map a function over records, join, group). The original is not changed.
- **Actions** compute a result from an RDD and return it (count the rows, collect results, save to a file).

Transformations are **lazily evaluated**: Spark only records them, and runs them when a later action needs their result. A chain of transformations is compiled into a **directed acyclic graph (DAG)** of steps, which Spark optimizes and then spreads over the cluster. RDDs are kept in memory as long as possible, so later steps can reuse them without reading from disk.

Lazy evaluation has a practical consequence: an error in a transformation (a misspelled column, say) may only appear when an action runs, several lines later.

## Spark SQL and DataFrames

The RDD API is easier than writing MapReduce programs, but **Spark SQL** is easier still. It runs on top of Spark Core and introduces the **DataFrame**: a distributed collection of data organized into named columns, like a database table. DataFrames can be created from RDDs by specifying a schema, or loaded directly from files (CSV, JSON, Parquet and others) and databases. Behind the scenes they still use RDDs, but users work with columns and rows.

Spark implements a full SQL query engine that converts SQL statements into a series of RDD transformations and actions. The **DataFrame API** (method calls such as `filter` and `groupBy`) and **Spark SQL** (SQL strings) are two ways of writing the same thing, and Spark turns both into the same execution plan.

MLlib was originally built on RDDs; its current version works directly with DataFrames.

### Schema on read

A relational database applies **schema on write**: the table and its column types are defined before any data is loaded ([SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md)). Spark applies **schema on read**: it reads a file directly, and the structure is decided at the moment of reading, either from options you give (or a schema you supply) or by Spark's defaults. That flexibility is what makes Spark good at raw data, and it puts the responsibility for data types on you: always check the types Spark assigned before you sort, compare or calculate.

## Where to run Spark

- **Google Colab**: a free notebook in the browser where PySpark runs on a single machine. The code is the same code that runs on a large cluster, which makes Colab ideal for learning.
- **Databricks**: a cloud platform built on Spark by Spark's creators. Its free edition provides notebooks attached to a real Spark environment, with extensive learning material.
- **A Hadoop cluster in the cloud**, such as Google Cloud Dataproc or Amazon EMR: run `pyspark` on the master node for an interactive Python shell with Spark.
- **Your own computer**: `pip install pyspark`, for development with sample data.

## Worked example: PySpark in Colab

This session reads the Fortune Global 500 file ([Data Processing](../data-processing/intro.md)) into Spark and queries it with both the DataFrame API and Spark SQL. Run the cells in order in a Colab notebook.

**1. Start Spark.** Colab usually has PySpark installed; install it if not. A **SparkSession** is the entry point to Spark: every DataFrame and every SQL query goes through it.

```python
import importlib.util, subprocess, sys
if importlib.util.find_spec("pyspark") is None:
    subprocess.run([sys.executable, "-m", "pip", "install", "-q", "pyspark"], check=True)

from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = SparkSession.builder.appName("Fortune500").getOrCreate()
```

**2. Get the file.** Spark reads files from the session's storage (or cloud storage on a real cluster), not from web addresses, so download a local copy first:

```python
import os, urllib.request

URL = "https://github.com/KarAnalytics/datasets/raw/refs/heads/master/FortuneGlobal500_2024.csv"
FILE = "FortuneGlobal500_2024.csv"
if not os.path.exists(FILE):
    urllib.request.urlretrieve(URL, FILE)
```

**3. Read it into a DataFrame.** `header=True` takes column names from the first line; the encoding option matches the file's Windows/Latin-1 encoding, so that *Düsseldorf* and *São Paulo* are read correctly.

```python
fortune = spark.read.csv(FILE, header=True, encoding="ISO-8859-1")

print(fortune.count(), len(fortune.columns))     # 500 rows, 13 columns
fortune.show(5)                                  # the first five rows
fortune.printSchema()                            # the type Spark assigned to each column
```

`count()` and `show()` are actions: they make Spark actually read the data. **Look carefully at `printSchema()`**: which type did Spark give the numeric columns, given that you did not supply a schema? What would happen if you sorted or compared them? (The [Spark documentation](https://spark.apache.org/docs/latest/sql-data-sources-csv.html) lists the CSV reader's options.)

**4. Summarize.** `describe()` returns the count, mean, standard deviation, minimum and maximum of each column:

```python
fortune.select("Rank", "Company", "RevenueMil").describe().show()
```

**5. Group and count with the DataFrame API.** The equivalent of `GROUP BY HqCountry` with `COUNT(*)`, sorted by the count:

```python
(fortune.groupBy("HqCountry")
        .count()
        .orderBy(F.desc("count"))
        .show(10))

fortune.select(F.countDistinct("HqCountry")).show()       # 35 countries
```

The United States (140 companies) and China (127) together account for more than half of the Global 500.

**6. Query with Spark SQL.** Register the DataFrame as a **temporary view**, which gives it a name SQL can use. No data is copied, and the view disappears when the session ends.

```python
fortune.createOrReplaceTempView("FortuneCompanies")

spark.sql("""
    SELECT IndustrySector, COUNT(*) AS Companies
    FROM   FortuneCompanies
    WHERE  HqCountry = 'United States'
    GROUP  BY IndustrySector
    ORDER  BY Companies DESC, IndustrySector
    LIMIT  3
""").show(truncate=False)
```

The result is Commercial Banks (11), then Petroleum Refining and Pharmaceuticals (6 each). It is the same SQL you would write in Oracle, with one difference in setup: in Oracle, the table had to be created and loaded first. Here Spark reads the file directly, and on a real cluster the query would be split across many machines.

**7. The same query, two ways.** These return the same 140 rows:

```python
us_sql = spark.sql("SELECT Company, RevenueMil FROM FortuneCompanies WHERE HqCountry = 'United States'")
us_api = fortune.filter(F.col("HqCountry") == "United States").select("Company", "RevenueMil")
print(us_sql.count(), us_api.count())             # 140 140
```

**8. Stop the session** when you are done, to release its memory: `spark.stop()`.

## Common mistakes

- **Trusting the default types.** Without a schema (or schema inference), CSV columns are read as text, so numbers sort and compare as strings. Check `printSchema()` every time.
- **Calling `show()` and expecting a DataFrame back.** `show()` only displays; it returns nothing.
- **Collecting a big DataFrame to the driver** with `collect()` or `toPandas()`, which tries to move all the data onto one machine.
- **Forgetting lazy evaluation**: a transformation that "worked" may fail when the first action runs.
- **Reading a web address directly** with `spark.read.csv`: download the file, or put it in cloud storage, first.

:::{warning} Check the AI
GenAI assistants write PySpark readily, and often mix it with pandas: `df.head()` with pandas expectations, `df['col'].sum()`, `inplace=True`, or `.apply()` with a Python function, which either fails or silently becomes slow. They also tend to omit the reading options (header, encoding, schema) that determine whether the data is right in the first place. For any generated Spark code, check `printSchema()` and a few rows after reading, and compare one result (a count or a total) with a number you know.
:::

## Practice

These exercises continue the Colab session above.

**1. Concepts.** Classify each operation as a transformation or an action: `filter`, `count`, `groupBy`, `show`, `select`, `write.csv`, `orderBy`.

:::{note} Answer
:class: dropdown
Transformations: `filter`, `groupBy`, `select`, `orderBy`. Actions: `count`, `show`, `write.csv`.
:::

**2. DataFrame API.** Which industry sectors have the most Global 500 companies headquartered in Japan? Show the top three.

:::{note} Answer
:class: dropdown
```python
(fortune.filter(F.col("HqCountry") == "Japan")
        .groupBy("IndustrySector").count()
        .orderBy(F.desc("count"))
        .show(3, truncate=False))
```
Motor Vehicles & Parts (8), Trading (6), and Electronics, Electrical Equipment (4).
:::

**3. Spark SQL.** List the companies headquartered in Germany with their headquarters city. How many are there?

:::{note} Answer
:class: dropdown
```python
germany = spark.sql("""
    SELECT Company, Hqcity
    FROM   FortuneCompanies
    WHERE  HqCountry = 'Germany'
""")
germany.show(truncate=False)
germany.count()          # 30
```
:::

**4. Types.** Read the file again, this time with `inferSchema=True`, and compare `printSchema()` with your first read. Then sort the companies by revenue, highest first, with each version. Explain any difference in the order.

## Key terms

Apache Spark, resilient distributed dataset (RDD), partition, Spark Core, Spark SQL, DataFrame, MLlib, Spark Streaming, GraphX, transformation, action, lazy evaluation, directed acyclic graph (DAG), in-memory processing, schema on read, schema on write, SparkSession, PySpark, temporary view, Databricks

## Further reading

- Apache Spark, [Spark SQL, DataFrames and Datasets Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html).
- Chambers, B., & Zaharia, M. (2018). *Spark: The Definitive Guide*. O'Reilly Media.
