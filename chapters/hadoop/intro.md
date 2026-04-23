# Introduction to Hadoop

When a dataset outgrows a single machine — whether because it's too big to store, too big to process, or too big to process fast enough — you need a distributed system. **Apache Hadoop**, launched in 2006, was the first widely adopted open-source framework for doing this, and it shaped the vocabulary the whole big-data industry still uses.

## The five Vs of big data

A modern working definition:

| V | What it means |
|---|---|
| **Volume** | Data too large for a single server to store or process |
| **Variety** | Structured, semi-structured (JSON, logs), and unstructured (text, images, audio) |
| **Velocity** | Arrival rate; streaming vs. batch |
| **Veracity** | Trustworthiness — is this data correct? |
| **Value** | Does extracting insight from it pay back the infrastructure cost? |

Hadoop was built primarily for the first two.

## Hadoop's three core components

### 1. HDFS — the storage layer

**Hadoop Distributed File System** splits files into large blocks (default 128 MB) and stores each block on multiple nodes (default replication factor 3). Key actors:

- **NameNode** — maintains the file-system metadata: what files exist, which blocks make them up, where those blocks live
- **DataNodes** — actually store the blocks and serve reads/writes
- **Secondary NameNode** — keeps periodic snapshots of NameNode metadata (it is *not* a hot failover)

Why it works:
- **Write once, read many** — HDFS doesn't support random writes; it streams whole blocks
- **Replication** gives fault tolerance — lose a DataNode and the data is still readable elsewhere
- **Move compute to data** — the scheduler places tasks on the nodes that already hold the relevant blocks

### 2. MapReduce — the original compute model

A job splits into two phases:

1. **Map** — process each input record, emit zero or more `(key, value)` pairs
2. **Shuffle and sort** — group pairs by key across the cluster
3. **Reduce** — process each key's list of values, emit output

Classic word count:

```
map:    input line → for each word, emit (word, 1)
reduce: (word, [1,1,1,...]) → (word, sum)
```

MapReduce is heavyweight for interactive analytics — every step writes to HDFS. Spark (Chapter 21) largely replaced it for most workloads, but the pattern is foundational.

### 3. YARN — the resource manager

**YARN** (Yet Another Resource Negotiator) schedules workloads across the cluster:

- **ResourceManager** — cluster-wide scheduler
- **NodeManager** — per-node agent that runs containers
- **ApplicationMaster** — per-job coordinator that negotiates containers from ResourceManager

YARN lets MapReduce, Spark, Tez, Flink, and others share the same cluster.

## The Hadoop ecosystem

"Hadoop" in practice means a stack of adjacent projects:

- **Hive** — SQL-on-Hadoop; compiles queries to MapReduce / Tez / Spark
- **Pig** — dataflow scripting (mostly historical now)
- **HBase** — NoSQL column-family store on top of HDFS
- **Sqoop** — bulk transfer between relational DBs and HDFS
- **Flume / Kafka** — log / event ingestion
- **Oozie / Airflow** — workflow orchestration
- **Zookeeper** — distributed coordination (leader election, config)
- **Impala / Presto / Trino** — low-latency SQL engines on HDFS / object storage

## Modern context

Most enterprises no longer run their own Hadoop clusters. The two reasons:

1. **Cloud object storage + serverless compute** (S3/GCS + BigQuery/Athena/Databricks) gives you the same *logical* capability without cluster operations
2. **Spark on Kubernetes** handles the same workloads with less Hadoop-specific machinery

But the concepts — distributed storage with replication, data locality, shuffle costs, YARN-style resource management — are still the right mental model for the modern stack. Most "serverless" services are Hadoop-style systems with the operations hidden.

## When Hadoop (or its cloud successors) is the right tool

- Data volumes well beyond a single machine (10s–100s of TB and up)
- Primarily batch workloads — hourly, daily
- Mixed workload — SQL + custom code + ML training
- Data that is too messy or variable for a warehouse to ingest efficiently

## Learning outcomes

- Define the five Vs and map a concrete scenario to them
- Describe HDFS's block-replication model and the NameNode / DataNode roles
- Walk through a MapReduce job end-to-end
- Explain YARN's scheduler model
- Recognize the Hadoop ecosystem components and their cloud-native successors

