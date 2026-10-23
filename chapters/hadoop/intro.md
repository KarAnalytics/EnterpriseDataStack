# Introduction to Hadoop

Every minute, the world sends millions of messages, searches, payments and sensor readings. Data at that scale does not fit on one computer, and buying an ever-bigger computer stops being affordable long before the data stops growing. **Hadoop** made a different approach mainstream: spread both the data and the processing across many ordinary computers working in parallel. This chapter introduces big data, the Hadoop framework and its three components (HDFS, MapReduce and YARN), and the command line used to work with HDFS. The [next chapter](../spark/intro.md) covers Apache Spark, which builds on these ideas.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Define big data and its five Vs, and explain horizontal versus vertical scaling.
2. Describe Hadoop's components: HDFS, MapReduce and YARN.
3. Contrast Hadoop with a relational DBMS, and HDFS with a local file system.
4. Explain the roles of the name node and data nodes, blocks and replication.
5. Use basic HDFS commands, and trace a MapReduce computation.
:::

## Big data

**Big data** describes large volumes of data, both structured and unstructured, that flow into a business every day, at a scale and speed that traditional systems cannot handle. It is usually characterized by **five Vs**:

| V | Meaning | Example |
|---|---|---|
| **Volume** | the amount of data, data "at rest" | petabytes of transaction history |
| **Velocity** | the speed at which data arrives and must be processed, data "in motion" | clickstreams, sensor readings, payments |
| **Variety** | the range of data types and sources, data in "many forms" | tables, text, images, logs, video |
| **Veracity** | the uncertainty of the data, data "in doubt" | duplicated, missing or contradictory records |
| **Value** | the total cost of ownership and the return on investment | is the data worth storing and processing? |

### Scaling up or scaling out

There are two ways to handle more data:

- **Vertical scaling** (scaling up): buy a bigger, faster machine, such as a mainframe or a specialized data warehouse appliance. Simple, but costs rise steeply, and there is always a largest machine.
- **Horizontal scaling** (scaling out): connect many **commodity** computers (ordinary, inexpensive servers) and make them work in parallel. Capacity grows by adding machines.

Hadoop is built for horizontal scaling. It applies **divide and conquer**: split the data into pieces, process the pieces in parallel on different machines, and combine the results. The same model serves ETL, general data storage, business intelligence, statistical analysis, optimization, and machine learning.

## Hadoop

**Hadoop** is an open-source software framework for distributed storage and processing of very large datasets on clusters of commodity hardware. It grew from two ideas published by Google: a **distributed file system** that spreads files across many machines while tolerating failures, and **MapReduce**, a programming model whose programs can be parallelized and executed across a cluster automatically. Yahoo! built an open-source implementation and released it through the Apache Software Foundation as **Apache Hadoop** in 2008. Many vendors offer their own Hadoop stacks, among them Amazon, Cloudera, Google, IBM, Microsoft and Oracle.

Hadoop has three core components:

- **HDFS** (Hadoop Distributed File System) stores any type of file, split into blocks that are replicated across machines as they are written, which makes storage resilient and highly available. Hadoop handles this automatically.
- **MapReduce** is the distributed processing model.
- **YARN** (Yet Another Resource Negotiator) manages the cluster's processing resources, schedules jobs, and runs processing frameworks (MapReduce, and others such as Spark).

### Hadoop versus a relational DBMS

| Hadoop | Relational DBMS (OLTP) |
|---|---|
| data is loaded and appended, like a warehouse; updating records in place is not supported | supports all CRUD operations |
| schema applied when data is read (**schema on read**) | schema defined before data is loaded (**schema on write**) |
| scales to petabytes and more | typically gigabytes to terabytes on one system |
| batch reading of data | batch and interactive reading |
| data duplicated across nodes for fault tolerance | each record stored once |

Hadoop does not replace the relational databases and warehouses of the earlier parts of this book; it complements them for data that is too large, too varied or too raw for them.

## HDFS

**HDFS** is a distributed file system that stores data across a cluster of commodity machines, with a strong emphasis on fault tolerance.

- Files are split into **blocks**, typically 64 MB or 128 MB each.
- Each block is **replicated** (three copies by default) on different machines, so a failed disk or server loses nothing.
- An HDFS cluster has one **name node** and many **data nodes**.

**The name node** (the master) holds all the metadata about stored files: which blocks make up each file and which data nodes hold each block. It manages incoming file system operations and maps blocks to data nodes. Name nodes need a lot of memory.

**The data nodes** (the workers) handle read and write requests, and create, delete and replicate blocks on their disks. They continuously report to the name node and ask it for instructions. Data nodes need a lot of disk storage. Both need fast network connections.


:::{figure} images/hdfs_architecture.png
:label: fig-hdfs-architecture
:alt: A client asks the name node where a file is. Below, two racks each hold two data nodes; blocks B1, B2 and B3 are each stored on three data nodes across both racks.
HDFS: the name node holds the metadata; data nodes hold replicated blocks. Source: {download}`hdfs_architecture.drawio <images/hdfs_architecture.drawio>`.
:::

Physically, nodes are mounted in **racks**, a **cluster** is a collection of racks, and racks are housed in **data centers**. Google, for example, operates dozens of data centers in the United States and well over a hundred worldwide. HDFS is rack-aware: it places replicas of a block on different racks, so a whole rack can fail without losing data.

### HDFS versus a local file system

| HDFS | Local file system |
|---|---|
| a global view: a file looks the same from every node | a local view, specific to one computer |
| supports distributed processing of blocks | stores each file in one place |
| survives disk failure through replicated blocks | corrupted or deleted data is lost |
| scales by adding data nodes | limited by one server's capacity |
| cost-effective on commodity hardware | large-scale storage and processing on one machine is expensive |

### HDFS commands

HDFS is used from the command line of a cluster node, with commands that mirror the Linux commands of [EDM Foundations](../edm-foundations/intro.md), prefixed by `hadoop fs`:

| Command | Purpose |
|---|---|
| `hadoop fs -ls /user` | list files and directories |
| `hadoop fs -mkdir /user/sales` | create a directory |
| `hadoop fs -copyFromLocal file.csv /user/sales` | copy a file from the local file system into HDFS (also `-put`) |
| `hadoop fs -copyToLocal /user/sales/file.csv ./` | copy a file from HDFS to the local file system (also `-get`) |
| `hadoop fs -cat /user/sales/file.csv` | view a file's contents |
| `hadoop fs -cp` / `-mv` | copy or move files within HDFS |
| `hadoop fs -rm file` / `-rm -r dir` | delete a file, or a directory and its contents |
| `hadoop fs -du` | disk space used |
| `hadoop fs -stat file` | a file's size, owner and other details |
| `hadoop fs -touchz file` | create an empty file |
| `hadoop fs -test -e file` | check whether a file exists (`-z`: is empty; `-d`: is a directory) |
| `hadoop fs -chown` / `-chgrp` | change a file's owner or group |
| `hadoop fs -expunge` | empty the HDFS trash |

### A short HDFS session

On the master node of a cloud Hadoop cluster (such as Google Cloud Dataproc, Amazon EMR or Azure HDInsight), with a CSV file first uploaded to your home directory on that node:

```bash
hadoop fs -ls /                                  # the root of HDFS
hadoop fs -mkdir /user/handson1                  # a working directory in HDFS
hadoop fs -copyFromLocal /home/<username>/FortuneGlobal500_2024.csv /user/handson1
hadoop fs -ls /user/handson1                     # the file is now in HDFS

mkdir temp                                       # a local folder (not HDFS)
hadoop fs -copyToLocal /user/handson1/FortuneGlobal500_2024.csv /home/<username>/temp/

hadoop fs -mkdir /user/handson2
hadoop fs -cp /user/handson1/FortuneGlobal500_2024.csv /user/handson2
hadoop fs -rm -r /user/handson2                  # remove the directory and its contents
```

Notice the two file systems: `mkdir` and `ls` act on the node's local disk, and `hadoop fs` commands act on HDFS. Moving data between them is an explicit step.

## MapReduce

**MapReduce** is the programming model, popularized by Google and implemented in Hadoop, that processes data in parallel with an emphasis on scalability and fault tolerance. A job has two main phases:

1. **Map**: each worker applies a mapper function to its own blocks of data, turning each record into key-value pairs.
2. **Shuffle and sort**: the framework groups all pairs with the same key together and sends each group to one worker.
3. **Reduce**: each worker applies a reducer function to its groups, combining the values for each key into a result.

The classic example counts words in a huge collection of documents:


:::{figure} images/mapreduce_wordcount.png
:label: fig-mapreduce
:alt: Two input splits are mapped to (word, 1) pairs, shuffled into groups by word, and reduced to counts: big 2, data 3, is 2.
Counting words with MapReduce. Source: {download}`mapreduce_wordcount.drawio <images/mapreduce_wordcount.drawio>`.
:::

Each mapper sees only its own block, and each reducer only its own keys, so the work spreads across as many machines as are available. If a machine fails, its piece of the work is simply rerun elsewhere.

The idea comes from functional programming, and Python has the same two building blocks:

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

list(map(lambda x: x * x, numbers))        # map: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
reduce(lambda x, y: x + y, numbers)        # reduce to a sum: 55
reduce(lambda x, y: x * y, numbers)        # reduce to a product: 3628800
```

## YARN

**YARN** distributes a job, such as a MapReduce program, across the nodes of the cluster and coordinates it. Three of its services:

- **ResourceManager**: the global service that receives applications and allocates the cluster's resources to them.
- **NodeManager**: runs on every node, launching tasks and overseeing the node's resource use.
- **JobHistoryServer**: keeps a log of finished jobs.

Because YARN is general, frameworks other than MapReduce, including Spark, can run on the same Hadoop cluster.

## Running Hadoop

Managing a cluster of name nodes and data nodes is a job in itself. **Administration tools** such as Cloudera Manager install, configure, manage and monitor the Hadoop stack through a graphical interface.

Hadoop is available as:

- **Distributions** to install on your own hardware: Cloudera's (which absorbed Hortonworks) and, historically, MapR.
- **Managed cloud services** that create a cluster in minutes and delete it when the job is done: Amazon EMR (Elastic MapReduce), Google Cloud Dataproc, Microsoft Azure HDInsight, and Oracle's and IBM's cloud big data services.

Cloud services have made short-lived clusters the norm: create a cluster, run the job, save the results to cloud storage, and delete the cluster so you stop paying for it.

## Common mistakes

- **Confusing the local file system with HDFS** on a cluster node.
- **Forgetting to delete a cloud cluster** after use. Clusters bill by the minute.
- **Using Hadoop for small data.** A file that fits comfortably in a laptop's memory is faster to process with Python, R or SQL.
- **Expecting database behavior**: HDFS does not support updating records in place, and MapReduce jobs are batch, not interactive.

:::{warning} Check the AI
GenAI assistants are helpful for the command line, but they cannot see your cluster. They mix up local and HDFS paths (`/home/...` versus `/user/...`), suggest `hdfs dfs` and `hadoop fs` interchangeably (usually fine) along with flags that your version does not support, and propose `-rm -r` on paths without checking what is there. Run `hadoop fs -ls` on any path before you delete it, and remember to delete cloud resources, which an assistant will rarely remind you to do.
:::

## Practice

**1. The five Vs.** Which V does each describe? (a) A bank processes 20,000 card payments per second. (b) A hospital combines lab results, scanned forms and physicians' notes. (c) Sensor readings are often duplicated or missing. (d) A retailer stores 40 petabytes of history.

:::{note} Answer
:class: dropdown
(a) Velocity. (b) Variety. (c) Veracity. (d) Volume.
:::

**2. Blocks.** A 1 GB file is stored in HDFS with 128 MB blocks and the default replication factor of 3. How many blocks does the file have, how many block copies are stored, and how much raw disk space does it use?

:::{note} Answer
:class: dropdown
1,024 MB / 128 MB = 8 blocks; 8 × 3 = 24 block copies; about 3 GB of raw disk space.
:::

**3. Trace MapReduce.** Trace a MapReduce job that computes total sales by store from these records, split across two mappers: mapper 1 gets `(S1, 20), (S2, 5), (S1, 10)`; mapper 2 gets `(S2, 15), (S3, 8)`. Show the map output, the shuffle groups and the reduce output.

:::{note} Answer
:class: dropdown
Map output: the records are already key-value pairs (store, amount). Shuffle: S1: [20, 10]; S2: [5, 15]; S3: [8]. Reduce (sum): S1: 30, S2: 20, S3: 8.
:::

**4. Two file systems.** On a cluster node, you run `ls` and see `sales.csv`, but `hadoop fs -ls /user/data` does not list it. Why, and what command fixes it?

:::{note} Answer
:class: dropdown
`ls` shows the node's local file system, and `hadoop fs -ls` shows HDFS, a separate file system. Copy the file in: `hadoop fs -copyFromLocal sales.csv /user/data`.
:::

## Key terms

big data, volume, velocity, variety, veracity, value, vertical scaling, horizontal scaling, commodity hardware, divide and conquer, Hadoop, HDFS, block, replication, name node, data node, rack, cluster, data center, MapReduce, map, shuffle, reduce, YARN, ResourceManager, NodeManager, JobHistoryServer, schema on read, Cloudera Manager, Amazon EMR, Google Cloud Dataproc, Azure HDInsight

## Further reading

- Apache Hadoop, [HDFS Architecture](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html) and [FileSystem Shell](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html).
- White, T. (2015). *Hadoop: The Definitive Guide* (4th ed.). O'Reilly Media.
