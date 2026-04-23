# Setting Up a Dataproc Cluster on GCP

Running your own Hadoop / Spark cluster on physical hardware takes weeks of setup and a dedicated operations team. **Google Cloud Dataproc** collapses that into a few minutes and a single `gcloud` command. In this chapter we stand up a managed cluster, submit a job, and tear it down — the modern workflow for big-data experiments.

## What Dataproc is

Dataproc is GCP's managed Hadoop + Spark service. It provisions a cluster of Google Compute Engine VMs, installs and configures the Hadoop ecosystem (HDFS, YARN, Hive, Spark, Pig), and lets you submit jobs through the console, the `gcloud` CLI, or a REST API.

Why use it for a course:

- **No hardware required** — you pay per-second for running clusters and shut them down when done
- **Familiar stack** — standard Apache distributions, not a custom fork
- **Tight integration with GCS** — you can treat `gs://bucket/path` as if it were `hdfs:///path`
- **Ephemeral clusters are encouraged** — create a cluster for a job, run it, delete the cluster

## Prerequisites

1. A GCP account with a billing account attached (students typically get free credits)
2. A GCP project
3. The `gcloud` CLI installed locally, authenticated (`gcloud auth login`) and pointing at your project (`gcloud config set project PROJECT_ID`)
4. APIs enabled: Dataproc, Compute Engine, Cloud Storage

## Create a GCS bucket for your data

Dataproc reads and writes through Google Cloud Storage by default:

```bash
gsutil mb -l us-central1 gs://bsan726-yourname-bucket
gsutil cp local_data.csv gs://bsan726-yourname-bucket/input/
```

## Create a Dataproc cluster

A minimal cluster for coursework:

```bash
gcloud dataproc clusters create bsan-cluster \
  --region=us-central1 \
  --zone=us-central1-a \
  --master-machine-type=n1-standard-2 \
  --worker-machine-type=n1-standard-2 \
  --num-workers=2 \
  --image-version=2.2-debian12
```

What each flag means:

- `--region` — GCP region to run in; keep this consistent with your bucket for lower data-transfer costs
- `--master-machine-type`, `--worker-machine-type` — VM sizes
- `--num-workers=2` — two worker nodes plus one master is the practical minimum
- `--image-version=2.2-debian12` — the Dataproc OS / software image; 2.x images ship Spark 3.x

The cluster takes 2–3 minutes to come up.

## Submit a job

**PySpark example** — count words in a Cloud Storage file:

```bash
gcloud dataproc jobs submit pyspark gs://bsan726-yourname-bucket/jobs/wordcount.py \
  --cluster=bsan-cluster \
  --region=us-central1 \
  -- gs://bsan726-yourname-bucket/input/shakespeare.txt \
     gs://bsan726-yourname-bucket/output/
```

**Hive example** — run a SQL query against a Hive external table:

```bash
gcloud dataproc jobs submit hive \
  --cluster=bsan-cluster \
  --region=us-central1 \
  --execute="SELECT country, COUNT(*) AS n
             FROM customers_external
             GROUP BY country ORDER BY n DESC LIMIT 10;"
```

You can also submit Spark (`spark`), Spark-SQL (`spark-sql`), Hadoop (`hadoop`), and Pig (`pig`) jobs the same way.

## Connecting interactively

For exploration, SSH into the master and use the familiar CLIs:

```bash
gcloud compute ssh bsan-cluster-m --zone=us-central1-a
# then on the master:
pyspark
hive
spark-shell
```

For notebooks, enable the **Jupyter optional component** when you create the cluster and Dataproc will host JupyterLab on the master.

## Cost discipline

Dataproc bills per-second of cluster life regardless of whether a job is running. Habits that keep bills low:

- **Delete clusters when you're done:** `gcloud dataproc clusters delete bsan-cluster --region=us-central1`
- **Use preemptible workers** for batch work: `--num-preemptible-workers=4` — ~80% cheaper, may be reclaimed
- **Prefer single-node clusters** for coursework that doesn't truly need parallelism: `--single-node`
- **Use ephemeral clusters** — spin up for the job, spin down at the end. Dataproc Workflow Templates automate this pattern.

## Dataproc Serverless

A newer offering: **Dataproc Serverless for Spark** runs Spark jobs without you managing a cluster at all. You submit a job, Google picks the resources, and you pay only for the job's runtime. For many coursework and production jobs, this is now the preferred path.

## Learning outcomes

- Create and configure a Dataproc cluster from the `gcloud` CLI
- Upload data to Cloud Storage and read it from cluster jobs
- Submit PySpark, Spark-SQL, and Hive jobs
- SSH into the master and use interactive tools
- Apply cost-control habits (ephemeral clusters, preemptible workers, serverless)

