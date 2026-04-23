# Apache Spark

**Apache Spark** is the de facto compute engine for big data. Where MapReduce wrote every intermediate step to disk, Spark holds data in memory across the cluster, runs 10–100× faster on many workloads, and exposes a clean API in Python, Scala, Java, R, and SQL. Most modern "big data" and ML pipelines — in companies of any size — run on Spark.

## Why Spark beat MapReduce

- **In-memory computation** — intermediate results stay in RAM across stages
- **Rich API** — functional transforms (`map`, `filter`, `reduceByKey`, `join`) compose cleanly
- **Lazy evaluation and DAG optimization** — Spark plans the whole query before executing, then runs an optimized plan
- **Unified engine** — batch, streaming, SQL, ML, and graph processing share the same runtime
- **Much faster development** — a PySpark job is 10–20 lines; the MapReduce equivalent is hundreds

## Core abstractions

### RDD — Resilient Distributed Dataset

The original Spark abstraction: a distributed, immutable collection of objects. You still encounter RDDs in lower-level code, but for most work the higher-level DataFrame / Dataset APIs are preferred.

### DataFrame

A distributed table with a schema — conceptually the same shape as a pandas DataFrame or a SQL table, but operating across the cluster. The preferred API for most analytics workloads.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum as _sum

spark = SparkSession.builder.appName("sales").getOrCreate()

sales = spark.read.parquet("gs://my-bucket/fact_sales/")

top_customers = (
    sales
      .filter(col("order_date") >= "2025-01-01")
      .groupBy("customer_sk")
      .agg(_sum("extended_price").alias("lifetime_value"))
      .orderBy(col("lifetime_value").desc())
      .limit(10)
)

top_customers.show()
```

### Dataset

Strongly typed variant of DataFrame, available in Scala and Java (not Python). Worth knowing about; most coursework stays in PySpark.

### Spark SQL

You can do everything above in SQL:

```python
sales.createOrReplaceTempView("sales")

spark.sql("""
    SELECT customer_sk, SUM(extended_price) AS lifetime_value
    FROM   sales
    WHERE  order_date >= DATE '2025-01-01'
    GROUP  BY customer_sk
    ORDER  BY lifetime_value DESC
    LIMIT  10
""").show()
```

## Transformations vs. actions

- **Transformations** (`select`, `filter`, `join`, `groupBy`) are **lazy** — they build a plan but don't execute
- **Actions** (`count`, `collect`, `show`, `write`) **trigger execution** — Spark runs the DAG up to and including the action

This model is why `spark.read.parquet(...).filter(...).select(...)` doesn't do anything until you call `.show()` or `.count()` — the planner folds the filter into the read and skips columns you don't select.

## The execution model

A Spark job breaks down as:

- **Job** → triggered by an action
- **Stages** → separated by shuffles (wide dependencies)
- **Tasks** → one per partition per stage, run in parallel on executors

Key terms:

- **Driver** — coordinates the job, holds the DAG
- **Executor** — JVM process on a worker node that runs tasks
- **Partition** — a chunk of a DataFrame; one task per partition
- **Shuffle** — reorganizing data across the network (by key, for joins and group-bys) — the most expensive operation; minimize it

## Performance basics

- **Prefer DataFrames over RDDs** — the Catalyst optimizer rewrites your plan
- **Broadcast small dimension tables** in joins: `spark.sql.autoBroadcastJoinThreshold` (or explicit `broadcast()`)
- **Partition-prune** — partition writes by date so reads only touch relevant days
- **Use Parquet/ORC** — columnar, compressed, predicate-pushdown enabled
- **Cache sparingly** — `df.cache()` helps when the same DF is used multiple times, hurts when it isn't
- **Watch shuffles** — look for them in the Spark UI; consider repartitioning or bucketing

## Spark's other libraries

- **Spark Streaming / Structured Streaming** — real-time data processing using the same DataFrame API
- **MLlib** — distributed machine learning (classification, regression, clustering, recommendation)
- **GraphX / GraphFrames** — graph analytics on Spark
- **Koalas / pandas-on-Spark** — pandas API backed by Spark for very large data

## Running Spark

- **Locally** — `pip install pyspark`, then `pyspark` in a terminal. Great for development with sample data.
- **Dataproc** — Chapter 20 covers submitting Spark jobs to a managed cluster
- **Databricks** — the commercial distribution; notebooks, scheduling, Delta Lake, Unity Catalog
- **AWS EMR / Azure Synapse / HDInsight** — cloud-native managed Spark
- **Spark on Kubernetes** — self-hosted modern deployment pattern

## Spark and the warehouse

Spark is increasingly the engine *inside* modern data platforms:

- **Delta Lake** (Databricks) — ACID tables on object storage, Spark-native
- **Apache Iceberg** — open table format, Spark-native
- **dbt on Spark** — run dbt transformations against a Spark backend

The line between "warehouse" and "Spark cluster" is blurring.

## Learning outcomes

- Explain why Spark replaced MapReduce for most big-data workloads
- Read, transform, and write DataFrames in PySpark
- Use the SQL API interchangeably with the DataFrame API
- Distinguish transformations from actions and reason about lazy execution
- Identify the main cost drivers (shuffles, wide dependencies) and the standard fixes
- Choose a Spark runtime environment for a given scenario

